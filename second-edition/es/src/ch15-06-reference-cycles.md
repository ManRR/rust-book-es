## Los ciclos de referencia pueden perder memoria

Las garantías de seguridad de la memoria de Rust hacen que sea difícil, pero
no imposible, crear accidentalmente una memoria que nunca se limpia
(conocida como *pérdida/fuga de memoria* (*memory leak*). La prevención de
fugas de memoria por completo no es una de las garantías de Rust, del mismo
modo que no permite carreras de datos en tiempo de compilación, lo que
significa que las fugas de memoria son seguras para la memoria en Rust.
Podemos ver que Rust permite fugas de memoria usando `Rc<T>` y
`RefCell<T>`: es posible crear referencias donde los elementos se refieren
entre sí en un ciclo. Esto crea pérdidas de memoria porque el recuento de
referencias de cada elemento en el ciclo nunca llegará a 0, y los valores
nunca se descartarán.

### Crear un ciclo de referencia

Veamos cómo podría suceder un ciclo de referencia y cómo prevenirlo,
comenzando con la definición de la enumeración `List` y un método `tail`
en el Listado 15-25:

<span class="filename">Filename: src/main.rs</span>

<!-- Hidden fn main is here to disable the automatic wrapping in fn main that
doc tests do; the `use List` fails if this listing is put within a main -->

```rust
# fn main() {}
use std::rc::Rc;
use std::cell::RefCell;
use List::{Cons, Nil};

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match *self {
            Cons(_, ref item) => Some(item),
            Nil => None,
        }
    }
}
```

<span class="caption">Listado 15-25: Una definición de lista de conscriptos
que contiene un `RefCell<T>` para que podamos modificar a qué se refiere una
variante `Cons`</span>

Estamos usando otra variación de la definición de `List` en el Listado 15-5.
El segundo elemento en la variante `Cons` ahora es `RefCell<Rc<List>>`, lo
que significa que en lugar de tener la capacidad de modificar el valor `i32`
como lo hicimos en el Listado 15-24, queremos modificar cual `List` value
está apuntando a una variante `Cons`. También estamos agregando un método de
`tail` para que sea conveniente para nosotros acceder al segundo elemento si
tenemos una variante `Cons`.

En el Listado 15-26, estamos agregando una función `main` que usa las
definiciones en el Listado 15-25. Este código crea una lista en `a` y una
lista en `b` que apunta a la lista en `a`. Luego modifica la lista en `a`
para apuntar a `b`, creando un ciclo de referencia. Hay declaraciones
`println!` A lo largo del camino para mostrar cuáles son los recuentos de
referencia en varios puntos de este proceso.

<span class="filename">Filename: src/main.rs</span>

```rust
# use List::{Cons, Nil};
# use std::rc::Rc;
# use std::cell::RefCell;
# #[derive(Debug)]
# enum List {
#     Cons(i32, RefCell<Rc<List>>),
#     Nil,
# }
#
# impl List {
#     fn tail(&self) -> Option<&RefCell<Rc<List>>> {
#         match *self {
#             Cons(_, ref item) => Some(item),
#             Nil => None,
#         }
#     }
# }
#
fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));

    println!("a initial rc count = {}", Rc::strong_count(&a));
    println!("a next item = {:?}", a.tail());

    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));

    println!("a rc count after b creation = {}", Rc::strong_count(&a));
    println!("b initial rc count = {}", Rc::strong_count(&b));
    println!("b next item = {:?}", b.tail());

    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }

    println!("b rc count after changing a = {}", Rc::strong_count(&b));
    println!("a rc count after changing a = {}", Rc::strong_count(&a));

    // Descomenta la siguiente línea para ver que tenemos un ciclo;
    // desbordará la pila
    // println!("a next item = {:?}", a.tail());
}
```

<span class="caption">Listado 15-26: Creación de un ciclo de referencia de
dos valores `List` que apuntan el uno al otro</span>

Creamos una instancia `Rc<List>` que contiene un valor `List` en la variable
`a` con una lista inicial de `5, Nil`. Luego creamos una instancia
`Rc<List>` que contiene otro valor `List` en la variable `b` que contiene el
valor 10 y apunta a la lista en `a`.

Modificamos `a` para que apunte a `b` en lugar de `Nil`, creando un ciclo.
Hacemos eso usando el método `tail` para obtener una referencia al
`RefCell<Rc<List>>` en `a`, que ponemos en la variable `link`. Luego usamos
el método `borrow_mut` en `RefCell<Rc<List>>` para cambiar el valor dentro de
un `Rc<List>` que contiene un valor `Nil` al `Rc<List>` en `b`.

Cuando ejecutamos este código, manteniendo el último `println!`. Comentado por
el momento, obtendremos este resultado:

```text
a initial rc count = 1
a next item = Some(RefCell { value: Nil })
a rc count after b creation = 2
b initial rc count = 1
b next item = Some(RefCell { value: Cons(5, RefCell { value: Nil }) })
b rc count after changing a = 2
a rc count after changing a = 2
```

El recuento de referencias de las instancias `Rc<List>` en `a` y `b` es 2
después de que cambiamos la lista en `a` para que apunte a `b`. Al final de
`main`, Rust intentará soltar `b` primero, lo que disminuirá el recuento en
cada una de las instancias `Rc<List>` en `a` y `b` en 1.

Sin embargo, debido a que `a` todavía hace referencia a `Rc<List>` que estaba
en `b`, ese `Rc<List>` tiene un recuento de 1 en lugar de 0, por lo que la
memoria `Rc<List>` tiene en el montículo no se eliminará. La memoria se quedará
allí con un conteo de 1, para siempre. Para visualizar este ciclo de
referencia, hemos creado un diagrama en la Figura 15-4.

<img alt="Reference cycle of lists" src="img/trpl15-04.svg" class="center" />

<span class="caption">Figura 15-4: Un ciclo de referencia de listas `a` y `b`
que apuntan entre sí</span>

Si elimina el último `println!` y ejecuta el programa, Rust intentará
imprimir este ciclo con `a` apuntando a `b` apuntando a `a` y así
sucesivamente hasta que desborda la pila.

En este caso, justo después de que creamos el ciclo de referencia, el programa finaliza. Las consecuencias de este ciclo no son muy graves. Sin
embargo, si un programa más complejo asignó mucha memoria en un ciclo y la retuvo durante mucho tiempo, el programa usaría más memoria de la necesaria y podría abrumar al sistema, haciendo que se quede sin memoria disponible.

Crear ciclos de referencia no se realiza fácilmente, pero tampoco es
imposible. Si tiene valores `RefCell<T>` que contienen valores `Rc<T>` o
anidados similares combinaciones de tipos con mutabilidad interior y recuento
de referencias, debe asegúrese de no crear ciclos; no puedes confiar en que
Rust los atrape. Crear un ciclo de referencia sería un error lógico en su
programa que debería utilizar pruebas automatizadas, revisiones de códigos y
otras prácticas de desarrollo de software para minimizar.

Otra solución para evitar ciclos de referencia es reorganizar sus datos
estructuras para que algunas referencias expresen propiedad y algunas
referencias no. Como resultado, puede tener ciclos formados por algunas
relaciones de propiedad y algunas relaciones que no son propiedad, y solo las
relaciones de propiedad afectan si se puede eliminar un valor o no en el
Listado 15-25, siempre queremos variantes `Cons` para poseer su lista, por lo
que no es posible reorganizar la estructura de datos. Veamos un ejemplo
utilizando gráficos formados por nodos principales y nodos secundarios
para ver cuándo las relaciones ajenas a la propiedad son una forma apropiada
de prevenir ciclos de referencia.

### Evitar ciclos de referencia: Convertir un `Rc<T>` en un `Weak<T>`

Hasta ahora, hemos demostrado que llamar `Rc::clone` aumenta la
`strong_count` de una instancia `Rc<T>`, y una instancia `Rc <T>` solo se
limpia si su `strong_count` es 0. También puede crear una *referencia débil*
al valor dentro de una instancia `Rc<T>` invocando `Rc::downgrade` y pasando
una referencia a `Rc<T>`. Cuando llamas a `Rc::downgrade`, obtienes un
puntero inteligente del tipo `Weak<T>`. En lugar de aumentar el
`strong_count` en la instancia `Rc<T>` por 1, llamando `Rc::downgrade`
aumenta el `weak_count` por 1.
El tipo `Rc<T>` utiliza `weak_count` para realizar un seguimiento de cuántas
referencias `Weak<T>` existen, similares a `strong_count`. La diferencia es
`weak_count` no necesita ser 0 para que la instancia `Rc<T>` sea limpiada.

Las referencias fuertes son cómo puede compartir la propiedad de una instancia `Rc<T>`. Las referencias débiles no expresan una relación de propiedad. No provocarán un ciclo de referencia porque cualquier ciclo que
implique algunas referencias débiles se romperá una vez que el recuento
fuerte de referencias involucrado sea 0.

Debido a que el valor que las referencias de `Weak<T>` pueden haberse
descartado, para hacer cualquier cosa con el valor al que apunta `Weak<T>`,
debe asegurarse de que el valor aún exista. Haga esto llamando al método
`upgrade` en una instancia `Weak<T>`, que devolverá un `Option<Rc<T>>`.
Obtendrá un resultado de `Some` si el valor `Rc<T>` no se ha eliminado aún y
un resultado de `None` si se ha eliminado el valor `Rc<T>`. Como `upgrade`
devuelve un `Option<T>`, Rust se asegurará de que se manejen el caso `Some` y
el caso `None`, y que no habrá un puntero inválido.

Como ejemplo, en lugar de utilizar una lista cuyos elementos solo conozcan el
siguiente ítem, crearemos un árbol cuyos elementos conozcan sus elementos
secundarios *y* sus elementos principales.

#### Crear una estructura *Tree Data*: un `Nodo` con nodos secundarios

Para empezar, construiremos un árbol con nodos que conozcan sus nodos
secundarios. Crearemos una estructura llamada `Node` que tenga su propio
valor `i32` y referencias a sus valores `Node` hijos:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    value: i32,
    children: RefCell<Vec<Rc<Node>>>,
}
```

Queremos un `Node` para poseer sus hijos, y queremos compartir esa propiedad
con variables para que podamos acceder a cada `Node` en el árbol
directamente. Para hacer esto, definimos los elementos `Vec<T>` para que
sean valores de tipo `Rc<Node>`. También queremos modificar qué nodos son
secundarios de otro nodo, por lo que tenemos un `RefCell<T>` en `children`
alrededor de `Vec<Rc<Node>>`.

A continuación, usaremos nuestra definición de estructura y crearemos una
instancia `Node` llamada `leaf` con el valor 3 y no *children*, y otra
instancia llamada `branch` con el valor 5 y `leaf` como uno de sus hijos,
como se muestra en el listado 15-27:

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::rc::Rc;
# use std::cell::RefCell;
#
# #[derive(Debug)]
# struct Node {
#     value: i32,
#    children: RefCell<Vec<Rc<Node>>>,
# }
#
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        children: RefCell::new(vec![]),
    });

    let branch = Rc::new(Node {
        value: 5,
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });
}
```

