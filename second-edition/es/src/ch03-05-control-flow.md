## Flujo de control

Decidir si ejecutar o no algún código según si una condición es verdadera
y decidir ejecutar algún código repetidamente mientras una condición es
verdadera es básico en la construcción de bloques de codigo en la mayoría
de los lenguajes de programación. Las construcciones más comunes que le permiten
controlar el flujo de ejecución del código Rust son las expresiones `if` y los bucles.

### Expresiones `if`

Una expresión `if` le permite bifurcar su código dependiendo de las condiciones. Usted
proporcionará una condición, “Si se cumple esta condición, ejecute este bloque
de código. Si la condición no se cumple, no ejecute este bloque de código”.

Cree un nuevo proyecto llamado *branches* en su directorio *projects* para explorar
la expresión `if`. En el archivo *src/main.rs*, ingrese lo siguiente:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}
```

<!-- NEXT PARAGRAPH WRAPPED WEIRD INTENTIONALLY SEE #199 -->

Todas las expresiones `if` comienzan con la palabra clave `if`, que es seguida por una
condición. En este caso, la condición verifica si la variable `number` tiene un valor
menor que 5. El bloque de código que queremos ejecutar si la condición es verdadera se
coloca inmediatamente después de la condición dentro de las llaves.
Los bloques de código asociados con las condiciones en las expresiones `if` son
a veces llamados *brazos*, al igual que los brazos en expresiones `match` que
tratamos en la sección “Comparación de la conjetura con el número secreto” del Capitulo 2.

Opcionalmente, también podemos incluir una expresión `else`, que elegimos hacer aquí, para
darle al programa un bloque de código alternativo que debería ejecutar si la condición
evalúa a falso. Si no proporciona una expresión `else` y la condición es falsa, el programa
saltará el bloque `if` y continuará al siguiente código.

Intente ejecutar este código; debería ver el siguiente resultado:

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/branches`
condition was true
```

Probemos cambiando el valor de `number` a un valor que haga que la condición sea
`falso` para ver qué sucede:

```rust,ignore
let number = 7;
```

Ejecute el programa nuevamente y observe el resultado:

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/branches`
condition was false
```

También vale la pena señalar que la condición en este código *debe* ser un `bool`. Si
la condición no es un `bool`, obtendremos un error. Por ejemplo, intente ejecutar el
siguiente código:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let number = 3;

    if number {
        println!("number was three");
    }
}
```

La condición `if` se evalúa a un valor de `3` esta vez, y Rust arroja un
error:

```text
error[E0308]: mismatched types
 --> src/main.rs:4:8
  |
4 |     if number {
  |        ^^^^^^ expected bool, found integral variable
  |
  = note: expected type `bool`
             found type `{integer}`
```

El error indica que Rust esperaba un `bool` pero obtuvo un número entero. diferente a
lenguajes como Ruby y JavaScript, Rust no intentará convertir tipos no booleanos en booleanos.
Debe ser explícito y siempre proporcionar `if` con un Booleano como su condición. 
Si queremos que se ejecute el bloque de código `if` solo cuando un número no es igual a `0`,
por ejemplo, podemos cambiar el `if` expresión a lo siguiente:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let number = 3;

    if number != 0 {
        println!("number was something other than zero");
    }
}
```

Ejecutando este código se imprimirá `number was something other than zero`.

#### Manejo de múltiples condiciones con `else if`

Puede tener múltiples condiciones combinando `if` y `else` en una expresión `else if`.
Por ejemplo:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

Este programa tiene cuatro caminos posibles que puede tomar. Después de ejecutarlo, deberías
ver el siguiente resultado:

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running `target/debug/branches`
number is divisible by 3
```

Cuando se ejecuta este programa, comprueba cada una de las expresiones `if` y se ejecuta
el primer cuerpo para el cual la condición es verdadera. Tenga en cuenta que, aunque 6 es
divisible por 2, no vemos la salida `number is divisible by 2`, ni tampoco
vemos el texto `number is not divisible by 4, 3, or 2` del bloque `else`.
Eso es porque Rust solo ejecuta el bloque para la primera condición verdadera, y
una vez que encuentra una, ni siquiera comprueba el resto.

Si usa demasiadas expresiones `else if` puede saturar su código, entonces si tiene más
de una, es posible que desee refactorizar su código. El Capítulo 6 describe un poderoso
construct de ramificación de Rust llamado `match` para estos casos.

#### Usando `if` en una declaración `let`

