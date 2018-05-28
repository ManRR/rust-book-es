## Advanced Traits

Primero cubrimos los *traits* en la sección “Traits: Definición del
comportamiento compartido” del Capítulo 10, pero al igual que en los
*lifetimes*, no discutimos los detalles más avanzados. Ahora que sabes más
sobre Rust, podemos entrar en detalles.

### Especificación de tipos de marcadores de posición en definiciones de *trait* con tipos asociados

*Associated types* (*Tipos asociados*) conectan un *placeholder* (*marcador
de posición*) de tipo con un *trait* tal que las definiciones del método del
*trait* pueden usar estos tipos de marcadores en sus firmas. El implementador
de un *trait* especificará el tipo concreto que se utilizará en el lugar de
este tipo para la implementación particular. De esta forma, podemos definir un *trait* que usa algunos tipos sin necesidad de saber exactamente qué tipos
son hasta que se implemente el *trait*.

Hemos descrito la mayoría de las características avanzadas de este capítulo
como raramente necesarias. Los tipos asociados están en algún punto
intermedio: se usan con más raramente que las funciones explicadas en el
resto del libro, pero más comúnmente que muchas de las otras características
que se analizan en este capítulo.

Un ejemplo de un *trait* con un tipo asociado es el *trait* `Iterator` que
proporciona la biblioteca estándar. El tipo asociado se denomina `Item` y
representa el tipo de valores sobre los que se está iterando el tipo que
implementa el atributo `Iterator`. En la sección "El *trait* de `Iterator` y
método `next`” del Capítulo 13, mencionamos que la definición del *trait* de
`Iterator` es como se muestra en el Listado 19-20.

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;
}
```

<span class="caption">Listado 19-20: La definición del *trait `Iterator` que
tiene un tipo asociado `Item`</span>

El tipo `Item` es un tipo de marcador de posición, y la definición del método
`next` muestra que devolverá valores de tipo `Option<Self::Item>`. Los
implementadores del *trait* `Iterator` especificarán el tipo concreto
para `Item`, y el método `next` devolverá un `Option` que contiene un valor
de ese tipo concreto.

Los tipos asociados pueden parecer un concepto similar a los genéricos, ya
que estos últimos nos permiten definir una función sin especificar qué tipos
puede manejar. Entonces, ¿por qué usar tipos asociados?.

Examinemos la diferencia entre los dos conceptos con un ejemplo del Capítulo 13 que implementa el *trait* `Iterator` en la estructura `Counter`. En el
listado 13-21, especificamos que el tipo `Item` era `u32`:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        // --snip--
```

Esta sintaxis parece comparable a la de los genéricos. Entonces, ¿por qué no
simplemente definir el *trait* `Iterator` con genéricos, como se muestra en
el Listado 19-21?

```rust
pub trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
```

<span class="caption">Listado 19-21: Definición hipotética del *trait*
`Iterator` usando genéricos</span>

La diferencia es que al usar genéricos, como en el listado 19-21, debemos
anotar los tipos en cada implementación; porque también podemos implementar
`Iterator<String> for Counter` o cualquier otro tipo, podríamos tener
múltiples implementaciones de `Iterator` para `Counter`. En otras palabras,
cuando un *trait* tiene un parámetro genérico, puede implementarse para un
tipo varias veces, cambiando los tipos concretos de los parámetros de tipo
genérico cada vez. Cuando utilizamos el método `next` en `Counter`,
deberíamos proporcionar anotaciones tipo para indicar qué implementación de
`Iterator` queremos usar.

Con los tipos asociados, no necesitamos anotar tipos porque no podemos
implementar un *trait* en un tipo varias veces. En el listado 19-20 con la
definición que usa tipos asociados, solo podemos elegir cuál será el tipo de
`Item` una vez, porque solo puede haber un `impl Iterator for Counter`. No es
necesario que especifiquemos que queremos un iterador de valores `u32` en
todas partes que llamemos `next` en `Counter`.

