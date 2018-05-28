## Usando `Box <T>` para apuntar a los datos en el Heap

El puntero inteligente más sencillo es un *box*, cuyo tipo está escrito
`Box <T>`. Los *Boxes* le permiten almacenar datos en el montículo en lugar
de en la pila. Lo que queda en la pila es el puntero a los datos del
montículo. Consulte el Capítulo 4 para revisar la diferencia entre la pila y
el montículo.

Los *Boxes*  no tienen sobrecarga de rendimiento, aparte de almacenar sus
datos en el montículo en lugar de en la pila. Pero tampoco tienen muchas
capacidades adicionales. Los usarás más a menudo en estas situaciones:

* Cuando tiene un tipo cuyo tamaño no se puede conocer en el momento de la
 compilación y desea usar un valor de ese tipo en un contexto que requiere un
 tamaño exacto
* Cuando tiene una gran cantidad de datos y desea transferir la propiedad,
 pero asegúrese de que los datos no se copiarán cuando lo haga
* Cuando desea poseer un valor y solo le importa que sea un tipo que
 implemente un rasgo particular en lugar de ser de un tipo específico

Demostraremos la primera situación en la sección “Habilitación de tipos
recursivos con *Boxes*”. En el segundo caso, transferir la propiedad de una
gran cantidad de datos puede llevar mucho tiempo porque los datos se copian
en la pila. Para mejorar el rendimiento en esta situación, podemos almacenar
la gran cantidad de datos en el montículo en una *box*. Luego, solo se copia
la pequeña cantidad de datos del puntero en la pila, mientras que los datos a
los que hace referencia permanecen en un lugar en el montículo. El tercer
caso se conoce como *trait object*, y el capítulo 17 dedica una sección
completa, “Usar *trait object* que permiten valores de diferentes tipos”,
solo para ese tema. ¡Entonces, lo que aprendes aquí lo aplicará nuevamente en
el Capítulo 17!.

### Usando un `Box<T>` para almacenar datos en el montículo

Antes de analizar este caso de uso para `Box <T>`, cubriremos la sintaxis y
cómo interactuar con los valores almacenados dentro de `Box <T>`.

El listado 15-1 muestra cómo usar un *box* para almacenar un valor `i32` en el montículo:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```

<span class="caption">Listado 15-1: Almacenamiento de un valor `i32` en el
montículo usando un *box*</span>

Definimos la variable `b` para que tenga el valor de un `Box` que apunta al
valor `5`, que se asigna en el montículo. Este programa imprimirá `b = 5`; en
este caso, podemos acceder a los datos en el *box* de forma similar a como lo
haríamos si estos datos estuvieran en la pila. Al igual que cualquier valor
propio, cuando un *box* sale del alcance, como `b` lo hace al final de `main`
será desasignado. La desasignación ocurre para el *box* (almacenada en la
pila) y los datos a los que apunta (almacenados en el montículo).

Poner un solo valor en el montículo no es muy útil, por lo que no utilizará
los *boxes* por sí mismo de esta manera muy a menudo. Tener valores como un
solo `i32` en la pila, donde están almacenados por defecto, es más apropiado
en la mayoría de las situaciones. Veamos un caso en el que los *boxes* nos
permiten definir tipos que no se nos permitirían si no tuviéramos *boxes*.

### Habilitación de tipos recursivos con *Boxes*

En tiempo de compilación, Rust necesita saber cuánto espacio ocupa un tipo.
Un tipo cuyo tamaño no se puede conocer en el momento de la compilación es un
*tipo recursivo*, donde un valor puede tener como parte de sí mismo otro
valor del mismo tipo. Debido a que este anidamiento de valores podría,
teóricamente, continuar infinitamente, Rust no sabe cuánto espacio necesita
un valor de tipo recursivo. Sin embargo, los *boxes* tienen un tamaño
conocido, por lo que al insertar un *box* en una definición de tipo recursivo
puede tener tipos recursivos.

Exploremos la *cons list*, que es un tipo de datos común en los lenguajes de
programación funcionales, como un ejemplo de tipo recursivo. El tipo *cons
list* que definiremos es sencillo excepto por la recursión; por lo tanto, los
conceptos en el ejemplo con el que trabajaremos serán útiles cada vez que
ingrese en situaciones más complejas que involucren tipos recursivos.

#### Más información sobre el *Cons List*

Un *cons list* es una estructura de datos que proviene del lenguaje de
programación Lisp y sus dialectos en Lisp, la función `cons`
(abreviatura de “construct function”)
construye un nuevo par a partir de sus dos argumentos, que generalmente son
un valor único y otro par estos pares que contienen pares forman una lista.

El concepto de *cons function* se ha convertido en una jerga de programación funcional más general: “to cons *x* onto *y*” informalmente significa
construir una nueva instancia de contenedor poniendo el elemento *x* al
comienzo de este nuevo contenedor, seguido por el contenedor *y*.

Cada elemento en una *cons list* contiene dos elementos: el valor del
elemento actual y el siguiente ítem. El último elemento de la lista contiene
solo un valor llamado `Nil` sin un siguiente artículo. Una *cons list*
se produce recursivamente llamando a la función `cons`. El nombre canónico
para denotar el caso base de la recursión es `Nil`.
Tenga en cuenta que esto no es lo mismo que el concepto “null” o “nil” en el
Capítulo 6, que es un valor inválido o ausente.

Aunque los lenguajes de programación funcionales usan *cons list*
frecuentemente, las *cons list* no eson una estructura de datos comúnmente
utilizada en Rust. La mayoría de las veces cuando tiene una lista de
elementos en Rust, `Vec <T>` es una mejor opción para usar. Otros tipos de
datos recursivos más complejos *son* útiles en diversas situaciones, pero
comenzando con la *cons list*, podemos explorar cómo los *boxes* nos permiten definir un tipo de datos recursivo sin mucha distracción.

El listado 15-2 contiene una definición enum para una *cons list*. Tenga en
cuenta que este código aún no se compilará porque el tipo `List` no tiene un
tamaño conocido, que demostraremos

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
enum List {
    Cons(i32, List),
    Nil,
}
```

