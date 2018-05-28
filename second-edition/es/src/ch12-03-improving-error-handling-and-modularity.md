## Refactorización para mejorar la modularidad y el manejo de errores

Para mejorar nuestro programa, solucionaremos cuatro problemas que tienen que
ver con el estructura del programa y cómo maneja posibles errores.

Primero, nuestra función `main` ahora realiza dos tareas: analiza argumentos y
abre archivos para una función tan pequeña, este no es un problema
importante. Sin embargo, si continuamos haciendo crecer nuestro programa
dentro de `main`, aumentará la cantidad de tareas separadas que maneja la
función `main`. A medida que una función gana responsabilidades, se vuelve
más difícil razonar, más difícil de probar y más difícil de cambiar sin
romper una de sus partes. Lo mejor es separar la funcionalidad, por lo que
cada función es responsable de una tarea.

Este problema también se relaciona con el segundo problema: aunque `query`
y `filename` son variables de configuración para nuestro programa, variables
como `f` y `contents` se utilizan para realizar la lógica del programa.
Cuanto más largo sea el `main`, más variables que tendremos que llevar al
alcance; cuantas más variables tengamos en alcance, más difícil será hacer un
seguimiento del propósito de cada uno. Lo mejor es agrupar las variables de
configuración en una estructura para aclarar su propósito.

El tercer problema es que hemos utilizado `expect` para imprimir un mensaje
de error cuando abrir el archivo falla, pero el mensaje de error simplemente
imprime `file not found`. Abrir un archivo puede fallar de varias maneras
además del archivo que falta: por ejemplo, el archivo puede existir, pero es
posible que no tengamos permiso para abrirlo.
En este momento, si estamos en esa situación, imprimiríamos el error
`file not found` ¡mensaje, que le daría al usuario la información incorrecta!

En cuarto lugar, usamos `expect` repetidamente para manejar diferentes errores, y si el usuario ejecuta nuestro programa sin especificar suficientes
argumentos, obtendrán un `index out of bounds` error de Rust que no explica claramente el problema.Sería mejor si todos los códigos de manejo de errores
estuvieran en un solo lugar para que los futuros mantenedores solo tengan un
lugar para consultar en el código si la lógica de manejo de errores necesitara
un cambio. Tener todo el código de manejo de errores en un lugar también
asegurará que estamos imprimiendo mensajes que serán significativos para
nuestros usuarios finales.

Vamos a abordar estos cuatro problemas mediante la refactorización de nuestro
proyecto.

### Separación de preocupaciones para proyectos binarios

El problema organizativo de asignar la responsabilidad de tareas múltiples a
la función `main` es común a muchos proyectos binarios. Como resultado, la
comunidad de Rust ha desarrollado un proceso para utilizar como una guía para
dividir las preocupaciones por separado de un programa binario cuando `main`
comienza a hacerse grande. El proceso tiene los siguientes pasos:

* Divida su programa en *main.rs* y *lib.rs* y mueva la lógica de su programa
 a *lib.rs *.
* Siempre que su lógica de análisis de línea de comandos sea pequeña, puede
 permanecer en *main.rs*.
* Cuando la lógica de análisis de línea de comando comienza a complicarse,
 extráigala de *main.rs* y muévala a *lib.rs*.

Las responsabilidades que permanecen en la función `main` después de este
proceso deberían limitarse a lo siguiente:

* Llamar a la lógica de análisis de línea de comando con los valores de
 argumento
* Configuración de cualquier otra configuración
* Llamar a una función `run` en *lib.rs*
* Manejando el error si `run` devuelve un error

Este patrón trata de separar las preocupaciones: *main.rs*
maneja ejecutar el programa, y *lib.rs* maneja toda la lógica de la tarea en
cuestión. Como no puede probar la función `main` directamente, esta
estructura le permite probar toda la lógica de su programa moviéndola a
funciones en *lib.rs*. El único código que permanece en *main.rs* será lo
suficientemente pequeño para verificar su corrección al leerlo. Repasemos
nuestro programa siguiendo este proceso.

#### Extrayendo el analizador de argumentos

Extraeremos la funcionalidad para analizar argumentos en una función a la que
`main` llamará para prepararse para mover la lógica de análisis de línea de
comando a *src/lib.rs *. El listado 12-5 muestra el nuevo inicio de `main`
que llama a una nueva función `parse_config`, que definiremos en
*src/main.rs* por el momento.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let args: Vec<String> = env::args().collect();

    let (query, filename) = parse_config(&args);

    // --snip--
}