### Parámetros genéricos predeterminados y sobrecarga del operador

Cuando usamos parámetros genéricos, podemos especificar un tipo concreto
predeterminado para el tipo genérico. Esto elimina la necesidad de que los
implementadores del *trait* especifiquen un tipo concreto si el tipo
predeterminado funciona. La sintaxis para especificar un tipo predeterminado
para un tipo genérico es `<PlaceholderType=ConcreteType>` cuando se declara
el tipo genérico.

Un gran ejemplo de una situación donde esta técnica es útil es con la
sobrecarga del operador. *Sobrecarga del operador* (*Operator overloading*)
es la personalización del comportamiento de un operador (como `+`) en
situaciones particulares.

Rust no le permite crear sus propios operadores ni sobrecargar operadores
arbitrarios. Pero puede sobrecargar las operaciones y los *trait*
correspondientes listados en `std::ops` implementando los *trait*  asociados
con el operador. Por ejemplo, en el listado 19-22 sobrecargamos el operador
`+` para agregar dos instancias `Point` juntas. Hacemos esto implementando el
*trait*  `Add` en una estructura `Point`:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

fn main() {
    assert_eq!(Point { x: 1, y: 0 } + Point { x: 2, y: 3 },
               Point { x: 3, y: 3 });
}
```

<span class="caption">Listado 19-22: Implementando el *trait* `Add` para
sobrecargar el operador `+` para instancias `Point`</span>

El método `add` agrega los valores `x` de dos instancias `Point` y los
valores `y` de dos instancias `Point` para crear un `Point` nuevo. El *trait*
`Add` tiene un tipo asociado llamado `Output` que determina el tipo devuelto
por el método `add`.

El tipo genérico predeterminado en este código está dentro del *trait*
`Add`. Aquí está su definición:

```rust
trait Add<RHS=Self> {
    type Output;

    fn add(self, rhs: RHS) -> Self::Output;
}
```

Este código debería parecer generalmente familiar: un *trait* con un método y
un tipo asociado. La parte nueva es `RHS=Self`: esta sintaxis se llama
*parámetros de tipo por defecto*. El parámetro de tipo genérico `RHS`
(abreviatura de “right hand side”) define el tipo del parámetro `rhs` en el
método `add`. Si no especificamos un tipo concreto para `RHS` cuando
implementamos el *trait* `Add`, el tipo de `RHS` cambiará automáticamente a
`Self`, que será del tipo en el que estamos implementando `Add`.

Cuando implementamos `Add` para `Point`, usamos el valor predeterminado para
`RHS` porque queríamos agregar dos instancias `Point`. Veamos un ejemplo de
implementación del *trait* `Add` donde queremos personalizar el tipo `RHS` en
lugar de usar el predeterminado.

Tenemos dos estructuras, `Millimeters` y `Meters`, que contienen valores en
diferentes unidades. Queremos agregar valores en milímetros a valores en
metros y hacer que la implementación de `Add` haga la conversión
correctamente. Podemos implementar `Add` para `Millimeters` con `Meters` como
`RHS`, como se muestra en el Listado 19-23.

<span class="filename">Filename: src/lib.rs</span>

```rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
    type Output = Millimeters;

    fn add(self, other: Meters) -> Millimeters {
        Millimeters(self.0 + (other.0 * 1000))
    }
}
```

<span class="caption">Listado 19-23: Implementando el *trait* `Add` en
`Millimeters` para agregar `Millimeters` a `Meters`</span>

Para agregar `Millimeters` y `Meters`, especificamos `impl Add<Meters>` para
establecer el valor del parámetro de tipo `RHS` en lugar de usar el valor
predeterminado de `Self`.

Utilizará parámetros de tipo predeterminados de dos formas principales:

* Para extender un tipo sin romper el código existente
* Para permitir la personalización en casos específicos, la mayoría de los
 usuarios no necesitarán

El *trait* `Add` de la biblioteca estándar es un ejemplo del segundo
propósito: generalmente, agregará dos tipos similares, pero el *trait* `Add`
proporciona la capacidad de personalizar más allá de eso. El uso de un
parámetro de tipo predeterminado en la definición del  *trait* `Add`
significa que no tiene que especificar el parámetro extra la mayor parte del
tiempo. En otras palabras, no se necesita un poco de texto repetitivo de
implementación, lo que facilita el uso del *trait*.

El primer propósito es similar al segundo pero a la inversa: si desea agregar
un parámetro de tipo a un *trait* existente, puede otorgarle un valor
predeterminado para permitir la extensión de la funcionalidad del *trait* sin
romper el código de implementación existente.

### Sintaxis totalmente calificada para desambiguación: métodos de llamada con el mismo nombre

Nada en Rust impide que un *trait* tenga un método con el mismo nombre que el
de otro *trait*, ni Rust le impide implementar ambos *traits* en un solo
tipo. También es posible implementar un método directamente en el tipo con el
mismo nombre que los métodos de los *traits*.

Cuando llame a métodos con el mismo nombre, tendrá que decirle a Rust cuál
quiere usar. Considere el código en el listado 19-24 donde hemos definido dos
*traits*, `Pilot` y `Wizard`, que tienen un método llamado `fly`. Luego
implementamos ambos *traits* en un tipo `Human` que ya tiene implementado un método llamado `fly`. Cada método `fly` hace algo diferente.

<span class="filename">Filename: src/main.rs</span>

```rust
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("This is your captain speaking.");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Up!");
    }
}