<span class="caption">Listado 15-2: El primer intento de definir una
enumeración para representar una estructura de datos de *cons list* de
valores `i32`</span>

> Nota: Estamos implementando una *cons list* que solo contiene valores `i32`
> para los propósitos de este ejemplo. Podríamos haberlo implementado usando
> genéricos, como discutimos en el Capítulo 10, para definir un tipo de
> *cons list* que pudiera almacenar valores de cualquier tipo.

Usar el tipo `List` para almacenar la lista `1, 2, 3` se vería como el código
en Listado 15-3:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use List::{Cons, Nil};

fn main() {
    let list = Cons(1, Cons(2, Cons(3, Nil)));
}
```

<span class="caption">Listado 15-3: Usando la enumeración `List` para
almacenar la lista `1,2, 3`</span>

El primer valor de `Cons` tiene `1` y otro valor de `List`. Este valor de
`List` es otro valor `Cons` que contiene `2` y otro valor `List`. Este valor
de `List` es un valor más `Cons` que contiene `3` y un valor `List`, que
finalmente es `Nil`, la variante no recursiva que señala el final de la lista.

Si tratamos de compilar el código en el Listado 15-3, obtenemos el error que
se muestra en el Listado 15-4:

```text
error[E0072]: recursive type `List` has infinite size
 --> src/main.rs:1:1
  |
