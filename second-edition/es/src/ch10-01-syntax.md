## Tipos de datos genéricos

Podemos usar genéricos para crear definiciones para elementos como firmas de
funciones o estructuras, que luego podemos usar con muchos tipos de datos
concretos diferentes. Primero veamos cómo definir funciones, estructuras,
enumeraciones y métodos usando genéricos. Luego discutiremos cómo los
genéricos afectan el rendimiento del código.

### En Definiciones de funciones

Al definir una función que usa genéricos, colocamos los genéricos en la firma
de la función en la que normalmente especificamos los tipos de datos de los
parámetros y el valor de retorno. Hacerlo hace que nuestro código sea más
flexible y proporciona más funcionalidad a los llamantes de nuestra función a
la vez que evita la duplicación de código.

Continuando con nuestra función `largest`, el Listado 10-4 muestra dos
funciones que encuentran el valor más grande en un *slice*.

<span class="filename">Filename: src/main.rs</span>

```rust
fn largest_i32(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn largest_char(list: &[char]) -> char {
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

    let result = largest_i32(&number_list);
    println!("The largest number is {}", result);
#    assert_eq!(result, 100);

    let char_list = vec!['y', 'm', 'a', 'q'];

    let result = largest_char(&char_list);
    println!("The largest char is {}", result);
#    assert_eq!(result, 'y');
}
```

<span class="caption">Listado 10-4: dos funciones que difieren solo en sus
nombres y los tipos en sus firmas</span>

La función `largest_i32` es la que extrajimos en el Listado 10-3 que
encuentra el `i32` más grande en una porción. La función `largest_char`
encuentra el `char` más grande en una porción. Los cuerpos de función tienen
el mismo código, así que eliminemos la duplicación introduciendo un parámetro
de tipo genérico en una sola función.

Para parametrizar los tipos en la nueva función que definiremos, necesitamos
nombrar el parámetro tipo, tal como lo hacemos para los parámetros de valor
de una función. Puede usar cualquier identificador como nombre de parámetro
de tipo. Pero usaremos `T` porque, por convención, los nombres de los
parámetros en Rust son cortos, a menudo solo una letra, y la convención de
nombres de Rust es CamelCase. Abreviatura de "tipo", `T` es la opción
predeterminada de la mayoría de los programadores de Rust.

Cuando usamos un parámetro en el cuerpo de la función, tenemos que declarar
el nombre del parámetro en la firma para que el compilador sepa lo que
significa ese nombre. De forma similar, cuando usamos un nombre de parámetro
de tipo en una firma de función, tenemos que declarar el nombre del parámetro
de tipo antes de usarlo. Para definir la función genérica `largest`,
coloque declaraciones de nombre de tipo dentro de corchetes angulares,`<>`,
entre el nombre de la función y la lista de parámetros, como esta:

```rust,ignore
fn largest<T>(list: &[T]) -> T {
```

Leemos esta definición como: la función `largest` es genérica sobre algún
tipo `T`. Esta función tiene un parámetro llamado `list`, que es un segmento
de valores de tipo `T`. La función `largest` devolverá un valor del mismo
tipo `T`.