<span class="caption">Listado 15-27: Creando un nodo `leaf` sin hijos y un
nodo `branch` con `leaf` como uno de sus hijos</span>

Clonamos `Rc<Node>` en `leaf` y almacenamos eso en `branch`, lo que significa
que `Node` en `leaf` ahora tiene dos dueños: `leaf` y `branch`. Podemos pasar
de `branch` a` leaf` a `branch.children`, pero no hay forma de pasar de
`leaf` a `branch`. La razón es que `leaf` no hace referencia a `branch` y no
sabe que están relacionadas. Queremos que `leaf` sepa que `branch` es su
padre. Lo haremos a continuación.

#### Agregar una referencia de un hijo a su padre

Para hacer que el nodo hijo esté al tanto de su elemento principal,
necesitamos agregar un campo `parent` a nuestra definición de estructura
`Node`. El problema está en decidir cuál debería ser el tipo de `parent`.
Sabemos que no puede contener un `Rc<T>`, porque eso crearía un ciclo de
referencia con `leaf.parent` apuntando a `branch` y `branch.children`
apuntando a `leaf`, lo que causaría su `strong_count` para nunca ser 0.

Al pensar en las relaciones de otra manera, un nodo padre debería ser dueño
de sus hijos: si se descarta un nodo padre, también se deben descartar sus
nodos secundarios. Sin embargo, un hijo no debería ser el propietario de su
padre: si eliminamos un nodo hijo, el padre debería existir. ¡Este es un caso
para referencias débiles!.

