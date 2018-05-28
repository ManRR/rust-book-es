## Variables y mutabilidad

Como se mencionó en el Capítulo 2, las variables predeterminadas son
inmutables. Este es uno de los muchos empujones que Rust le brinda para
escribir su código de una manera que aproveche la seguridad y la
fácil concurrencia que ofrece Rust. Sin embargo, todavía tienes la opción de
hacer que tus variables sean mutables. Exploremos cómo y por qué Rust lo
anima a elegir la inmutabilidad y por qué a veces es posible que desee
optar por no participar.

Cuando una variable es inmutable, una vez que un valor está vinculado a un
nombre, no puede cambiar ese valor. Para ilustrar esto, generemos un nuevo
proyecto llamado *variables* en su directorio *projects* usando
`cargo new --bin variables`.

Luego, en su nuevo directorio *variables*, abra *src/main.rs* y reemplace su
código con el siguiente código que no se compilará aún:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

Guarde y ejecute el programa usando `cargo run`. Debería recibir un mensaje
de error, como se muestra en este resultado:

```text
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:4:5
  |
2 |     let x = 5;
  |         - first assignment to `x`
3 |     println!("The value of x is: {}", x);
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable
```

Este ejemplo muestra cómo el compilador lo ayuda a encontrar errores en sus
programas. Aunque los errores del compilador pueden ser frustrantes, solo se
refieren a que su programa todavía no está haciendo lo que quiere hacer de
manera segura; ellos *no* significan, ¡que no eres un buen programador! Los
*Rustacean* experimentados aún tienen errores de compilación.

El mensaje de error indica que la causa del error es que `no puede
asignar dos veces a la variable inmutable x`, porque trataste de asignar un
segundo valor a la variable inmutable `x`.

Es importante que obtengamos errores de tiempo de compilación cuando
intentamos cambiar un valor que previamente hemos designado como inmutable
porque esta misma situación puede conducir a errores. Si una parte de nuestro
código opera bajo el supuesto de que el valor nunca cambiará y otra parte de
nuestro código cambia ese valor, es posible que la primera parte del código
no haga lo que estaba diseñado para hacer. La causa de este tipo de error
puede ser difícil de rastrear después de hecho, especialmente cuando la
segunda parte del código cambia el valor solo *algunas veces*.

En Rust, el compilador garantiza que cuando declaras que un valor no cambiará,
realmente no cambiará. Eso significa que cuando estás leyendo y escribiendo
código, no es necesario que realice un seguimiento de cómo y dónde puede
cambiar un valor. Tu codigo es así más fácil de razonar.

Pero la mutabilidad puede ser muy útil. Las variables son inmutables solo por
defecto; como lo hiciste en el Capítulo 2, puedes hacer que sean mutables
añadiendo `mut` delante del nombre de la variable. Además de permitir que
este valor cambie, `mut` transmite intención para los futuros lectores del
código al indicar que en otras partes del código cambiará esta variable de valor.

Por ejemplo, cambiemos *src/main.rs* a lo siguiente:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {}", x);
    x = 6;
    println!("The value of x is: {}", x);
}
```

Cuando ejecutamos el programa ahora, obtenemos esto:

```text
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30 secs
     Running `target/debug/variables`