fn parse_config(args: &[String]) -> (&str, &str) {
    let query = &args[1];
    let filename = &args[2];

    (query, filename)
}
```

<span class="caption">Listado 12-5: Extrayendo una función `parse_config`
de `main`</span>

Todavía estamos recopilando los argumentos de la línea de comando en un
vector, pero en lugar de asignar el valor del argumento en el índice 1 a la
variable `query` y el valor del argumento en el índice 2 a la variable
`filename` dentro de la función `main`, pasa el vector completo a la función
`parse_config`. La función `parse_config` contiene la lógica que determina
qué argumento va en qué variable y pasa los valores a `main`. Seguimos
creando las variables `query` y `filename` en `main`, pero `main` ya no tiene
la responsabilidad de determinar cómo se corresponden los argumentos y las
variables de la línea de comando.

Este reproceso puede parecer excesivo para nuestro pequeño programa, pero
estamos refactorizando en pasos pequeños e incrementales. Después de hacer
este cambio, ejecute el programa nuevamente para verificar que el análisis
del argumento aún funcione. Es bueno verificar su progreso a menudo, para
ayudar a identificar la causa de los problemas cuando ocurren.

#### Agrupación de valores de configuración

Podemos dar otro pequeño paso para mejorar aún más la función `parse_config`.
Por el momento, estamos devolviendo una tupla, pero luego inmediatamente
dividimos esa tupla en partes individuales nuevamente. Este es un signo de
que quizás todavía no tengamos la abstracción correcta.

Otro indicador que muestra que hay margen de mejora es la parte `config`
de `parse_config`, lo que implica que los dos valores que devolvemos están
relacionados y son ambos parte de un valor de configuración. Actualmente, no
transmitimos este significado en la estructura de los datos, salvo agrupando
los dos valores en una tupla; podríamos poner los dos valores en una
estructura y darle a cada uno de los campos de la estructura un nombre
significativo. Hacerlo hará que sea más fácil para los futuros mantenedores
de este código entender cómo los diferentes valores se relacionan entre sí y
cuál es su propósito.

> Nota: Algunas personas llaman a este anti-patrón de usar valores primitivos
> cuando un tipo complejo sería más apropiado *obsesión primitiva*.

El listado 12-6 muestra las mejoras a la función `parse_config`.

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
# use std::env;
# use std::fs::File;
#
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = parse_config(&args);

    println!("Searching for {}", config.query);
    println!("In file {}", config.filename);

    let mut f = File::open(config.filename).expect("file not found");

    // --snip--
}

struct Config {
    query: String,
    filename: String,
}

fn parse_config(args: &[String]) -> Config {
    let query = args[1].clone();
    let filename = args[2].clone();

    Config { query, filename }
}
```

<span class="caption">Listado 12-6: Refactorizando `parse_config` para
devolver una instancia de una estructura `Config`</span>

Hemos agregado una estructura llamada `Config` definida para tener campos
llamados `query` y `filename`. La firma de `parse_config` ahora indica que
devuelve un valor `Config`. En el cuerpo de `parse_config`, donde solíamos
devolver segmentos de *string* que hacen referencia a los valores de `String`
en `args`, ahora definimos `Config` para que contenga los valores `String`.
La variable `args` en` main` es el propietario de los valores del argumento y
solo permite que la función `parse_config` los tome prestados, lo que
significa que violaremos las reglas de préstamo de Rust si `Config` intenta
tomar posesión de los valores en `args`.

Podríamos administrar los datos de `String` de varias maneras diferentes,
pero la ruta más fácil, aunque algo ineficiente, es llamar al método `clone`
en los valores. Esto hará que una copia completa de los datos para la
instancia `Config` sea de su propiedad, lo que requiere más tiempo y memoria
que el almacenamiento de una referencia a los datos de *string*. Sin embargo,
la clonación de datos también hace que nuestro código sea muy sencillo porque
no tenemos que administrar el *lifetime* de las referencias; En esta
circunstancia, renunciar a un pequeño rendimiento para ganar simplicidad es
una valiosa compensación.

> ### Las compensaciones de usar `clone`
>
> Existe una tendencia entre muchos Rustaceans de evitar el uso de `clone`
> para corregir problemas de propiedad debido a su costo de tiempo de
> ejecución. En el Capítulo 13, aprenderá cómo usar métodos más eficientes en
> este tipo de situaciones. Pero, por ahora, está bien copiar algunos
> *strings* para seguir avanzando porque harás estas copias solo una vez y tu
> nombre de archivo y *string* de consulta serán muy pequeños. Es mejor tener
> un programa que funcione que sea un poco ineficiente que tratar de
> hiperoptimizar el código en su primer pase. A medida que tenga más
> experiencia con Rust, será más fácil comenzar con la solución más eficiente,
> pero por ahora, es perfectamente aceptable llamar `clone`.

