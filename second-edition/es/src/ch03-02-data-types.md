## Tipos de datos

Cada valor en Rust es de un cierto *tipo de datos*, que le informa a Rust qué
tipo de datos se están especificando para que sepa cómo trabajar con esos datos.
Veremos dos subconjuntos de tipos de datos: escalares y compuestos.

Tenga en cuenta que Rust es un lenguaje *estáticamente tipado*, lo que
significa que debe conocer los tipos de todas las variables en tiempo de
compilación. El compilador generalmente puede inferir qué tipo queremos usar
en función del valor y cómo lo usamos. En los casos en que son posibles
muchos tipos, como cuando convertimos un `String` a un tipo numérico usando
`parse` en la sección “Comparando la conjetura con el número secreto” en el
Capítulo 2, debemos agregar una *anotación de tipo* (*type annotation*), como esta :

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

Si no agregamos la anotación de tipo aquí, Rust mostrará el siguiente error,
lo que significa que el compilador necesita más información de nosotros para
saber qué tipo queremos usar:

```text
error[E0282]: type annotations needed
 --> src/main.rs:2:9
  |
2 |     let guess = "42".parse().expect("Not a number!");
  |         ^^^^^
  |         |
  |         cannot infer type for `_`
  |         consider giving `guess` a type
```

Verá anotaciones de tipos diferentes para otros tipos de datos.

### Tipos escalares

Un tipo *escalar* representa un solo valor. Rust tiene cuatro tipos escalares
principales: enteros, números de coma flotante, booleanos y caracteres. Puede
reconocer estos de otros lenguajes de programación. Veamos cómo funcionan en
Rust.

#### Tipos *enteros* (*Integer*)

Un *entero* es un número sin un componente fraccionario. Usamos un tipo entero
en el Capítulo 2, el tipo `u32`. Esta declaración de tipo indica que el valor
con el que está asociado debe ser un entero sin signo (los tipos enteros con
signo comienzan con `i`, en lugar de `u`) que ocupan 32 bits de espacio. La
Tabla 3-1 muestra los tipos enteros integrados en Rust. Cada variante en las
columnas *Firmado* y *No firmado* (por ejemplo, `i16`) se puede usar para declarar
el tipo de un valor entero.

<span class="caption">Tabla 3-1: Tipos enteros en Rust</span>

| Length | Signed  | Unsigned |
|--------|---------|----------|
| 8-bit  | `i8`    | `u8`     |
| 16-bit | `i16`   | `u16`    |
| 32-bit | `i32`   | `u32`    |
| 64-bit | `i64`   | `u64`    |
| arch   | `isize` | `usize`  |

Cada variante puede ser firmado o sin firmar y tiene un tamaño explícito.
*Firmado* y *sin firmar* (*Signed* and *unsigned*) se refieren a si es posible
que el número sea negativo o positivo; en otras palabras, si el número debe
tener un signo con él (firmado) o si solo será positivo y, por lo tanto,
puede ser representado sin un signo (sin firmar). Es como escribir números en
papel: cuando el signo importa, se muestra un número con un signo más o un
signo menos; sin embargo, cuando es seguro suponer que el número es positivo,
se muestra sin señal. Los números firmados se almacenan usando la
representación de dos complementos (si no está seguro de qué es esto, puede
buscarlo en línea, una explicación queda fuera del alcance de este libro).

Cada variante firmada puede almacenar números de -(2<sup>n - 1</sup>) a
2<sup>n - 1</sup> - 1 inclusive, donde *n* es el número de bits que utiliza la
variante. Entonces, un `i8` puede almacenar números de -(2<sup>7</sup>) a
2<sup>7</sup> - 1, lo que equivale a -128 a 127. Las variantes sin firmar
pueden almacenar números del 0 al 2<sup>n</sup> - 1, por lo que un `u8` puede
almacenar números de 0 a 2<sup>8</sup> - 1, que es igual a 0 a 255.

Además, los tipos `isize` y `usize` dependen del tipo de computadora en la que
se ejecute el programa: 64 bits si está en una arquitectura de 64 bits y 32
bits si está en una arquitectura de 32 bits.

Puede escribir literales enteros en cualquiera de las formas que se muestran
en la Tabla 3-2. Tenga en cuenta que todos los literales numéricos excepto el
byte literal permiten un sufijo de tipo, como `57u8`, y `_` como un separador
visual, como `1_000`.

