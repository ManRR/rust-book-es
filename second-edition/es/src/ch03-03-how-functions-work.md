## Funciones

Las funciones son omnipresentes en el código Rust. Ya has visto una de las
funciones más importantes en el lenguaje: la función `main`, que es el punto
de entrada de muchos programas. También ha visto la palabra clave `fn`, que le
permite declarar nuevas funciones.

El código de Rust usa *snake case* como el estilo convencional (convención de nombres )
para nombres de funciones y variables. En *snake case*, todas las letras son
minúsculas y resaltan palabras separadas. Aquí hay un programa que contiene una
definición de función de ejemplo:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");

    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

Las definiciones de funciones en Rust comienzan con `fn` y tienen un par de
paréntesis después del nombre de la función. Las llaves le dicen al compilador
dónde comienza y termina el cuerpo de la función.

Podemos llamar a cualquier función que hayamos definido al ingresar su nombre
seguido de un conjunto de paréntesis. Debido a que `another_function` se define
en el programa, puede ser llamado desde el interior de la función `main`. Tenga
en cuenta que definimos `another_function` *después de* la función `main` en
el código fuente; podríamos haberlo definido antes también. A Rust no le importa
dónde defines tus funciones, solo que estén definidas en alguna parte

Comencemos un nuevo proyecto binario llamado *functions* para explorar funciones
más allá. Coloque el ejemplo `another_function` en *src/main.rs* y ejecútelo. Usted
debería ver el siguiente resultado:

```text
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.28 secs
     Running `target/debug/functions`
Hello, world!
Another function.
```

Las líneas se ejecutan en el orden en que aparecen en la función `main`.
En primer lugar, se imprime el mensaje "¡Hola, mundo!", Y luego es llamada
`another_function` y su mensaje se imprime.

### Parámetros de función

Las funciones también se pueden definir para tener *parámetros*, que son
variables especiales que son parte de la firma de una función. Cuando una
función tiene parámetros, puede proporcionarle valores concretos para esos
parámetros. Técnicamente, los valores concretos se llaman *argumentos*,
pero en una conversación informal, la gente tiende usar las palabras
*parámetro* y *argumento* intercambiablemente ya sea para las variables
en la definición de una función o los valores concretos pasados cuando se llama
a una función.

La siguiente versión reescrita de `another_function` muestra cómo se ven los parámetros
en Rust:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    another_function(5);
}

fn another_function(x: i32) {
    println!("The value of x is: {}", x);
}
```

Intenta ejecutar este programa; deberías obtener el siguiente resultado:

```text
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 1.21 secs
     Running `target/debug/functions`
The value of x is: 5
```

La declaración de `another_function` tiene un parámetro llamado `x`. El tipo de
`x` se especifica como `i32`. Cuando `5` se pasa a `another_function`, la
macro `println!` pone `5` donde estaban el par de llaves en formato
*string*.

En las firmas de funciones, *debe* declarar el tipo de cada parámetro. Esto es
una decisión deliberada en el diseño de Rust: requerir anotaciones de tipo en
definiciones de función significa que el compilador casi nunca necesita que las
use en otro lugar en el código para entender lo que quieres decir.

Cuando desee que una función tenga múltiples parámetros, separe las declaraciones
de parámetros con comas, como esta:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    another_function(5, 6);
}

fn another_function(x: i32, y: i32) {
    println!("The value of x is: {}", x);
    println!("The value of y is: {}", y);
}
```

Este ejemplo crea una función con dos parámetros, ambos son del tipo `i32`.
La función luego imprime los valores de sus dos parámetros. Tenga en cuenta que
los parámetros de función no necesitan ser todos del mismo tipo, simplemente lo
son para este ejemplo.

Intentemos ejecutar este código. Reemplace el programa actualmente en sus proyecto
*src/main.rs*, *functions* con el ejemplo anterior y ejecútelo usando `cargo
run`:

```text
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/functions`
The value of x is: 5
The value of y is: 6
```

Los dos textos se imprimen con estos valores pasados a la función con `5` cuando
se pasa el valor de `x` y `6` como el valor de `y`.

### Los cuerpos de función contienen declaraciones y expresiones

Los cuerpos de funciones están formados por una serie de declaraciones que opcionalmente
terminan en un expresión. Hasta ahora, solo cubrimos funciones sin una expresión final,
pero has visto una expresión como parte de una declaración. Porque Rust es un
lenguaje basado en expresiones, esta es una distinción importante de entender.
Otros lenguajes no tienen las mismas distinciones, así que veamos qué son
declaraciones y expresiones y cómo sus diferencias afectan los cuerpos de las
funciones.

Ya hemos usado declaraciones y expresiones. *Las declaraciones* son
instrucciones que realizan alguna acción y no devuelven un valor. *Expresiones*
para evaluar un valor resultante. Veamos algunos ejemplos.

Crear una variable y asignarle un valor con la palabra clave `let` es
declaración. En el Listado 3-1, `let y = 6;` es una declaración.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let y = 6;
}
```

<span class="caption">Listing 3-1: La declaración de función `main` contiene una instrucción</span>