Hemos actualizado `main` para que coloque la instancia de `Config` devuelta
por `parse_config` en una variable llamada `config`, y actualizamos el código
que previamente usaba las variables separadas `query` y `filename` para que
ahora usa los campos en la estructura `Config` en su lugar.

Ahora nuestro código transmite más claramente que `query` y `filename` están
relacionados y que su propósito es configurar cómo funcionará el programa.
Cualquier código que use estos valores sabe para encontrarlos en la instancia
`config` en los campos nombrados para su propósito.

#### Creando un Constructor para `Config`

Hasta ahora, hemos extraído la lógica responsable de analizar los argumentos
de la línea de comando de `main` y lo colocamos en la función `parse_config`.
Al hacerlo, nos ayudó a ver que los valores `query` y `filename` estaban
relacionados y esa relación debería transmitirse en nuestro código. Luego
agregamos una estructura `Config` para nombrar el propósito relacionado de
`query` y `filename` y para poder devolver los nombres de los valores como
los nombres de los campos struct de la función `parse_config`.

Entonces ahora que el propósito de la función `parse_config` es crear una
instancia `Config`, podemos cambiar `parse_config` de una función simple a
una función llamada `new` que está asociada con la estructura `Config`.
Hacer este cambio hará que el código sea más idiomático. Podemos crear
instancias de tipos en la biblioteca estándar, como `String`, llamando a
`String::new`. Del mismo modo, al cambiar `parse_config` en una `new` función
asociada con `Config`, podremos crear instancias de `Config` llamando a `Config::new`. El listado 12-7 muestra los cambios que debemos hacer.

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
# use std::env;
#
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args);

    // --snip--
}

# struct Config {
#     query: String,
#     filename: String,
# }
#
// --snip--

impl Config {
    fn new(args: &[String]) -> Config {
        let query = args[1].clone();
        let filename = args[2].clone();

        Config { query, filename }
    }
}
```

<span class="caption">Listado 12-7: Cambiar `parse_config` en
`Config::new`</span>

Hemos actualizado `main` donde llamamos `parse_config` para llamar a
`Config::new`. Cambiamos el nombre de `parse_config` a `new` y lo movimos
dentro de un bloque `impl`, que asocia la función `new` con `Config`. Intenta
compilar este código nuevamente para asegurarte de que funciona.

### Reparar el manejo de errores

Ahora trabajaremos en arreglar nuestro manejo de errores. Recuerde que
intentar acceder a los valores en el vector `args` en el índice 1 o en el
índice 2 hará que el programa entre en pánico si el vector contiene menos de
tres elementos. Intenta ejecutar el programa sin ningún argumento; se verá
así:

```text
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep`
thread 'main' panicked at 'index out of bounds: the len is 1
but the index is 1', src/main.rs:29:21
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

El índice de la línea `index out of bounds: the len is 1 but the index is 1` es un mensaje de error destinado a los programadores. No ayudará a nuestros
usuarios finales a comprender qué sucedió y qué deberían hacer en su lugar.
Arreglemos eso ahora.

#### Mejorando el mensaje de error

En el listado 12-8, agregamos un cheque en la función `new` que verificará
que el segmento sea lo suficientemente largo antes de acceder al índice 1
y 2. Si el segmento no es lo suficientemente largo, el programa entra en pánico
y muestra un mensaje de error mejor que el mensaje `index out of bounds`.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
// --snip--
fn new(args: &[String]) -> Config {
    if args.len() < 3 {
        panic!("not enough arguments");
    }
    // --snip--
```

<span class="caption">Listado 12-8: Agregar un control para la cantidad de
argumentos</span>

Este código es similar a la función `Guess::new` que escribimos en el Listado
9-9, donde llamamos `panic!` Cuando el argumento `value` estaba fuera del
rango de valores válidos. En lugar de buscar un rango de valores aquí,
estamos comprobando que la longitud de `args` es al menos 3 y el resto de la
función puede operar bajo la suposición de que se ha cumplido esta condición.
Si `args` tiene menos de tres elementos, esta condición será verdadera, y
llamamos a la macro `panic!` para finalizar el programa de inmediato.