<span class="caption">Tabla 3-2: Literales enteros en Rust</span>

| Number literals  | Example       |
|------------------|---------------|
| Decimal          | `98_222`      |
| Hex              | `0xff`        |
| Octal            | `0o77`        |
| Binary           | `0b1111_0000` |
| Byte (`u8` only) | `b'A'`        |

Entonces, ¿cómo sabes qué tipo de número entero usar?, si no está seguro, los
valores predeterminados de Rust generalmente son buenas opciones, y los tipos
enteros predeterminados para `i32`: este tipo es generalmente el más rápido,
incluso en sistemas de 64 bits. La situación principal en la que usaría
`isize` o `usize` es cuando se indexa algún tipo de colección.

#### Tipos de punto flotante

Rust también tiene dos tipos primitivos para *números de punto flotante*, que
son números con puntos decimales. Los tipos de punto flotante de Rust son
`f32` y `f64`, que son de 32 bits y 64 bits de tamaño, respectivamente. El
tipo predeterminado es `f64` porque en las CPU modernas es más o menos la
misma velocidad que `f32`, pero es capaz de obtener más precisión.

Aquí hay un ejemplo que muestra los números de coma flotante en acción:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = 2.0; // f64

    let y: f32 = 3.0; // f32
}
```

Los números de coma flotante se representan según el estándar IEEE-754. El
tipo `f32` es un *float* de precisión simple, y `f64` tiene doble precisión.

#### Operaciones Numéricas

Rust admite las operaciones matemáticas básicas que esperarías para todos los
tipos de números: suma, resta, multiplicación, división y resto. El siguiente
código muestra cómo usarías cada uno en una declaración `let`:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    // addition
    let sum = 5 + 10;

    // subtraction
    let difference = 95.5 - 4.3;

    // multiplication
    let product = 4 * 30;

    // division
    let quotient = 56.7 / 32.2;

    // remainder
    let remainder = 43 % 5;
}
```

Cada expresión en estas declaraciones usa un operador matemático y evalúa a un
solo valor, que luego se vincula a una variable. El Apéndice B contiene una
lista de todos los operadores que Rust proporciona.

#### El tipo *booleano* (*Boolean*)

Como en la mayoría de los demás lenguajes de programación, un tipo *booleano* en
Rust tiene dos valores posibles: `true` y `false`. El tipo *booleano* en Rust se
especifica con `bool`. Por ejemplo:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let t = true;

    let f: bool = false; // with explicit type annotation
}
```

La forma principal de usar valores *booleanos* es mediante condicionales, como
una expresión `if`. Cubriremos cómo funcionan las expresiones `if` en Rust en
la sección “Flujo de control”.

#### El tipo carácter

Hasta ahora solo hemos trabajado con números, pero Rust también admite letras.
El tipo `char` de Rust es el tipo alfabético más primitivo del lenguaje, y el
siguiente código muestra una forma de usarlo. (Tenga en cuenta que el tipo
`char` se especifica con comillas simples, a diferencia de los *strings*, que
usan comillas dobles).

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let c = 'z';
    let z = 'ℤ';
    let heart_eyed_cat = '😻';
}
```

El tipo `char` de Rust representa un valor escalar de Unicode, lo que
significa que puede representar mucho más que solo ASCII. Letras acentuadas;
Caracteres chinos, japoneses y coreanos; emoji; y los espacios *zero-width*
todos los valores `char` válidos en Rust. Los valores escalares Unicode van
desde `U + 0000` a `U + D7FF` y `U + E000` a `U + 10FFFF` inclusive. Sin
embargo, un “carácter” no es realmente un concepto en Unicode, por lo que su
intuición humana para lo que es un “carácter” puede no coincidir con lo que es
un `char` en Rust. Discutiremos este tema en detalle en “Strings” en el
Capítulo 8.

### Tipos de compuestos

*Los tipos compuestos* pueden agrupar múltiples valores en un tipo. Rust tiene
dos tipos de compuestos primitivos: tuplas y matrices.

#### El tipo de *tupla* (*Tuple*)

Una tupla es una forma general de agrupar algunos otros valores con una
variedad de tipos en un tipo compuesto.