The value of x is: 5
The value of x is: 6
```

Se nos permite cambiar el valor al que `x` se une de `5` a `6` cuando se usa
`mut`. En algunos casos, querrá hacer una variable mutable porque hace que el
código sea más cómodo de escribir que si solo tuviera variables inmutables.

Existen múltiples compensaciones que se deben considerar además de la
prevención de errores. Por ejemplo, en los casos en que está utilizando
estructuras de datos grandes, mutar una instancia en su lugar puede ser más
rápido que copiar y devolver instancias recientemente asignadas. Con
estructuras de datos más pequeñas, crear nuevas instancias y escribir en un
estilo de programación más funcional puede ser más fácil de pensar, por lo
que un rendimiento inferior podría ser una penalización que valga la pena para
obtener esa claridad.

### Diferencias entre variables y constantes

No poder cambiar el valor de una variable podría haberle recordado otro
concepto de programación que la mayoría de los demás lenguajes tienen:
*constantes*. Al igual que las variables inmutables, las constantes son
valores que están vinculados a un nombre y no pueden cambiar, pero existen
algunas diferencias entre las constantes y las variables.

Primero, no está permitido usar `mut` con constantes. Las constantes no son
solo inmutables por defecto, siempre son inmutables.

Usted declara constantes usando la palabra clave `const` en lugar de la
palabra clave `let`, y el tipo del valor *debe* ser anotado. Estamos a punto
de cubrir *types* y *type annotations* en la siguiente sección, “Tipos de
datos”, así que no se preocupe por los detalles en este momento. Solo debes
saber que siempre debes anotar el tipo.

Las constantes se pueden declarar en cualquier ámbito, incluido el alcance
global, lo que las hace útiles para valores que muchas partes del código
deben conocer.

La última diferencia es que las constantes se pueden establecer solo en una
expresión constante, no en el resultado de una llamada a función o cualquier
otro valor que solo se pueda calcular en tiempo de ejecución.

Aquí hay un ejemplo de una declaración constante donde el nombre de la
constante es `MAX_POINTS` y su valor está establecido en 100,000. (La
convención de nomenclatura de Rust para constantes es usar mayúsculas con
guiones bajos entre las palabras):

```rust
const MAX_POINTS: u32 = 100_000;
```

Las constantes son válidas durante todo el tiempo que se ejecuta un programa,
dentro del alcance en el que fueron declaradas, lo que las convierte en una
opción útil para los valores en su dominio de aplicación que varias partes
del programa podrían necesitar conocer, como la cantidad máxima de puntos que
cualquier jugador de un juego puede ganar o la velocidad de la luz.

Nombrar los valores codificados usados a lo largo de su programa como
constantes es útil para transmitir el significado de ese valor a los futuros
mantenedores del código. También ayuda tener solo un lugar en el código que
necesitaría cambiar si el valor codificado debe actualizarse en el futuro.

### *Sombreado* (*Shadowing*)

Como viste en el tutorial del juego de adivinanzas en la sección
“Comparando la conjetura con el número Secreto” en el Capítulo 2, puedes declarar una
nueva variable con el mismo nombre que una variable anterior, y la nueva
variable sombrea la variable anterior. Los Rustaceans dicen que la primera
variable es *sombreada* por la segunda, lo que significa que el valor de la
segunda variable es lo que aparece cuando se usa la variable. Podemos
sombrear una variable usando el mismo nombre de variable y repitiendo el uso
de la palabra clave `let` de la siguiente manera:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = 5;

    let x = x + 1;

    let x = x * 2;

    println!("The value of x is: {}", x);
}
```

Este programa primero vincula `x` a un valor de `5`. Luego sombrea `x`
repitiendo `let x =`, tomando el valor original y agregando `1` para que el
valor de `x` sea `6`. La tercera instrucción `let` también sombrea `x`,
multiplicando el valor anterior por `2` para dar `x` un valor final de `12`.
Cuando ejecutamos este programa, dará como resultado lo siguiente:

```text
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/variables`
The value of x is: 12
```

El *sombreado* (*shadowing*) es diferente a marcar una variable como `mut`,
porque obtendremos un error en tiempo de compilación si accidentalmente
intentamos reasignar a esta variable sin usar la palabra clave `let`. Al
usar `let`, podemos realizar algunas transformaciones en un valor, pero la
variable debe ser inmutable después de que se hayan completado esas
transformaciones.

La otra diferencia entre `mut` y *shadowing* es que, debido a que estamos
creando efectivamente una nueva variable cuando usamos la palabra clave `let`
nuevamente, podemos cambiar el tipo del valor pero reutilizar el mismo
nombre. Por ejemplo, supongamos que nuestro programa le pide a un usuario que
muestre cuántos espacios quieren entre algunos textos ingresando caracteres
espaciales, pero realmente queremos almacenar esa entrada como un número:

```rust
let spaces = "   ";
let spaces = spaces.len();
```

Esta construcción está permitida porque la primera variable `spaces` es un
tipo de *string* y la segunda variable `spaces`, que es una nueva variable
que pasa a tener el mismo nombre que la primera, es un tipo de número. El
sombreado nos ahorra tener que encontrar diferentes nombres, como
`spaces_str` y `spaces_num`; en su lugar, podemos reutilizar el nombre de
`spaces` más simple. Sin embargo, si tratamos de usar `mut` para esto, como
se muestra aquí, obtendremos un error en tiempo de compilación:

```rust,ignore
let mut spaces = "   ";
spaces = spaces.len();
```

El error dice que no podemos cambiar el tipo de una variable:

```text
error[E0308]: mismatched types
 --> src/main.rs:3:14
  |
3 |     spaces = spaces.len();
  |              ^^^^^^^^^^^^ expected &str, found usize
  |
  = note: expected type `&str`
             found type `usize`
```

Ahora que hemos explorado cómo funcionan las variables, veamos más tipos de
datos que pueden haber.