Con estas pocas líneas de código extra en `new`, ejecutemos el programa sin
ningún argumento nuevamente para ver cómo se ve el error ahora:

```text
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep`
thread 'main' panicked at 'not enough arguments', src/main.rs:30:12
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

Este resultado es mejor: ahora tenemos un mensaje de error razonable. Sin
embargo, también tenemos información extraña que no queremos dar a nuestros
usuarios. Quizás usar la técnica que usamos en el Listado 9-9 no sea la mejor
para usar aquí: una llamada a `panic!` Es más apropiada para un problema de
programación que un problema de uso, como se discutió en el Capítulo 9. En su
lugar, podemos usar la otra técnica que aprendió en el Capítulo 9-devolviendo
un `Result` que indica éxito o un error.

#### Devolviendo un `Result` desde `new` en lugar de llamar `panic!`

En su lugar, podemos devolver un valor `Result` que contendrá una instancia
`Config` en el caso exitoso y describiremos el problema en el caso de error.
Cuando `Config::new` se está comunicando con `main`, podemos usar el tipo
`Result` para indicar que hubo un problema. Entonces podemos cambiar `main`
para convertir una variante `Err` en un error más práctico para nuestros
usuarios sin el texto circundante sobre `thread' main'` y `RUST_BACKTRACE`
que causa una llamada a `panic!`.

El listado 12-9 muestra los cambios que debemos hacer en el valor de retorno
de `Config::new` y el cuerpo de la función necesaria para devolver un
`Result`. Tenga en cuenta que esto no se compilará hasta que actualicemos
`main` también, lo cual haremos en la siguiente lista.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
impl Config {
    fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let filename = args[2].clone();

        Ok(Config { query, filename })
    }
}
```

<span class="caption">Listado 12-9: Devolviendo un `Result` desde
`Config::new`</span>

Nuestra función `new` ahora devuelve un `Result` con una instancia `Config`
en el caso de éxito y un `&'static str` en el caso de error. Recuerde de la
sección “The Static Lifetime” en el Capítulo 10 que `&'static str` es el tipo
de *string literals*, que es nuestro tipo de mensaje de error por el momento.

Hemos realizado dos cambios en el cuerpo de la función `new`: en lugar de
invocar `panic!` cuando el usuario no pasa suficientes argumentos, ahora
devolvemos un valor `Err`, y hemos envuelto `Config` devolver el valor en un
`Ok`. Estos cambios hacen que la función se ajuste a su nueva firma de tipo.

Devolver un valor `Err` de `Config::new` permite que la función `main` maneje
el valor `Result` devuelto por la función `new` y salga del proceso más
limpiamente en el caso de error.

#### Llamar a 'Config::new` y manejo de errores

Para manejar el caso de error e imprimir un mensaje fácil de usar,
necesitamos actualizar `main` para manejar el `Result` que devuelve
`Config:: new`, como se muestra en el Listado 12-10. También tomaremos la
responsabilidad de salir de la herramienta de línea de comandos con un código
de error distinto de cero de `panic!` e implementarlo a mano. Un estado de
salida distinto de cero es una convención para indicar al proceso que llamó a
nuestro programa que el programa salió con un estado de error.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use std::process;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    // --snip--
```

<span class="caption">Listado 12-10: Salir con un código de error si falla la
creación de una nueva `Config`</span>

En esta lista, hemos utilizado un método que no hemos cubierto anteriormente:
`unwrap_or_else`, que se define en `Result <T, E>` por la biblioteca estándar.
El uso de `unwrap_or_else` nos permite definir un manejo de error
personalizado, no `panic!`. Si el `Result` es un valor `Ok`, el
comportamiento de este método es similar para `unwrap`: devuelve el valor
interno `Ok` está envolviendo. Sin embargo, si el valor
es un valor `Err`, este método llama al código en *closure*, que es una
función anónima que definimos y pasamos como un argumento para
`unwrap_or_else`. Bien cubriremos los *closure* con más detalle en el
Capítulo 13. Por ahora, solo necesita saber que ese `unwrap_or_else` pasará
el valor interno del `Err`, que en este caso es el *string* estático
`no enough arguments` que agregamos en el listado 12-9,
a nuestro *closure* en el argumento `err` que aparece entre los
*pipes* (||)verticales. El código en el *closure* puede usar el valor `err`
cuando se ejecuta.

Hemos agregado una nueva línea `use` para importar `process` de la biblioteca
estándar. El código en el *closure*  se ejecutará en el caso de error, son
solo dos líneas: imprimimos el valor `err` y luego llamamos
`process::exit`. La función `process::exit` detendrá el programa
inmediatamente y devolverá el número que se pasó como código de estado de
salida. Esto es similar al manejo basado en `panic!` que utilizamos en el listado 12-8, pero ya no obtenemos todos los resultados adicionales.
Intentemos eso:

```text
$ cargo run
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48 secs
     Running `target/debug/minigrep`
