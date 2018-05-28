## Traits: Definición de comportamiento compartido

Un *trait* le dice al compilador Rust acerca de la funcionalidad que un tipo
en particular tiene y puede compartir con otros tipos. Podemos usar *trait*
para definir el comportamiento compartido de una manera abstracta. Podemos
usar límites de *trait* para especificar que un genérico puede ser de
cualquier tipo que tenga cierto comportamiento.

> Nota: Los *trait* son similares a una característica a menudo llamada
> *interfaces* en otros lenguaje, aunque con algunas diferencias.

### Definiendo un Trait

El comportamiento de un tipo consiste en los métodos que podemos invocar en
ese tipo. Los diferentes tipos comparten el mismo comportamiento si podemos
llamar a los mismos métodos en todos esos tipos. Las definiciones de *trait*
son una forma de agrupar las firmas de métodos para definir un conjunto de
comportamientos necesarios para lograr algún propósito.

Por ejemplo, supongamos que tenemos estructuras múltiples que contienen
varios tipos y cantidades de texto: una estructura `NewsArticle` que contiene
una historia de noticias presentada en una ubicación particular y un `Tweet`
que puede tener como máximo 280 caracteres junto con metadatos que indican si
fue un nuevo tweet, un retweet o una respuesta a otro tweet.

Queremos crear una biblioteca de agregadores de medios que pueda mostrar
resúmenes de datos que podrían almacenarse en una instancia de `NewsArticle`
o `Tweet`. Para hacer esto, necesitamos un resumen de cada tipo, y
necesitamos solicitar ese resumen llamando a un método `summarize` en una
instancia. El listado 10-12 muestra la definición de un *trait* `Summary` que
expresa este comportamiento.

<span class="filename">Filename: src/lib.rs</span>

```rust
pub trait Summary {
    fn summarize(&self) -> String;
}
```

<span class="caption">Listing 10-12: Un *trait* `Resumen` que consiste en el comportamiento proporcionado por un método `summarize`</span>

Aquí, declaramos un *trait* usando la palabra clave `trait` y luego el nombre
del *trait*, que es `Summary`en este caso. Dentro de las llaves, declaramos
las firmas de métodos que describen los comportamientos de los tipos que
implementan este *trait*, que en este caso es
`fn summarize(& self) -> String`.

Después de la firma del método, en lugar de proporcionar una implementación
entre llaves, usamos un punto y coma. Cada tipo que implementa este *trait*
debe proporcionar su propio comportamiento personalizado para el cuerpo del
método. El compilador hará cumplir que cualquier tipo que tenga el *trait*
`Summary`tendrá el método `summarize` definido con esta firma exactamente.

Un *trait* puede tener múltiples métodos en su cuerpo: las firmas del método
se enumeran una por línea y cada línea termina en punto y coma.

### Implementando un rasgo en un tipo