Entonces en lugar de `Rc<T>`, haremos que el tipo de `parent` use `Weak<T>`,
específicamente un `RefCell<Weak<Node>>`. Ahora nuestra definición de
estructura `Node` se ve así:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

#[derive(Debug)]
struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}
```

Un nodo podrá hacer referencia a su nodo padre pero no posee su padre. En el
listado 15-28, actualizamos `main` para usar esta nueva definición para que
el nodo `leaf` tenga una manera de referirse a su elemento primario, `branch`:

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::rc::{Rc, Weak};
# use std::cell::RefCell;
#
# #[derive(Debug)]
# struct Node {
#     value: i32,
#     parent: RefCell<Weak<Node>>,
#     children: RefCell<Vec<Rc<Node>>>,
# }
#
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());

    let branch = Rc::new(Node {
        value: 5,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![Rc::clone(&leaf)]),
    });

    *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
}
```

<span class="caption">Listado 15-28: Un nodo `leaf` con una referencia débil
a su `branch` nodo padre</span>

La creación del nodo `leaf` es similar a la creación del nodo `leaf` en el
Listado 15-27 con la excepción del campo `parent`: `leaf` comienza sin un
padre, por lo que creamos una nueva `Weak<Node>` vacía instancia de
referencia.

En este punto, cuando tratamos de obtener una referencia al padre de `leaf`
utilizando el método `upgrade`, obtenemos un valor `None`. Vemos esto en el
resultado de la primera declaración `println!`:

```text
leaf parent = None
```

Cuando creamos el nodo `branch`, también tendrá una nueva referencia
`Weak<Node>`en el campo `parent`, porque `branch` no tiene un nodo padre.
Todavía tenemos `leaf` como uno de los hijos de `branch`. Una vez que tenemos
la instancia `Node` en `branch`, podemos modificar `leaf` para darle una
referencia `Weak<Node>` a su padre. Usamos el método `borrow_mut` en
`RefCell<Weak<Node>>` en el campo `parent` de `leaf`, y luego usamos la
función `Rc::downgrade` para crear `Weak<Node>` referencia a `branch` desde
`Rc<Node>` en `branch`.

Cuando imprimimos el padre de `leaf` nuevamente, esta vez obtendremos una
variante `Some` sosteniendo `branch`: ¡ahora `leaf` puede acceder a su padre!
Cuando imprimimos `leaf`, también evitamos el ciclo que finalmente terminó en
un desbordamiento de pila como el que teníamos en el Listado 15-26; las
referencias `Weak<Node>` se imprimen como `(Weak)`:

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

La falta de salida infinita indica que este código no creó un ciclo de
referencia. También podemos decir esto mirando los valores que obtenemos al
llamar `Rc::strong_count` y `Rc::weak_count`.

#### Visualización de cambios en `strong_count` y `weak_count`