Problem parsing arguments: not enough arguments
```

¡Estupendo! Este resultado es mucho más amigable para nuestros usuarios.

### Extrayendo la lógica de `main`

Ahora que hemos terminado de refactorizar el análisis de configuración,
veamos la lógica del programa. Como dijimos en “Separación de preocupaciones
para proyectos binarios”, vamos a extraer una función llamada `run` que
mantendrá toda la lógica actualmente en la función `main` que no está
relacionada con la configuración de la configuración o el manejo de errores.
Cuando hayamos terminado, `main` será conciso y fácil de verificar mediante
inspección, y podremos escribir pruebas para la otra lógica.

El listado 12-11 muestra la función extraída `run`. Por ahora, solo estamos
haciendo una mejora pequeña y gradual al extraer la función. Todavía estamos
definiendo la función en *src/main.rs*.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    // --snip--

    println!("Searching for {}", config.query);
    println!("In file {}", config.filename);

    run(config);
}

fn run(config: Config) {
    let mut f = File::open(config.filename).expect("file not found");

    let mut contents = String::new();
    f.read_to_string(&mut contents)
        .expect("something went wrong reading the file");

    println!("With text:\n{}", contents);
}

// --snip--
```

<span class="caption">Listado 12-11: Extracción de una función `run` que
contiene el resto de la lógica del programa</span>

La función `run` ahora contiene toda la lógica restante de `main`, comenzando
por leer el archivo. La función `run` toma la instancia `Config` como un
argumento.

#### Devolución de errores de la función `run`

Con la lógica restante del programa separada en la función `run`, podemos
mejorar el manejo de errores, como hicimos con `Config::new` en el Listado
12-9. En lugar de permitir que el programa entre en pánico al llamar a
`expect`, la función `run` devolverá un `Result <T, E>` cuando algo sale mal.
Esto nos permitirá consolidar aún más en `main` la lógica en torno al manejo
de errores de una manera fácil de usar. El listado 12-12 muestra los cambios
que debemos realizar en la firma y el cuerpo de `run`.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use std::error::Error;

// --snip--

fn run(config: Config) -> Result<(), Box<Error>> {
    let mut f = File::open(config.filename)?;

    let mut contents = String::new();
    f.read_to_string(&mut contents)?;

    println!("With text:\n{}", contents);

    Ok(())
}
```

<span class="caption">Listado 12-12: Cambiar la función `run` para devolver
`Result`</span>

Hemos realizado tres cambios significativos aquí. Primero, cambiamos el tipo
de retorno de la función `run` a `Result<(), Box<Error>>`. Esta función
devolvió previamente el tipo de unidad, `()`, y lo mantenemos como el valor
devuelto en el caso `Ok`.

Para el tipo de error, utilizamos el objeto *trait* `Box <Error>` (y hemos
llevado `std::error::Error` al ámbito con una instrucción `use` en la parte
superior). Cubriremos los objetos *trait* en el Capítulo 17. Por ahora, solo
saber que `Box<Error>` significa que la función devolverá un tipo que
implementa el *trait* de `Error`, pero no tenemos que especificar de qué tipo
particular será el valor de retorno. Esto nos da flexibilidad para devolver
valores de error que pueden ser de diferentes tipos en diferentes casos de
error.

En segundo lugar, hemos eliminado las llamadas a `expect` a favor del
operador `?`, como hablamos en el Capítulo 9. En lugar de `panic!` En un
error, el operador `?` devolverá el valor de error de la función actual para
que la persona que llama pueda manejarlo.

En tercer lugar, la función `ejecutar` ahora devuelve un valor `Ok` en el
caso de éxito. Hemos declarado el tipo de éxito de la función `run` como `()`
en la firma, lo que significa que debemos ajustar el valor del tipo de unidad
en el valor `Ok`. Esta sintaxis `Ok (())` puede parecer un poco extraña al
principio, pero usar `()` como este es la manera idiomática de indicar que
estamos llamando `run` solo por sus efectos secundarios; no devuelve un valor
que necesitamos.

Cuando ejecuta este código, compilará pero mostrará una advertencia:

```text
warning: unused `std::result::Result` which must be used
  --> src/main.rs:18:5
   |