Ahora que hemos definido el comportamiento deseado utilizando el *trait* `Summary`, podemos implementarlo en los tipos en nuestro agregador de medios.
El listado 10-13 muestra una implementación del *trait* `Summary` en la
estructura `NewsArticle` que usa el título, el autor y la ubicación para
crear el valor de retorno de `summarize`. Para la estructura `Tweet`,
definimos `summarize` como el nombre de usuario seguido por el texto completo
del tweet, suponiendo que el contenido del tweet ya está limitado a 280
caracteres.

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub trait Summary {
#     fn summarize(&self) -> String;
# }
#
pub struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

pub struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

<span class="caption">Listado 10-13: Implementando el *trait* `Summary` en los
tipos `NewsArticle` y `Tweet`</span>

La implementación de un *trait* en un tipo es similar a la implementación de
métodos regulares. La diferencia es que después de `impl`, ponemos el nombre
del *trait* que queremos implementar, luego usamos la palabra clave `for`, y
luego especificamos el nombre del tipo para el cual queremos implementar el
*trait*. Dentro del bloque `impl`, ponemos las firmas de método que ha
definido la definición de *trait*. En lugar de agregar un punto y coma
después de cada firma, usamos llaves y completamos el cuerpo del método
con el comportamiento específico que queremos que tengan los métodos del
*trait* para el tipo particular.

Después de implementar el *trait*, podemos llamar a los métodos en instancias
de `NewsArticle` y `Tweet` de la misma manera que llamamos a los métodos
regulares, como este:

```rust,ignore
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

println!("1 new tweet: {}", tweet.summarize());
```

Este código imprime `1 new tweet: horse_ebooks: of course, as you probably already
know, people`.

Tenga en cuenta que porque definimos el *trait* `Summary` y el `NewsArticle`
y tipos de `Tweet` en la misma *lib.rs* en el Listado 10-13, todos están en
el mismo alcance. Digamos que esto *lib.rs* es para un *create* que hemos llamado `aggregator` y alguien más quiere usar la funcionalidad de nuestro
*crate* para implementar el `Summary` *trait* en una estructura definida
dentro del alcance de su biblioteca. Tendrían que importar el *trait* en su alcance primero. Lo harían especificando
`use aggregator::Summary;`, que luego le permitiría implementar `Summary` para
su tipo. El *trait* `Summary` también necesitaría ser un *trait* público para
otra *create* para implementarlo, que es porque ponemos la palabra clave `pub`
antes de *trait* en el Listado 10-12.

Una restricción a tener en cuenta con las implementaciones de *traits* es que
podemos implementar un *trait* en un tipo solo si el *trait* o el tipo es
local para nuestro *crate*. Por ejemplo, podemos implementar *traits* de la biblioteca estándar como `Display` en un tipo personalizado como `Tweet` como
parte de nuestra funcionalidad `aggregator`, porque el tipo `Tweet` es local
a nuestro *crate* `aggregator`. También podemos implementar `Summary` en
`Vec <T>` en nuestro *crate* `aggregator`, porque el *trait* `Summary` es
local para nuestro *crate* `aggregator`.

Pero no podemos implementar *traits* externos en tipos externos. Por ejemplo,
no podemos implementar el *trait* `Display` en `Vec <T>` dentro de nuestro
*crate* `aggregator`, porque `Display` y` Vec <T>` están definidos en la
biblioteca estándar y no son local a nuestro *crate* `agregador` Esta
restricción es parte de una propiedad de programas llamados *coherencia*, y
más específicamente, la *regla huérfana*, llamada así porque el tipo
principal no está presente. Esta regla asegura que el código de otras
personas no puede romper su código y viceversa. Sin la regla, dos *crates*
podrían implementar el mismo *trait* para el mismo tipo, y Rust no sabría qué
implementación usar.

### Implementaciones predeterminadas

A veces es útil tener un comportamiento predeterminado para algunos o todos
los métodos en un *trait* en lugar de requerir implementaciones para todos
los métodos en cada tipo. Luego, a medida que implementamos el *trait* en un
tipo particular, podemos mantener o anular el comportamiento predeterminado
de cada método.

El Listado 10-14 muestra cómo especificar un *string* predeterminada para el
método `summarize` del *trait* `Summary` en lugar de solo definir la firma
del método, como hicimos en el Listado 10-12.

<span class="filename">Filename: src/lib.rs</span>

```rust
pub trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

<span class="caption">Listado 10-14: Definición de un *trait* `Summary` con una implementación predeterminada del método `summarize`</span>

Para usar una implementación predeterminada para resumir instancias de
`NewsArticle` en lugar de definir una implementación personalizada,
especificamos un bloque `impl` vacío con `impl Summary for NewsArticle {}`.

Aunque ya no estamos definiendo el método `summarize` en `NewsArticle`
directamente, hemos proporcionado una implementación predeterminada y
especificamos que `NewsArticle` implementa el *trai* `Summary`. Como
resultado, todavía podemos llamar al método `summarize` en una instancia de
`NewsArticle`, así:

```rust,ignore
let article = NewsArticle {
    headline: String::from("Penguins win the Stanley Cup Championship!"),
    location: String::from("Pittsburgh, PA, USA"),
    author: String::from("Iceburgh"),
    content: String::from("The Pittsburgh Penguins once again are the best
    hockey team in the NHL."),
};

println!("New article available! {}", article.summarize());
```

Este código imprime `New article available! (Read more...)`.

Crear una implementación predeterminada para `summarize` no requiere que
cambiemos nada sobre la implementación de `Summary` en `Tweet` en el Listado
10-13. La razón es que la sintaxis para anular una implementación
predeterminada es la misma que la sintaxis para implementar un método de
*trait* que no tiene una implementación predeterminada.

Las implementaciones predeterminadas pueden llamar a otros métodos en el
mismo *trait*, incluso si esos otros métodos no tienen una implementación
predeterminada. De esta forma, un *trait* puede proporcionar una gran
cantidad de funcionalidades útiles y solo requiere que los implementadores
especifiquen una pequeña parte de él. Por ejemplo, podríamos definir el
*trait* `Summary` para tener un método `summarize_author` cuya implementación
es necesaria, y luego definir un método `summarize` que tenga una
implementación predeterminada que llame al método `summarize_author`:

```rust
pub trait Summary {
    fn summarize_author(&self) -> String;

    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}
```

Para usar esta versión de `Summary`, solo necesitamos definir
`summarize_author` cuando implementamos el *trait* en un tipo:

```rust,ignore
impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }
}
```

Después de definir `summarize_author`, podemos llamar `summarize` en las
instancias de la estructura `Tweet`, y la implementación predeterminada de
`summarize` llamará a la definición de `summarize_author` que hemos
proporcionado. Debido a que hemos implementado `summarize_author`, el
*trait* `Summary` nos ha dado el comportamiento del método `summarize` sin
necesidad de escribir más código.

```rust,ignore
let tweet = Tweet {
    username: String::from("horse_ebooks"),
    content: String::from("of course, as you probably already know, people"),
    reply: false,
    retweet: false,
};