impl Human {
    fn fly(&self) {
        println!("*waving arms furiously*");
    }
}
```

<span class="caption">Listado 19-24: Dos *traits* se definen para tener un
método `fly` y se implementan en el tipo `Human`, y un método `fly` que se
implementa en `Human` directamente</span>

Cuando llamamos `fly` en una instancia de `Human`, el compilador establece de
manera predeterminada el método que se implementa directamente en el tipo,
como se muestra en el Listado 19-25.

<span class="filename">Filename: src/main.rs</span>

```rust
# trait Pilot {
#     fn fly(&self);
# }
#
# trait Wizard {
#     fn fly(&self);
# }
#
# struct Human;
#
# impl Pilot for Human {
#     fn fly(&self) {
#         println!("This is your captain speaking.");
#     }
# }
#
# impl Wizard for Human {
#     fn fly(&self) {
#         println!("Up!");
#     }
# }
#
# impl Human {
#     fn fly(&self) {
#         println!("*waving arms furiously*");
#     }
# }
#
fn main() {
    let person = Human;
    person.fly();
}
```

<span class="caption">Listado 19-25: Llamar a `fly` en una instancia de
`Human`</span>

Al ejecutar este código se imprimirá `*waving arms furiously*`, mostrando que
Rust llamó directamente al método `fly` implementado en `Human`.

Para llamar a los métodos `fly` desde el *trait* `Pilot` o el *trait*
`Wizard`, necesitamos usar una sintaxis más explícita para especificar a qué
método `fly` nos referimos. El listado 19-26 demuestra esta sintaxis.

<span class="filename">Filename: src/main.rs</span>

```rust
# trait Pilot {
#     fn fly(&self);
# }
#
# trait Wizard {
#     fn fly(&self);
# }
#
# struct Human;
#
# impl Pilot for Human {
#     fn fly(&self) {
#         println!("This is your captain speaking.");
#     }
# }
#
# impl Wizard for Human {
#     fn fly(&self) {
#         println!("Up!");
#     }
# }
#
# impl Human {
#     fn fly(&self) {
#         println!("*waving arms furiously*");
#     }
# }
#
fn main() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
```

<span class="caption">Listado 19-26: Especificando el método `fly` de ese
*trait* que queremos llamar</span>

Especificando el nombre del *trait* antes de que el nombre del método aclare
a Rust qué implementación de `fly` queremos llamar. También podríamos
escribir `Human::fly(&person)`, que es equivalente a `person.fly()` que
usamos en el listado 19-26, pero esto es un poco más largo de escribir si no
necesitamos desambiguar.

Al ejecutar este código, se imprime lo siguiente:

```text
This is your captain speaking.
Up!
*waving arms furiously*
```

Debido a que el método `fly` toma un parámetro `self`, si tuviéramos dos
*tipos* que implementaran un *trait*, Rust podría averiguar qué
implementación de un *trait* utilizar en función del tipo de `self`.

Sin embargo, las funciones asociadas que son parte de *trait* no tienen un
parámetro `self`. Cuando dos tipos en el mismo ámbito implementan ese *trait*
Rust no puede determinar a qué tipo se refiere a menos que use la
*sintaxis completa*. Por ejemplo, el *trait* `Animal` en el Listado 19-27
tiene la función asociada `baby_name`, la implementación de `Animal` para la
estructura `Dog`, y la función asociada `baby_name` definida en `Dog`
directamente.

<span class="filename">Filename: src/main.rs</span>

```rust
trait Animal {
    fn baby_name() -> String;
}