1 | enum List {
  | ^^^^^^^^^ recursive type has infinite size
2 |     Cons(i32, List),
  |               ----- recursive without indirection
  |
  = help: insert indirection (e.g., a `Box`, `Rc`, or `&`) at some point to
  make `List` representable
```

<span class="caption">Listado 15-4: El error que obtenemos al intentar
definir una enumeración recursiva</span>

El error muestra que este tipo “tiene un tamaño infinito”. La razón es que
hemos definido `List` con una variante que es recursiva: tiene otro valor de
sí mismo directamente. Como resultado, Rust no puede determinar cuánto
espacio necesita para almacenar un valor de `List`. Analicemos por qué
tenemos este error un poco. Primero, veamos cómo Rust decide cuánto espacio
necesita para almacenar un valor de tipo no recursivo.

#### Calcular el tamaño de un tipo no recursivo

Recuerde la enumeración `Message` que definimos en el Listado 6-2 cuando
discutimos las definiciones de enum en el Capítulo 6:

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

Para determinar cuánto espacio asignar para un valor de `Message`, Rust
revisa cada una de las variantes para ver qué variante necesita más espacio.
Rust ve que `Message::Quit` no necesita ningún espacio, `Message::Move`
necesita suficiente espacio para almacenar dos valores `i32`, y así
sucesivamente. Debido a que solo se usará una variante, la mayor cantidad de
espacio que necesitará un valor de `Message` es el espacio que le llevaría
almacenar la mayor de sus variantes.

En contraste esto con lo que ocurre cuando Rust intenta determinar cuánto
espacio necesita un tipo recursivo como la enumeración `List` en el Listado
15-2. El compilador comienza mirando la variante `Cons`, que contiene un
valor de tipo `i32` y un valor de tipo `List`. Por lo tanto, `Cons` necesita
una cantidad de espacio igual al tamaño de un `i32` más el tamaño de una
`List`. Para averiguar cuánta memoria necesita el tipo `List`, el compilador
observa las variantes, comenzando con la variante `Cons`. La variante `Cons`
contiene un valor de tipo `i32` y un valor de tipo `List`, y este proceso
continúa infinitamente, como se muestra en la figura 15-1.

<img alt="An infinite Cons list" src="img/trpl15-01.svg" class="center" style="width: 50%;" />

<span class="caption">Figura 15-1: Una `List` infinita que consiste en
infinitas variantes `Cons`</span>

#### Usando `Box <T>` para obtener un tipo recursivo con un tamaño conocido

Rust no puede determinar cuánto espacio asignar para los tipos definidos
recursivamente, por lo que el compilador da el error en el Listado 15-4. Pero
el error incluye esta útil sugerencia:

```text
  = help: insert indirection (e.g., a `Box`, `Rc`, or `&`) at some point to
  make `List` representable
```

En esta sugerencia, “indirection” significa que en lugar de almacenar un
valor directamente, cambiaremos la estructura de datos para almacenar el
valor indirectamente almacenando un puntero al valor en su lugar.

Debido a que `Box<T>` es un puntero, Rust siempre sabe cuánto espacio
necesita `Box<T>`: el tamaño de un puntero no cambia en función de la
cantidad de datos a los que apunta. Esto significa que podemos poner
`Box<T>` dentro de la variante `Cons` en lugar de otro `List` directamente.
El `Box <T>` apuntará al siguiente valor `List` que estará en el montículo en
lugar de dentro de la variante `Cons`. Conceptualmente, todavía tenemos una
lista, creada con listas que “sostienen” otras listas, pero esta
implementación ahora es más como colocar los elementos uno al lado del otro
en lugar de uno dentro del otro.

Podemos cambiar la definición de la enumeración `List` en el Listado 15-2 y
el uso de la `List` en el Listado 15-3 del código en el Listado 15-5, que
compilará:

<span class="filename">Filename: src/main.rs</span>

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let list = Cons(1,
        Box::new(Cons(2,
            Box::new(Cons(3,
                Box::new(Nil))))));
}
```

<span class="caption">Listado 15-5: Definición de `List` que usa
`Box<T>` para tener un tamaño conocido</span>

La variante `Cons` necesitará el tamaño de un `i32` más el espacio para
almacenar los datos del puntero de la *box*. La variante `Nil` no almacena
valores, por lo que necesita menos espacio que la variante `Cons`. Ahora
sabemos que cualquier valor de `List` tomará el tamaño de un `i32` más el
tamaño de los datos del puntero de un *box*. Al usar una *box*, hemos roto la
cadena infinita y recursiva, por lo que el compilador puede calcular el
tamaño que necesita para almacenar un valor de `List`. La figura 15-2 muestra
cómo es la variante `Cons` ahora.

<img alt="A finite Cons list" src="img/trpl15-02.svg" class="center" />

<span class="caption">Figura 15-2: Una  `List` que no tiene un tamaño
infinito porque `Cons` contiene un `Box`</span>

Las casillas solo proporcionan la asignación indirecta y del montículo; no
tienen otras capacidades especiales, como las que veremos con los otros tipos
de punteros inteligentes. Tampoco tienen ninguna sobrecarga de rendimiento en
la que incurran estas capacidades especiales, por lo que pueden ser útiles en
casos como la *cons list*, donde la indirección es la única característica
que necesitamos. Veremos más casos de uso para las *boxes* en el Capítulo 17,
también.

El tipo `Box<T>` es un puntero inteligente porque implementa el *trait*
`Deref`, que permite que los valores `Box<T>`sean tratados como referencias.
Cuando un valor `Box<T>` sale del alcance, los datos del montículo a los que
apunta el *box* también se limpian debido a la implementación del *trait*
`Drop`. Exploremos estos dos *traits* con más detalle. Estos dos *traits*
serán aún más importantes para la funcionalidad proporcionada por los otros
tipos de punteros inteligentes que discutiremos en el resto de este capítulo.