println!("1 new tweet: {}", tweet.summarize());
```

Este código imprime `1 new tweet: (Read more from @horse_ebooks...)`.

Tenga en cuenta que no es posible llamar a la implementación predeterminada
desde una implementación principal de ese mismo método.

### Trait Bounds

Ahora que sabe cómo definir *trait* e implementar esos *traits* en los tipos,
podemos explorar cómo usar los *traits* con los parámetros de tipo genérico.
Podemos usar *trait bounds* para restringir los tipos genéricos para
garantizar que el tipo se limitará a aquellos que implementan un *trait* y
comportamiento particular.

Por ejemplo, en el listado 10-13, implementamos el *trait* `Summary` en los
tipos `NewsArticle` y `Tweet`. Podemos definir una función `notify` que llama
al método `summarize` en su parámetro `item`, que es del tipo genérico `T`.
Para poder llamar `summarize` on `item` sin recibir un error que nos diga que
el tipo genérico `T` no implementa el método `summarize`, podemos usar
límites de *traits* en `T` para especificar ese `item` debe ser de un tipo
que implemente el *trait* `Summary`:

```rust,ignore
pub fn notify<T: Summary>(item: T) {
    println!("Breaking news! {}", item.summarize());
}
```

Colocamos límites de *trait* con la declaración del parámetro de tipo
genérico, después de dos puntos y dentro de corchetes angulares. Debido a la
característica de límite en `T`, podemos llamar a `notify` y pasar cualquier
instancia de `NewsArticle` o `Tweet`. El código que llama a la función con
cualquier otro tipo, como un `String` o un `i32`, no compilará, porque esos
tipos no implementan `Summary`.

Podemos especificar múltiples límites de *trait* en un tipo genérico usando
la sintaxis `+`. Por ejemplo, para usar el formato de visualización en el
tipo `T` en una función así como en el método `summarize`, podemos usar
`T: Summary + Display` para decir `T` puede ser cualquier tipo que implemente
`Summary` y `Display`.

Sin embargo, hay desventajas al uso de demasiados límites de *trait*. Cada
genérico tiene sus propios límites de *trait*, por lo que las funciones con
múltiples parámetros de tipo genérico pueden tener mucha información de
límite de caracteres entre el nombre de una función y su lista de parámetros,
lo que hace que la firma de la función sea difícil de leer. Por esta razón,
Rust tiene una sintaxis alternativa para especificar límites de *trait*
dentro de una cláusula `where` después de la firma de la función. Entonces,
en lugar de escribir esto:

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: T, u: U) -> i32 {
```