El listado 10-5 muestra la definición de función "más grande" combinada que
utiliza el tipo de datos genéricos en su firma. La lista también muestra cómo
podemos llamar a la función con una porción de valores `i32` o `char`. Tenga
en cuenta que este código aún no se compilará, pero lo solucionaremos más
adelante en este capítulo.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn largest<T>(list: &[T]) -> T {
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

<span class="caption">Listado 10-5: Una definición de la función `largest`
que usa parámetros de tipo genérico pero aún no compila</span>

Si compilamos este código ahora, obtendremos este error:

```text
error[E0369]: binary operation `>` cannot be applied to type `T`
 --> src/main.rs:5:12
  |
5 |         if item > largest {
  |            ^^^^^^^^^^^^^^
  |
  = note: an implementation of `std::cmp::PartialOrd` might be missing for `T`
```

La nota menciona `std::cmp::PartialOrd`, que es un *trait*. Hablaremos sobre
los *trait* en la siguiente sección. Por ahora, este error indica que el
cuerpo de `largest` no funcionará para todos los tipos posibles que `T`
podría ser. Como queremos comparar valores del tipo `T` en el cuerpo, solo
podemos usar tipos cuyos valores se puedan ordenar. Para permitir las
comparaciones, la biblioteca estándar tiene el *trait* `std::cmp::PartialOrd`
que puede implementar en los tipos (consulte el Apéndice C para obtener más
información sobre este *trait*). Aprenderá cómo especificar que un tipo
genérico tiene un *trait* particular en la sección “Límites de
características”, pero primero exploremos otras maneras de usar parámetros
genéricos de tipo.

### En las definiciones de *Struct*

También podemos definir estructuras para usar un parámetro de tipo genérico
en uno o más campos usando la sintaxis `<>`. El Listado 10-6 muestra cómo
definir una estructura `Point <T>` para contener valores de coordenadas `x`
y `y` de cualquier tipo.

<span class="filename">Filename: src/main.rs</span>

```rust
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer = Point { x: 5, y: 10 };
    let float = Point { x: 1.0, y: 4.0 };
}
```

<span class="caption">Listing 10-6: Una estructura `Point <T>` que contiene los valores `x` y `y` de tipo `T`</span>

La sintaxis para usar genéricos en las definiciones de estructuras es similar a la utilizada en las definiciones de funciones. Primero, declaramos el nombre del parámetro tipo dentro de corchetes angulares justo después del nombre de la estructura. Entonces podemos usar el tipo genérico en la definición de estructura donde especificaríamos tipos de datos concretos.

Tenga en cuenta que debido a que hemos usado solo un tipo genérico para definir `Point <T>`, esta definición dice que la estructura `Point <T>` es genérica sobre algún tipo `T`, y los campos `x` y `y` son *ambos* del mismo tipo, cualquiera que sea ese tipo. Si creamos una instancia de un `Punto <T>` que tiene valores de diferentes tipos, como en el Listado 10-7, nuestro código no se compilará.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let wont_work = Point { x: 5, y: 4.0 };
}
```

<span class="caption">Listado 10-7: Los campos `x` y `y` deben ser del mismo
tipo porque ambos tienen el mismo tipo de datos genéricos `T`.</span>

En este ejemplo, cuando asignamos el valor entero 5 a `x`, dejamos que el
compilador sepa que el tipo genérico `T` será un entero para esta instancia
de `Point <T>`. Luego, cuando especifiquemos 4.0 para `y`, que hemos definido
para que tenga el mismo tipo que `x`, obtendremos un error de desajuste de
tipo como este:

```text
error[E0308]: mismatched types
 --> src/main.rs:7:38
  |
7 |     let wont_work = Point { x: 5, y: 4.0 };
  |                                      ^^^ expected integral variable, found
floating-point variable
  |
  = note: expected type `{integer}`
             found type `{float}`
```

Para definir una estructura `Point` donde `x` y `y` son ambos genéricos pero
podrían tener diferentes tipos, podemos usar múltiples parámetros genéricos
de tipo. Por ejemplo, en el listado 10-8, podemos cambiar la definición de
`Point` para que sea genérica sobre los tipos` T` y `U` donde `x` es de tipo
`T` y `y` es de tipo `U` .

<span class="filename">Filename: src/main.rs</span>

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

fn main() {
    let both_integer = Point { x: 5, y: 10 };
    let both_float = Point { x: 1.0, y: 4.0 };
    let integer_and_float = Point { x: 5, y: 4.0 };
}
```

<span class="caption">Listado 10-8: Un `Point <T, U>` genérico sobre dos
tipos para que `x` y `y` puedan ser valores de diferentes tipos</span>

¡Ahora todas las instancias de `Point` se muestran permitidas! Puede usar
tantos parámetros de tipo genérico como desee en una definición, pero usar
más de unos pocos hace que su código sea difícil de leer. Cuando necesite
muchos tipos genéricos en su código, podría indicar que su código necesita
una reestructuración en piezas más pequeñas.