Las definiciones de funciones también son declaraciones; todo el ejemplo anterior es un
declaración en sí misma.

Las declaraciones no devuelven valores. Por lo tanto, no puede asignar una instrucción `let`
a otra variable, como intenta hacer el siguiente código; obtendrás un error:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let x = (let y = 6);
}
```

Cuando ejecuta este programa, el error que obtendrá es similar a este:

```text
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
error: expected expression, found statement (`let`)
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^
  |
  = note: variable declaration using `let` is a statement
```

La instrucción `let y = 6` no devuelve un valor, por lo que no hay nada para
enlazar a `x`. Esto es diferente de lo que sucede en otros lenguajes, como
C y Ruby, donde la asignación devuelve el valor de la tarea. En esos
lenguajes, puede escribir `x = y = 6` y tener tanto `x` como `y` tienen el valor
`6`; ese no es el caso en Rust.

Las expresiones evalúan algo y componen la mayor parte del resto del código que
escribirás en Rust. Considere una operación matemática simple, como `5 + 6`, que
es una expresión que evalúa el valor `11`. Las expresiones pueden ser parte de
declaraciones: en el Listado 3-1, el `6` en la declaración ` let y = 6; `es un
expresión que evalúa el valor `6`. Llamar a una función es una expresión. 
Llamar a una macro es una expresión. El bloque que utilizamos para crear nuevos ámbitos,
`{}`, es una expresión, por ejemplo:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = 5;

    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {}", y);
}
```

Esta expresión:

```rust,ignore
{
    let x = 3;
    x + 1
}
```

es un bloque que, en este caso, se evalúa como `4`. Ese valor se liga a `y`
como parte de la declaración `let`. Tenga en cuenta la línea `x + 1`
sin un punto y coma en el final, que es diferente a la mayoría de las líneas
que has visto hasta ahora. Las expresiones no incluye el punto y coma final.
Si agrega un punto y coma al final de un expresión, lo convierte en una declaración,
que luego no devolverá un valor.
Tenga esto en cuenta mientras explora los valores y las expresiones de retorno de
funciones a continuación.

### Funciones con valores de retorno

Las funciones pueden devolver valores al código que las llama. No nombramos los valores
de retorno, pero declaramos su tipo después de una flecha (`->`). En Rust, el retorno
del valor de la función es sinónimo del valor de la expresión final en
el bloque del cuerpo de una función. Puede regresar temprano de una función usando la
palabra clave `return` y especificando un valor, pero la mayoría de las
funciones vuelven la última expresión implícitamente. Aquí hay un ejemplo de una función
que devuelve un valor:

<span class="filename">Filename: src/main.rs</span>

```rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {}", x);
}
```

No hay llamadas a función, macros, o incluso declaraciones `let` en la función `five`
solo el número `5` en si mismo. Esa es una función perfectamente válida en Rust.
Tenga en cuenta que el tipo de retorno de la función también se especifica como `-> i32`.
Trate ejecutando este código; la salida debería verse así:

```text
$ cargo run
   Compiling functions v0.1.0 (file:///projects/functions)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30 secs
     Running `target/debug/functions`
The value of x is: 5
```

El `5` en `five` es el valor de retorno de la función, por lo que el tipo de retorno
es `i32`. Examinemos esto con más detalle. Hay dos partes importantes:
primero, la línea `let x = five ();` muestra que estamos usando el valor de retorno de un
función para inicializar una variable. Debido a que la función `five` devuelve un `5`,
esa línea es la misma que la siguiente:

```rust
let x = 5;
```

En segundo lugar, la función `five` no tiene parámetros y define el tipo del
valor de retorno, pero el cuerpo de la función es un `5` solitario sin punto y coma
porque es una expresión cuyo valor queremos devolver.

Veamos otro ejemplo:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {}", x);
}

fn plus_one(x: i32) -> i32 {
    x + 1
}
```

Al ejecutar este código se imprimirá `The value of x is: 6`. Pero si colocamos un
punto y coma al final de la línea que contiene `x + 1`, cambiándolo de un
expresión a una declaración, obtendremos un error.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let x = plus_one(5);

    println!("The value of x is: {}", x);
}

fn plus_one(x: i32) -> i32 {
    x + 1;
}
```

La ejecución de este código genera un error de la siguiente manera:

```text
error[E0308]: mismatched types
 --> src/main.rs:7:28
  |
7 |   fn plus_one(x: i32) -> i32 {
  |  ____________________________^
8 | |     x + 1;
  | |          - help: consider removing this semicolon
9 | | }
  | |_^ expected i32, found ()
  |
  = note: expected type `i32`
             found type `()`
```

El mensaje de error principal, “mismatched types,” (tipos no coincidentes),
revela el problema central con este código. La definición de la función `plus_one`
dice que devolverá un `i32`, pero las declaraciones no evalúan a un valor, que
se expresa por `()`, la tupla vacía. Por lo tanto, no se devuelve nada, lo que
contradice la definición de la función y resultando en un error. En esta salida,
Rust proporciona un mensaje que posiblemente ayude a rectificar este problema:
sugiere eliminar el punto y coma, que arreglaría el error.