struct Dog;

impl Dog {
    fn baby_name() -> String {
        String::from("Spot")
    }
}

impl Animal for Dog {
    fn baby_name() -> String {
        String::from("puppy")
    }
}

fn main() {
    println!("A baby dog is called a {}", Dog::baby_name());
}
```

<span class="caption">Listado 19-27: Un *trait* con una función asociada y un
tipo con una función asociada del mismo nombre que también implementa el
*trait*</span>

Este código es para un refugio de animales que quiere nombrar a todos los
cachorros *Spot*, que se implementa en la función asociada `baby_name` que se
define en `Dog`. El tipo `Dog` también implementa el *trait* `Animal`, que
describe las características que tienen todos los animales. Los perros bebé
se llaman cachorros, y eso se expresa en la implementación del *trait*
`Animal` en `Dog` en la función `baby_name` asociada con el *trait* `Animal`.

En `main`, llamamos a la función `Dog::baby_name`, que llama directamente a
la función asociada definida en `Dog`. Este código imprime lo siguiente:

```text
A baby dog is called a Spot
```

Este resultado no es lo que queríamos. Queremos llamar a la función
`baby_name` que forma parte del *trait* `Animal` que implementamos en `Dog`,
por lo que el código se imprime `A baby dog is called a puppy`. La técnica de
especificar el nombre del *trait* que utilizamos en el listado 19-26 no ayuda
aquí; si cambiamos `main` al código en el Listado 19-28, obtendremos un error
de compilación.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    println!("A baby dog is called a {}", Animal::baby_name());
}
```

<span class="caption">Listado 19-28: Intentando llamar a la función
`baby_name` del *trait* `Animal`, pero Rust no sabe qué implementación
usar</span>

Como `Animal::baby_name` es una función asociada más que un método, y por lo
tanto no tiene un parámetro `self`, Rust no puede determinar qué
implementación de `Animal::baby_name` queremos. Obtendremos este error de
compilación:

```text
error[E0283]: type annotations required: cannot resolve `_: Animal`
  --> src/main.rs:20:43
   |
20 |     println!("A baby dog is called a {}", Animal::baby_name());
   |                                           ^^^^^^^^^^^^^^^^^
   |
   = note: required by `Animal::baby_name`
```

Para desambiguar y decirle a Rust que queremos usar la implementación de
`Animal` para `Dog`, necesitamos usar una sintaxis totalmente calificada. El
listado 19-29 demuestra cómo utilizar la sintaxis totalmente calificada.

<span class="filename">Filename: src/main.rs</span>

```rust
# trait Animal {
#     fn baby_name() -> String;
# }
#
# struct Dog;
#
# impl Dog {
#     fn baby_name() -> String {
#         String::from("Spot")
#     }
# }
#
# impl Animal for Dog {
#     fn baby_name() -> String {
#         String::from("puppy")
#     }
# }
#
fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```