### En Enum Definiciones

Como hicimos con las estructuras, podemos definir las enumeraciones para
mantener los tipos de datos genéricos en sus variantes. Echemos otro vistazo
a la enumeración `Opción <T>` que proporciona la biblioteca estándar, que
usamos en el Capítulo 6:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Esta definición ahora debería tener más sentido para usted. Como puede ver,
`Option <T>` es una enumeración que es genérica sobre el tipo `T` y tiene dos
variantes: `Some`, que contiene un valor de tipo `T`, y una `None` variante
que no tiene ningún valor. Al utilizar la enumeración `Opción <T>`, podemos
expresar el concepto abstracto de tener un valor opcional, y como
`Opción <T>` es genérico, podemos usar esta abstracción sin importar el tipo
de valor opcional.

Los *enums* también pueden usar múltiples tipos genéricos. La definición de
la enumeración `Result` que utilizamos en el Capítulo 9 es un ejemplo:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

La enumeración `Result` es genérica sobre dos tipos, `T` y `E`, y tiene dos
variantes: `Ok`, que contiene un valor de tipo `T`, y `Err`, que contiene un
valor de tipo `E`. Esta definición hace que sea conveniente usar la
enumeración `Result` en cualquier lugar donde tengamos una operación que
pueda tener éxito (devuelva un valor de algún tipo `T`) o que falle
(devuelva un error de algún tipo `E`). De hecho, esto es lo que usamos para
abrir un archivo en el Listado 9-3, donde `T` se completó con el tipo
`std::fs::File` cuando el archivo se abrió correctamente y `E` se completó
con el tipo `std::io::Error` cuando hubo problemas al abrir el archivo.

Cuando reconoce situaciones en su código con múltiples definiciones *struct*
o *enum* que difieren solo en los tipos de los valores que contienen, puede
evitar la duplicación mediante el uso de tipos genéricos.

### En definiciones de métodos

Podemos implementar métodos en estructuras y enumeraciones (como lo hicimos
en el Capítulo 5) y también usar tipos genéricos en sus definiciones. El
Listado 10-9 muestra la estructura `Point <T>` que definimos en el Listado
10-6 con un método llamado `x` implementado en él.

<span class="filename">Filename: src/main.rs</span>

```rust
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

fn main() {
    let p = Point { x: 5, y: 10 };

    println!("p.x = {}", p.x());
}
```

<span class="caption">Listado 10-9: Implementando un método llamado `x` en la
estructura `Point <T>` que devolverá una referencia al campo `x` de tipo
`T`</span>

Aquí, hemos definido un método llamado `x` en `Point <T>` que devuelve una
referencia a los datos en el campo `x`.

Tenga en cuenta que tenemos que declarar `T` justo después de `impl` para que
podamos usarlo para especificar que estamos implementando métodos en el tipo
`Point <T>`. Al declarar `T` como un tipo genérico después de `impl`, Rust
puede identificar que el tipo en los paréntesis angulares en `Point` es un
tipo genérico en lugar de un tipo concreto.

Podríamos, por ejemplo, implementar métodos solo en instancias `Point <f32>`
en lugar de en instancias `Point <T>` con cualquier tipo genérico. En el
listado 10-10 usamos el tipo concreto `f32`, lo que significa que no
declaramos ningún tipo después de `impl`.