Como `if` es una expresión, podemos usarla en el lado derecho de una declaración
`let`, como en el Listado 3-2.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let condition = true;
    let number = if condition {
        5
    } else {
        6
    };

    println!("The value of number is: {}", number);
}
```

<span class="caption">Listing 3-2: Asignando el resultado de una expresión `if`
a una variable</span>

La variable `number` se vinculará a un valor basado en el resultado del `if`
expresión. Ejecute este código para ver qué sucede:

```text
$ cargo run
   Compiling branches v0.1.0 (file:///projects/branches)
    Finished dev [unoptimized + debuginfo] target(s) in 0.30 secs
     Running `target/debug/branches`
The value of number is: 5
```

Recuerde que los bloques de código evalúan la última expresión en ellos, y
los números en sí mismos también son expresiones. En este caso, el valor de la
toda la expresión `if` depende de qué bloque de código se ejecute. Esto significa
que los valores que tienen el potencial de ser resultados de cada brazo del `if` deben ser
el mismo tipo; en el Listado 3-2, los resultados tanto del brazo `if` como del brazo `else`
eran enteros `i32`. Si los tipos no coinciden, como en el siguiente
ejemplo, obtendremos un error:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let condition = true;

    let number = if condition {
        5
    } else {
        "six"
    };

    println!("The value of number is: {}", number);
}
```

Cuando intentemos compilar este código, obtendremos un error. Los brazos `if` y `else`
tienen tipos de valores que son incompatibles, y Rust indica exactamente dónde se
encuentra el problema en el programa:

```text
error[E0308]: if and else have incompatible types
 --> src/main.rs:4:18
  |
4 |       let number = if condition {
  |  __________________^
5 | |         5
6 | |     } else {
7 | |         "six"
8 | |     };
  | |_____^ expected integral variable, found &str
  |
  = note: expected type `{integer}`
             found type `&str`
```

La expresión en el bloque `if` se evalúa en un entero, y la expresión en
el bloque `else` se evalúa como un *string*. Esto no funcionará porque las variables
deben tener un solo tipo, Rust necesita saber en tiempo de compilación qué tipo es
la variable `number`, definitivamente, para que pueda verificar en tiempo de
compilación que su tipo es válido en cualquier lugar donde usemos `number`.
Rust no podría hacer eso si el tipo de `number` solo se determina en tiempo de ejecución;
el compilador sería más complejo y haría menos garantías sobre el código si tuviera
que realizar un seguimiento de múltiples tipos hipotéticos para cualquier variable.

### Repetición con bucles

A menudo es útil ejecutar un bloque de código más de una vez. Para esta tarea,
Rust proporciona varios *bucles*. Un bucle recorre el código dentro del cuerpo
del bucle hasta el final y luego comienza de inmediato al principio. Para
experimentar con bucles, hagamos un nuevo proyecto llamado *loops*.

Rust tiene tres tipos de bucles: `loop`,`while` y `for`. Probemos cada uno.

#### Repetición de código con `loop`

La palabra clave `loop` le dice a Rust que ejecute un bloque de código una y otra vez
por siempre o hasta que explícitamente le digas que se detenga.

Como ejemplo, cambie el archivo *loops* en su directorio *src/main.rs* para que
se vea así:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    loop {
        println!("again!");
    }
}
```

Cuando ejecutemos este programa, veremos `again!` Impreso una y otra vez continuamente
hasta que detengamos el programa manualmente. La mayoría de los terminales admiten un atajo de teclado,
<span class = "keystroke">ctrl-c</span>, para detener un programa que está atorado en un
bucle continuo. Darle una oportunidad:

```text
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.29 secs
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

El símbolo `^C` representa donde presionaste <span class =" keystroke ">ctrl-c</span>
Puede o no ver la palabra `again!` impresa después de `^C`,
dependiendo de dónde estaba el código en el bucle cuando recibió la señal de detención.

Afortunadamente, Rust ofrece otra forma más confiable de salir de un bucle.
Puede colocar la palabra clave `break` dentro del ciclo para indicarle al programa cuándo
dejar de ejecutar el ciclo. Recuerde que hicimos esto en el juego de adivinanzas en
la sección “Abandonar después de una conjetura correcta” del Capítulo 2 para salir
del programa cuando el usuario ganó el juego adivinando el número correcto.

#### Bucles condicionales con `while`

A menudo es útil para un programa evaluar una condición dentro de un ciclo. Mientras
la condición es verdadera, el ciclo se ejecuta. Cuando la condición deja de ser cierta,
el programa llama a `break`, deteniendo el ciclo. Este tipo de bucle podría implementarse
usando una combinación de `loop`,`if`, `else`, y `break`; podrías intentar eso
ahora en un programa, si lo desea.