18 |     run(config);
   |     ^^^^^^^^^^^^
= note: #[warn(unused_must_use)] on by default
```

Rust nos dice que nuestro código ignoró el valor `Result` y que el valor
`Result` podría indicar que ocurrió un error. ¡Pero no estamos verificando
si hubo un error o no, y el compilador nos recuerda que es probable que tengamos aquí algún código de manejo de errores! Vamos a rectificar ese
problema ahora.

#### Manejo de errores devueltos por `run` en `main`

Comprobaremos si hay errores y los manejaremos usando una técnica similar a la que usamos con `Config::new` en el listado 12-10, pero con una ligera
diferencia:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    // --snip--

    println!("Searching for {}", config.query);
    println!("In file {}", config.filename);

    if let Err(e) = run(config) {
        println!("Application error: {}", e);

        process::exit(1);
    }
}
```

Usamos `if let` en lugar de `unwrap_or_else` para comprobar si `run`
devuelve un valor `Err` y llama a `process::exit(1)` si es así. La función
`run` no devuelve un valor que queremos `unwrap` de la misma manera que
`Config::new` devuelve la instancia `Config`. Debido a que `run` devuelve
`()` en el caso de éxito, solo nos importa detectar un error, por lo que no
necesitamos `unwrap_or_else` para devolver el valor desenvuelto porque solo
sería `()`.

Los cuerpos de las funciones `if let` y `unwrap_or_else` son iguales en
ambos casos: imprimimos el error y salimos.

### División de código en un *Library Crate*

¡Nuestro proyecto `minigrep` se ve bien hasta ahora!. Ahora dividiremos el
archivo *src/main.rs* y colocaremos algún código en el archivo *src/lib.rs*
para que podamos probarlo y tener un archivo *src/main.rs* con menos
responsabilidades.

Vamos a mover todo el código que no es la función `main` de *src/main.rs* a
*src/lib.rs*:

* La definición de la función `run`
* Las declaraciones pertinentes de 'uso'
* La definición de `Config`
* La definición de la función `Config :: new`

El contenido de *src/lib.rs* debe tener las firmas que se muestran en el
listado 12-13 (hemos omitido los cuerpos de las funciones para abreviar).
Tenga en cuenta que esto no se compilará hasta que modifiquemos
*src/main.rs* en el listado 12-14.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
use std::error::Error;
use std::fs::File;
use std::io::prelude::*;

pub struct Config {
    pub query: String,
    pub filename: String,
}

impl Config {
    pub fn new(args: &[String]) -> Result<Config, &'static str> {
        // --snip--
    }
}

pub fn run(config: Config) -> Result<(), Box<Error>> {
    // --snip--
}
```

<span class="caption">Listado 12-13: Mover `Config` y `run` a
*src/lib.rs*</span>

Hemos hecho un uso liberal de la palabra clave `pub`: en `Config`, en sus
campos y en su método `new`, y en la función `run`. ¡Ahora tenemos un
*library crate* que tiene una API pública que podemos probar!

Ahora tenemos que llevar el código que cambiamos a *src/lib.rs* dentro del
alcance del *binary crate* en *src/main.rs*, como se muestra en el listado
12-14.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate minigrep;

use std::env;
use std::process;

use minigrep::Config;

fn main() {
    // --snip--
    if let Err(e) = minigrep::run(config) {
        // --snip--
    }
}
```

<span class="caption">Listado 12-14: Llevar el *crate* `minigrep` dentro del
alcance de *src/main.rs*</span>

Para traer el *library crate* a el *binary crate*, usamos
`extern crate minigrep`. Luego agregamos una línea `use minigrep::Config`
para poner el tipo `Config` en el alcance, y prefijamos la función `run` con
nuestro nombre del *crate*. Ahora toda la funcionalidad debe estar conectada
y debería funcionar. Ejecute el programa con `cargo run` y asegúrese de que
todo funcione correctamente.

¡Uf!. Eso fue mucho trabajo, pero nos hemos preparado para el éxito en el
futuro. Ahora es mucho más fácil manejar los errores y hemos hecho que el
código sea más modular. Casi todo nuestro trabajo se realizará en
*src/lib.rs* de aquí en adelante.

Aprovechemos esta nueva modularidad al hacer algo que hubiera sido difícil
con el código anterior, pero es fácil con el nuevo código: ¡escribiremos
algunas pruebas!