podemos usar una cláusula `where`, como esta:

```rust,ignore
fn some_function<T, U>(t: T, u: U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
```

La firma de esta función es menos desordenada porque el nombre de la función,
la lista de parámetros y el tipo de retorno están muy juntos, de forma
similar a una función sin muchos límites de caracteres.

### Reparar la función `largest` con límites de *Trait*

Ahora que sabe cómo especificar el comportamiento que desea usar utilizando
los límites del parámetro de tipo genérico, regresemos al Listado 10-5 para
corregir la definición de la función `largest` que usa un parámetro de
tipo genérico. La última vez que intentamos ejecutar ese código, recibimos
este error:

```text
error[E0369]: binary operation `>` cannot be applied to type `T`
 --> src/main.rs:5:12
  |
5 |         if item > largest {
  |            ^^^^^^^^^^^^^^
  |
  = note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

En el cuerpo de `largest` queríamos comparar dos valores de tipo `T` usando
el operador mayor que (`>`). Debido a que ese operador se define como un
método predeterminado en el *trait* de la biblioteca estándar
`std::cmp::PartialOrd`, necesitamos especificar `PartialOrd` en los límites
de *trait* para `T`, de modo que la función `largest` puede funcionar en
sectores de cualquier tipo que podamos comparar no necesitamos traer
`PartialOrd` al alcance porque está en el preludio. Cambie la firma de
`largest` para que se vea así:

```rust,ignore
fn largest<T: PartialOrd>(list: &[T]) -> T {
```

Esta vez, cuando compilamos el código, obtenemos un conjunto diferente de errores:

```text
error[E0508]: cannot move out of type `[T]`, a non-copy slice
 --> src/main.rs:2:23
  |
2 |     let mut largest = list[0];
  |                       ^^^^^^^
  |                       |
  |                       cannot move out of here
  |                       help: consider using a reference instead: `&list[0]`

error[E0507]: cannot move out of borrowed content
 --> src/main.rs:4:9
  |
4 |     for &item in list.iter() {
  |         ^----
  |         ||
  |         |hint: to prevent move, use `ref item` or `ref mut item`
  |         cannot move out of borrowed content
```

La línea clave en este error es
`cannot move out of type [T], a non-copy slice`. Con nuestras versiones no
genéricas de la función `largest`, solo intentábamos encontrar el `i32` o
`char` más grande. Como se estudió en la sección “Datos apilados solo: copia”
en el Capítulo 4, los tipos como `i32` y `char` que tienen un tamaño conocido
se pueden almacenar en la pila (*stack*), por lo que implementan el *trait*
`Copy`. Pero cuando hicimos genérica la función `largest`, se hizo posible
que el parámetro `list` tuviera tipos que no implementan el *trait* `Copy`.
En consecuencia, no podríamos mover el valor fuera de `list[0]` y dentro de
la variable `largest`, lo que daría como resultado este error.

Para llamar a este código solo con aquellos tipos que implementan el *trait*
`Copy`, podemos agregar `Copy` a los límites de *trait* de `T`! El Listado
10-15 muestra el código completo de una función genérica `largest` que se
compilará siempre que los tipos de los valores en la porción que pasamos a la
función implementen los *trait* `PartialOrd` *y* `Copy`, como `i32` y `char`
hacen.

<span class="filename">Filename: src/main.rs</span>

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest(&char_list);
    println!("The largest char is {}", result);
}
```

<span class="caption">Listing 10-15: Una definición trabajando de la función `largest` que funciona en cualquier tipo genérico que implemente los *traits* `PartialOrd` y `Copy`</span>

Si no queremos restringir la función `largest` a los tipos que implementan el
*trait* `Copy`, podríamos especificar que `T` tenga *trait bound* `Clone` en
lugar de `Copy`. Entonces podríamos clonar cada valor en el *slice* cuando
queremos que la función `largest` tenga propiedad. Usar la función `clone`
significa que potencialmente estamos haciendo más asignaciones de *heap* en
el caso de tipos que poseen datos de montículo (*heap*) como `String`, y las
asignaciones de *heap* pueden ser lentas si estamos trabajando con grandes
cantidades de datos.

Otra forma en que podríamos implementar `largest` es que la función devuelva
una referencia a un valor `T` en el *slice*. Si cambiamos el tipo de retorno
a `& T` en lugar de `T`, cambiando así el cuerpo de la función para devolver
una referencia, no necesitaríamos los límites *trait* (*trait bounds*)
`Clone` o `Copy` y podríamos evitar las asignaciones de *heap*. ¡Intente implementar estas soluciones alternativas por su cuenta!

### Usar límites de *trait* (*Trait Bounds*) para implementar métodos condicionalmente

Al usar un *trait* vinculado con un bloque `impl` que usa parámetros de tipo
genérico, podemos implementar métodos condicionalmente para tipos que
implementan los *traits* especificados. Por ejemplo, el tipo `Pair <T>` en el
Listado 10-16 siempre implementa la función `new`. Pero `Pair <T>` solo
implementa el método `cmp_display` si su tipo interno `T` implementa el
*trait* `PartialOrd` que permite la comparación *y* el *trait* `Display` que
permite la impresión.

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

<span class="caption">Listado 10-16: Implementar métodos condicionalmente en
un tipo genérico dependiendo de los *trait bounds*</span>

También podemos implementar condicionalmente un *trait* para cualquier tipo
que implemente otro *trait*. Las implementaciones de un *trait* en cualquier
tipo que satisfaga los límites de *traits* se denominan
*blanket implementations* (*implementaciones generales*) y se usan ampliamente en la biblioteca estándar de Rust. Por
ejemplo, la biblioteca estándar implementa el *trait* `ToString` en cualquier
tipo que implemente el *trait* `Display`. El bloque `impl` en la biblioteca
estándar se ve similar a este código:

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

Debido a que la biblioteca estándar tiene esta implementación general,
podemos llamar al método `to_string` definido por el *trait* `ToString` en
cualquier tipo que implemente el *trait* `Display`. Por ejemplo, podemos
convertir enteros en sus correspondientes valores `String` como este porque
los enteros implementan `Display`:

```rust
let s = 3.to_string();
```

Las implementaciones generales aparecen en la documentación del *trait* en la sección “Implementadores”.

*Traits* and *trait bounds* nos permiten escribir código que usa parámetros
de tipo genérico para reducir la duplicación pero también especifica al
compilador que queremos que el tipo genérico tenga un comportamiento
particular. El compilador puede usar la información de *trait bound* para
verificar que todos los tipos concretos utilizados con nuestro código
proporcionen el comportamiento correcto. En lenguajes tipados dinámicamente,
obtendríamos un error en el tiempo de ejecución si llamamos a un método en un
tipo que el tipo no implementó. Pero Rust mueve estos errores en tiempo de
compilación, por lo que nos vemos obligados a solucionar los problemas antes
de que nuestro código pueda ejecutarse. Además, no tenemos que escribir
código que verifique el comportamiento en el tiempo de ejecución porque ya lo
hemos comprobado en tiempo de compilación. Al hacerlo, mejora el rendimiento
sin tener que renunciar a la flexibilidad de los genéricos.

Otro tipo de genérico que ya hemos estado usando se llama *lifetimes*. En
lugar de garantizar que un tipo tenga el comportamiento que queremos, los
tiempos de vida (*lifetimes*) garantizan que las referencias sean válidas siempre que lo necesitemos. Veamos cómo *lifetimes* hacen eso.