Sin embargo, este patrón es tan común que Rust tiene una construcción en el lenguaje
incorporado para ello, llamado un ciclo `while`. El listado 3-3 usa `while`: el
programa se repite tres veces, contando hacia atras cada vez, y luego, después del ciclo,
imprime otro mensaje y sale.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{}!", number);

        number = number - 1;
    }

    println!("LIFTOFF!!!");
}
```

<span class="caption">Listing 3-3: Usando un ciclo `while` para ejecutar código mientras
la condición es verdadera</span>

Esta construcción elimina una gran cantidad de anidamiento que sería necesario si utilizó
`loop`,` if`, `else`, y` break`, y está más claro. Mientras se cumple una condición,
el código se ejecuta; de lo contrario, sale del bucle.

#### Looping a través de una colección con `for`

Podría usar la construcción `while` para recorrer los elementos de una colección,
como una matriz. Por ejemplo, veamos el Listado 3-4.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    let mut index = 0;

    while index < 5 {
        println!("the value is: {}", a[index]);

        index = index + 1;
    }
}
```

<span class="caption">Listado 3-4: Looping a través de cada elemento de una colección
usando un bucle `while`</span>

Aquí, el código cuenta hacia arriba a través de los elementos en la matriz. Comienza en el índice
`0`, y luego realiza un bucle hasta que alcanza el índice final en la matriz (es decir,
cuando `index < 5` ya no es verdadero). Al ejecutar este código, se imprimirá cada elemento
en la matriz:

```text
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
     Running `target/debug/loops`
the value is: 10
the value is: 20
the value is: 30
the value is: 40
the value is: 50
```

Los cinco valores de la matriz aparecen en la terminal, como se esperaba. Aunque `index`
alcanzará un valor de `5` en algún punto, el ciclo deja de ejecutarse antes de intentar
obtener un sexto valor en la matriz.

Pero este enfoque es propenso a errores; podríamos causar que el programa entrara en pánico si
la longitud del índice es incorrecta. También es lento, porque el compilador agrega en tiempo
de ejecución código para realizar la comprobación condicional en cada elemento en cada iteración
a través del bucle.

Como una alternativa más concisa, puede usar un bucle `for` y ejecutar algún código
para cada artículo en una colección. Un bucle `for` se parece al código del Listado 3-5.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a.iter() {
        println!("the value is: {}", element);
    }
}
```

<span class="caption">Listing 3-5: Looping a través de cada elemento de una colección
usando un bucle `for`</span>

Cuando ejecutamos este código, veremos el mismo resultado que en el listado 3-4. Lo más
importante, es que ahora hemos aumentado la seguridad del código y eliminado la
posibilidad de errores que podrían resultar de ir más allá del final de la matriz o no
ir lo suficientemente lejos y perder algunos elementos.

Por ejemplo, en el código del Listado 3-4, si eliminas un elemento de la matriz `a`
pero se olvida de actualizar la condición a `while index < 4`, el código entraría en pánico.
Usando el ciclo `for`, no necesitarías recordar cambiar cualquier otro código si modifica la
cantidad de valores en la matriz.

La seguridad y la concisión de los bucles `for` los convierten en el bucle más utilizado
en Rust. Incluso en situaciones en las que desea ejecutar algún código,
cierto número de veces, como en el ejemplo de cuenta regresiva que usó un bucle 'while'
en el Listado 3-3, la mayoría de los Rustaceans usaría un bucle `for`. La forma de hacerlo
sería usar un `Range`, que es un tipo proporcionado por la biblioteca estándar
que genera todos los números en secuencia a partir de un número y finalizando
antes de otro número.

Así es como se vería la cuenta atrás utilizando un ciclo `for` y otro método
todavía no hemos hablado, `rev`, para invertir el rango:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{}!", number);
    }
    println!("LIFTOFF!!!");
}
```

Este código es un poco mejor, ¿no?

## Resumen

¡Lo hiciste! Ese fue un capítulo considerable: aprendiste sobre variables, escalar
y tipos de datos compuestos, funciones, comentarios, expresiones `if` y bucles.
Si quieres practicar con los conceptos discutidos en este capítulo, intenta construir
programas para hacer lo siguiente:

* Convierta las temperaturas entre Fahrenheit y Celsius.
* Genera el enésimo número de Fibonacci.
* Imprime la letra del villancico "Los Doce Días de Navidad"
   aprovechando la repetición en la canción.

Cuando esté listo para seguir adelante, hablaremos de un concepto en Rust que *no*
comúnmente existen en otros lenguajes de programación: *ownership* (propiedad).