```rust
# struct Point<T> {
#     x: T,
#     y: T,
# }
#
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

<span class="caption">Listado 10-10: Un bloque `impl` que solo se aplica a
una estructura con un tipo concreto para el parámetro de tipo
genérico `T`</span>

Este código significa que el tipo `Point <f32>` tendrá un método llamado
`distance_from_origin` y otras instancias de `Point <T>` donde `T` no es del
tipo `f32` no tendrá este método definido. El método mide qué tan lejos está
nuestro punto del punto en coordenadas (0.0, 0.0) y utiliza operaciones
matemáticas que están disponibles solo para tipos de coma flotante.

Los parámetros de tipo genérico en una definición de estructura no son
siempre los mismos que los que usa en las firmas de métodos de esa
estructura. Por ejemplo, el Listado 10-11 define el método `mixup` en la
estructura `Point <T, U>` del Listado 10-8. El método toma otro `Point` como
parámetro, que puede tener diferentes tipos que el `self` `Point` al que
llamamos `mixup`. El método crea una nueva instancia `Point` con el valor `x`
del `self` `Point` (de tipo `T`) y el valor `y` del `Point` pasado (de tipo `W`)

<span class="filename">Filename: src/main.rs</span>

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}

fn main() {
    let p1 = Point { x: 5, y: 10.4 };
    let p2 = Point { x: "Hello", y: 'c'};

    let p3 = p1.mixup(p2);

    println!("p3.x = {}, p3.y = {}", p3.x, p3.y);
}
```

<span class="caption">Listado 10-11: un método que usa diferentes tipos
genéricos que la definición de su estructura</span>

En `main`, hemos definido un `Point` que tiene un `i32` para `x` (con valor
`5`) y un `f64` para `y` (con valor `10.4`). La variable `p2` es una
estructura `Point` que tiene un *string slice* para `x` (con valor `"Hello"`)
y `char` para `y` (con valor `c`). Llamar `mixup` en `p1` con el argumento
`p2` nos da `p3`, que tendrá un `i32` para `x`, porque `x` viene de `p1`. La
variable `p3` tendrá un `char` para `y`, porque `y` viene de `p2`. La llamada
a la macro `println!` Imprimirá `p3.x = 5, p3.y = c`.

El propósito de este ejemplo es demostrar una situación en la que algunos
parámetros genéricos se declaran con `impl` y algunos se declaran con la
definición del método. Aquí, los parámetros genéricos `T` y `U` se declaran
después de `impl`, porque van con la definición de estructura. Los parámetros
genéricos `V` y `W` se declaran después de `fn mixup`, porque solo son
relevantes para el método.

### Rendimiento del código usando genéricos

Es posible que se pregunte si hay un costo de tiempo de ejecución cuando usa
parámetros de tipo genérico. La buena noticia es que Rust implementa los
genéricos de tal manera que su código no se ralentiza con los tipos genéricos
más que con los tipos concretos.

Rust logra esto realizando la monomorfización del código que usa genéricos en
tiempo de compilación. *Monomorphization* es el proceso de convertir código
genérico en código específico completando los tipos concretos que se utilizan
cuando se compilan.

En este proceso, el compilador hace lo contrario de los pasos que usamos para
crear la función genérica en el Listado 10-5: el compilador observa todos los
lugares donde se llama el código genérico y genera código para los tipos
concretos con los que se llama el código genérico.

Veamos cómo funciona esto con un ejemplo que usa la enumeración `Option <T>`
de la biblioteca estándar:

```rust
let integer = Some(5);
let float = Some(5.0);
```

Cuando Rust compila este código, realiza la monomorfización. Durante ese
proceso, el compilador lee los valores que se han utilizado en las instancias
`Option <T>` e identifica dos tipos de `Option <T>`: uno es `i32` y el otro
es `f64`. Como tal, expande la definición genérica de `Opción <T>` en
`Opción_i32` y `Opción_f64`, reemplazando así la definición genérica por las
específicas.

La versión monomorfizada del código tiene el siguiente aspecto. La
`Opción <T>` genérica se reemplaza por las definiciones específicas creadas
por el compilador:

<span class="filename">Filename: src/main.rs</span>

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

Debido a que Rust compila código genérico en código que especifica el tipo en
cada instancia, no pagamos costo de tiempo de ejecución por el uso de
genéricos. Cuando el código se ejecuta, funciona igual que si hubiéramos
duplicado cada definición a mano. El proceso de monomorfización hace que los
genéricos de Rust sean extremadamente eficientes en tiempo de ejecución.