Creamos una tupla escribiendo una lista de valores separados por comas dentro
de paréntesis. Cada posición en la tupla tiene un tipo, y los tipos de los
diferentes valores en la tupla no tienen que ser los mismos. Agregamos
anotaciones de tipo opcionales en este ejemplo:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
}
```

La variable `tup` se une a la tupla completa, porque una tupla se considera un
solo elemento compuesto. Para obtener los valores individuales de una tupla,
podemos usar la coincidencia de patrones para desestructurar un valor de tupla
como este:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is: {}", y);
}
```

Este programa crea primero una tupla y la vincula a la variable `tup`. Luego
usa un patrón con `let` para tomar `tup` y convertirlo en tres variables
separadas, `x`, `y`, y `z`. Esto se llama *desestructuración*, porque divide
la tupla individual en tres partes. Finalmente, el programa imprime el valor
de `y`, que es `6.4`.

Además de la desestructuración mediante la coincidencia de patrones, podemos
acceder directamente a un elemento de tupla usando un punto (`.`) seguido del
índice del valor al que queremos acceder. Por ejemplo:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```

Este programa crea una tupla, `x`, y luego crea nuevas variables para cada
elemento usando su índice. Como con la mayoría de los lenguajes de
programación, el primer índice en una tupla es 0.

#### El tipo de *matriz* (*Array*)

Otra forma de tener una colección de valores múltiples es con una *matriz*. A
diferencia de una tupla, cada elemento de una matriz debe tener el mismo tipo.
Las matrices en Rust son diferentes de las matrices en algunos otros lenguajes
porque las matrices en Rust tienen una longitud fija: una vez declaradas, no
pueden crecer o reducirse de tamaño.

En Rust, los valores que entran en una matriz se escriben como una lista
separada por comas entre corchetes:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];
}
```

Las matrices son útiles cuando quieres que tus datos se asignen en la pila en
lugar de en el montículo (discutiremos más sobre la pila y el montículo en el
Capítulo 4) o cuando quieras asegurarte de tener siempre una cantidad fija de
elementos. Sin embargo, una matriz no es tan flexible como el tipo de vector.
Un vector es un tipo de colección similar provisto por la biblioteca estándar
*que* permite crecer o reducir de tamaño. Si no está seguro de utilizar una
matriz o un vector, probablemente debería usar un vector. El Capítulo 8
discute los vectores con más detalle.

Un ejemplo de cuándo puede querer usar una matriz en lugar de un vector es un
programa que necesita saber los nombres de los meses del año. Es muy poco
probable que dicho programa necesite agregar o eliminar meses, por lo que
puede usar una matriz porque sabe que siempre contendrá 12 elementos:

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

##### Accediendo a elementos de una matriz

Una matriz es un solo trozo de memoria asignado en la pila. Puede acceder a
los elementos de una matriz mediante indexación, como esta:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];

    let first = a[0];
    let second = a[1];
}
```

En este ejemplo, la variable llamada `first` obtendrá el valor `1`, porque ese
es el valor en el índice `[0]` en la matriz. La variable llamada `second`
obtendrá el valor `2` del índice `[1]` en la matriz.

##### Acceso a elementos de matriz no válidos

¿Qué sucede si intenta acceder a un elemento de una matriz que está más allá
del final de la matriz?. Supongamos que cambia el ejemplo al siguiente código,
que se compilará pero saldrá con un error cuando se ejecute:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let a = [1, 2, 3, 4, 5];
    let index = 10;

    let element = a[index];

    println!("The value of element is: {}", element);
}
```

Ejecutar este código usando `cargo run` produce el siguiente resultado:

```text
$ cargo run
   Compiling arrays v0.1.0 (file:///projects/arrays)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/arrays`
thread '<main>' panicked at 'index out of bounds: the len is 5 but the index is
 10', src/main.rs:6
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

La compilación no produjo ningún error, pero el programa provocó un error
*runtime* y no finalizó correctamente. Cuando intente acceder a un elemento
mediante indexación, Rust comprobará que el índice que ha especificado es
inferior a la longitud de la matriz. Si el índice es mayor que la longitud,
Rust entrará en *pánico* (*panic*), que es el término que utiliza Rust cuando
un programa sale con un error.

Este es el primer ejemplo de los principios de seguridad de Rust en acción. En
muchos lenguajes de bajo nivel, este tipo de comprobación no se realiza, y
cuando proporciona un índice incorrecto, se puede acceder a la memoria no
válida. Rust lo protege contra este tipo de error al salir de inmediato en
lugar de permitir el acceso a la memoria y continuar. El Capítulo 9 analiza
más sobre el manejo de errores de Rust.