Veamos cómo cambian los valores `strong_count` y `weak_count` de las
instancias `Rc<Node>` al crear un nuevo ámbito interno y mover la creación de
`branch` a ese ámbito. Al hacerlo, podemos ver qué sucede cuando `branch` se
crea y luego se descarta cuando sale del alcance. Las modificaciones se
muestran en el listado 15-29:

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::rc::{Rc, Weak};
# use std::cell::RefCell;
#
# #[derive(Debug)]
# struct Node {
#     value: i32,
#     parent: RefCell<Weak<Node>>,
#     children: RefCell<Vec<Rc<Node>>>,
# }
#
fn main() {
    let leaf = Rc::new(Node {
        value: 3,
        parent: RefCell::new(Weak::new()),
        children: RefCell::new(vec![]),
    });

    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );

    {
        let branch = Rc::new(Node {
            value: 5,
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![Rc::clone(&leaf)]),
        });

        *leaf.parent.borrow_mut() = Rc::downgrade(&branch);

        println!(
            "branch strong = {}, weak = {}",
            Rc::strong_count(&branch),
            Rc::weak_count(&branch),
        );

        println!(
            "leaf strong = {}, weak = {}",
            Rc::strong_count(&leaf),
            Rc::weak_count(&leaf),
        );
    }

    println!("leaf parent = {:?}", leaf.parent.borrow().upgrade());
    println!(
        "leaf strong = {}, weak = {}",
        Rc::strong_count(&leaf),
        Rc::weak_count(&leaf),
    );
}
```

<span class="caption">Listado 15-29: Crea `branch` en un ámbito interno y
examina los recuentos de referencia fuertes y débiles</span>

Después de que se crea `leaf`, su `Rc <Node>` tiene un recuento fuerte de 1 y un recuento débil de 0. En el ámbito interno, creamos `branch` y lo asociamos
con `leaf`, en cuyo punto cuando imprimimos los conteos, `Rc<Node>` en
`branch` tendrá un recuento fuerte de 1 y un conteo débil de 1
(para el señalamiento `leaf.parent` a `branch` con un `Weak<Node>`). Cuando
imprimimos los recuentos en `leaf`, veremos tendrá una cuenta fuerte de 2,
porque `branch` ahora tiene un clon del `Rc<Node>` de `leaf` almacenado en
`branch.children`, pero aún tendrá un conteo débil de 0.

Cuando el alcance interno finaliza, `branch` sale del alcance y el recuento fuerte de el `Rc<Node>` disminuye a 0, por lo que su `Node` se elimina. El
conteo débil de 1 de `leaf.parent` no tiene relación con si `Node` se cae o
no, así que ¡así que no tenemos pérdidas de memoria!.

Si tratamos de acceder al padre de `leaf` después del final del alcance,
obtendremos `Nine` de nuevo. Al final del programa, `Rc<Node>` en `leaf`
tiene un recuento fuerte de 1 y un conteo débil de 0, porque la variable
`leaf` ahora es la única referencia al `Rc<Node>` de nuevo.

Toda la lógica que maneja los conteos y la caída de valor está integrada en
`Rc<T>` y `Weak<T>` y sus implementaciones del *trait* `Drop`. Por
especificando que la relación entre un hijo y su padre debe ser una
referencia `Weak<T>` en la definición de `Nodo`, puede hacer que los nodos principales apunten a nodos secundarios y viceversa sin crear un ciclo de referencia y pérdidas de memoria.

## Resumen

Este capítulo cubrió cómo usar punteros inteligentes para hacer diferentes
garantías y compensaciones de las que Rust fabrica por defecto con
referencias regulares. El tipo `Box<T>` tiene un tamaño conocido y apunta a
los datos asignados en el montículo. El tipo `Rc<T>` realiza un seguimiento
del número de referencias a los datos en el montículo para que los datos
puedan tener varios propietarios. El tipo `RefCell<T>` con su mutabilidad
interior nos da un tipo que podemos usar cuando necesitamos un tipo inmutable
pero necesitamos cambiar un valor interno de ese tipo; también aplica las
reglas de endeudamiento en tiempo de ejecución en lugar de en tiempo de
compilación.

También se discutieron los *trait* `Deref` y `Drop`, que permiten una gran
cantidad de la funcionalidad de punteros inteligentes. Exploramos los ciclos
de referencia que pueden causar pérdidas de memoria y cómo evitar que usen
`Weak<T>`.

Si este capítulo despertó su interés y desea implementar sus propios
indicadores inteligentes, consulte [“The Rustonomicon”][nomicon] para obtener más información útil.

[nomicon]: https://doc.rust-lang.org/stable/nomicon/

A continuación, hablaremos de concurrencia en Rust. Incluso aprenderá sobre
algunos nuevos *punteros inteligentes* (*smart pointers*).