<span class="caption">Listado 19-29: Usar la sintaxis totalmente calificada
para especificar que queremos llamar a la función `baby_name` del *trait*
`Animal` tal como se implementó en `Dog`</span>

Le proporcionamos a Rust una anotación de tipo dentro de los corchetes
angulares, lo que indica que queremos llamar al método `baby_name` del
*trait* `Animal` implementado en `Dog` diciendo que queremos tratar el tipo
`Dog` como un `Animal` para esta llamada de función. Este código ahora
imprimirá lo que queremos:

```text
A baby dog is called a puppy
```

En general, la sintaxis totalmente calificada se define de la siguiente
manera:

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

Para las funciones asociadas, no habría un `receiver`: solo habría una lista
de otros argumentos. Puede usar sintaxis totalmente calificada en todas
partes a las que llame funciones o métodos. Sin embargo, puede omitir
cualquier parte de esta sintaxis que Rust pueda deducir de otra información
en el programa. Solo necesita usar esta sintaxis más detallada en los casos
en que hay varias implementaciones que usan el mismo nombre y Rust necesita
ayuda para identificar a qué implementación desea llamar.

### Usar *Supertraits* para exigir la funcionalidad de un *Trait* dentro de otro *Trait*

Algunas veces, puede necesitar un *trait* para usar la funcionalidad de otro
*trait*. En este caso, debe confiar en que los *traits* dependientes también
se están implementando. El *trait* en el que confía es un *supertrait* del
*trait* que está implementando.

Por ejemplo, supongamos que queremos crear un *trait* `OutlinePrint` con un
método `outline_print` que imprimirá un valor enmarcado en asteriscos. Es
decir, dada una estructura `Point` que implementa `Display` para dar como
resultado `(x, y)`, cuando llamamos a `outline_print` en una instancia
`Point` que tiene `1` para `x` y `3` para `y`, debe imprimir lo siguiente:

```text
**********
*        *
* (1, 3) *
*        *
**********
```

En la implementación de `outline_print`, queremos usar la funcionalidad del
*trait* `Display`. Por lo tanto, debemos especificar que el *trait*
`OutlinePrint` solo funcionará para los tipos que también implementen
`Display` y proporcionen la funcionalidad que `OutlinePrint` necesita.
Podemos hacer eso en la definición de *trait* especificando
`OutlinePrint: Display`. Esta técnica es similar a agregar un *trait* ligado
al *trait*. El listado 19-30 muestra una implementación del *trait*
`OutlinePrint`.

<span class="filename">Filename: src/main.rs</span>

```rust
use std::fmt;

trait OutlinePrint: fmt::Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

<span class="caption">Listado 19-30: Implementando el *trait* `OutlinePrint`
que requiere la funcionalidad de `Display`</span>

Debido a que hemos especificado que `OutlinePrint` requiere el *trait*
`Display`, podemos usar la función `to_string` que se implementa
automáticamente para cualquier tipo que implemente `Display`. Si intentamos
usar `to_string` sin agregar dos puntos y especificando el *trait* `Display`
después del nombre del *trait*, obtendríamos un error al decir que no se
encontró ningún método llamado `to_string` para el tipo `&Self` en el alcance
actual .

Veamos qué sucede cuando tratamos de implementar `OutlinePrint` en un tipo
que no implementa `Display`, como la estructura `Point`:

<span class="filename">Filename: src/main.rs</span>

```rust
# trait OutlinePrint {}
struct Point {
    x: i32,
    y: i32,
}

impl OutlinePrint for Point {}
```

Recibimos un error al decir que `Display` es obligatorio pero no implementado:

```text
error[E0277]: the trait bound `Point: std::fmt::Display` is not satisfied
  --> src/main.rs:20:6
   |
20 | impl OutlinePrint for Point {}
   |      ^^^^^^^^^^^^ `Point` cannot be formatted with the default formatter;
try using `:?` instead if you are using a format string
   |
   = help: the trait `std::fmt::Display` is not implemented for `Point`
```

Para solucionar esto, implementamos `Display` en `Point` y satisfacemos la
restricción que `OutlinePrint` requiere, así:

<span class="filename">Filename: src/main.rs</span>

```rust
# struct Point {
#     x: i32,
#     y: i32,
# }
#
use std::fmt;

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```

Luego, la implementación del *trait* `OutlinePrint` en `Point` se compilará
correctamente, y podemos llamar `outline_print` en una instancia `Point` para
mostrarlo dentro de un contorno de asteriscos.

### Uso del patrón *Newtype* para implementar rasgos externos en tipos externos

En el Capítulo 10 en la sección “Implementación de un *trait* en un tipo”,
mencionamos la regla huérfana que establece que podemos implementar un
*trait* en un tipo, siempre que el *trait* o el tipo sean locales para
nuestro *crate*. Es posible sortear esta restricción usando el
*newtype pattern*, que implica la creación de un nuevo tipo en una estructura
tuple. (Cubrimos las estructuras de tuplas en la sección “Usar estructuras
*Tuple* sin *Named Fields* para crear diferentes tipos” del Capítulo 5). La
estructura de tuplas tendrá un campo y será un envoltorio del tipo para el
que queremos implementar un *trait*. Entonces el tipo de envoltura es local
para nuestro *crate*, y podemos implementar el *trait* en el envoltorio.
*Newtype* es un término que se origina del lenguaje de programación Haskell.
No hay penalización de rendimiento en el tiempo de ejecución para usar este
patrón, y el tipo de envoltura se elimina en el momento de la compilación.

Como ejemplo, digamos que queremos implementar `Display` en `Vec<T>`, que la
regla huérfana nos impide hacer directamente porque el *trait* `Display` y el
tipo `Vec<T>` se definen fuera de nuestro *crate* . Podemos hacer una
estructura `Wrapper` que contenga una instancia de `Vec<T>`; luego podemos
implementar `Display` en `Wrapper` y usar el valor `Vec<T>`, como se muestra
en el Listado 19-31.

<span class="filename">Filename: src/main.rs</span>

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

<span class="caption">Listado 19-31: Crear un tipo `Wrapper` alrededor d
`Vec<String>` para implementar `Display`</span>

La implementación de `Display` usa `self.0` para acceder al `Vec<T>` interno,
porque `Wrapper` es una estructura *tuple* y `Vec<T>`es el ítem en el índice
0 en la tupla. Entonces podemos usar la funcionalidad del tipo `Display` en
`Wrapper`.

La desventaja de usar esta técnica es que `Wrapper` es un tipo nuevo, por lo
que no tiene los métodos del valor que tiene. Tendríamos que implementar
todos los métodos de `Vec<T>` directamente en `Wrapper` de manera que los
métodos deleguen en `self.0`, lo que nos permitiría tratar `Wrapper`
exactamente como `Vec<T>`. Si quisiéramos que el nuevo tipo tuviera todos los
métodos del tipo interno, implementando el *trait* `Deref` (discutido en el
Capítulo 15 en la sección “Tratar punteros inteligentes como referencias
regulares con el *Trait* `Deref`”) en el `Wrapper` para devolver el tipo
interno sería una solución. Si no queremos que el tipo `Wrapper` tenga todos
los métodos del tipo interno, por ejemplo, para restringir el comportamiento
del tipo `Wrapper`, tendríamos que implementar solo los métodos que queremos
manualmente.

Ahora ya sabes cómo se usa el patrón de tipo nuevo en relación con los
*traits*; también es un patrón útil incluso cuando los *traits* no están
involucrados. Cambiemos el enfoque y veamos algunas formas avanzadas de
interactuar con el sistema de tipos de Rust.
