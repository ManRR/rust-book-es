## Aceptando argumentos de línea de comandos

Vamos a crear un nuevo proyecto con `cargo new`, como siempre. Llamaremos a
nuestro proyecto `minigrep` para distinguirlo de la herramienta `grep` que ya
puede tener en su sistema.

```text
$ cargo new --bin minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

La primera tarea es hacer que `minigrep` acepte sus dos argumentos de línea
de comando: el nombre de archivo y una cadena para buscar. Es decir, queremos
poder ejecutar nuestro programa con `cargo run`, una cadena para buscar y una
ruta a un archivo para buscar, de esta forma:

```text
$ cargo run searchstring example-filename.txt
```

En este momento, el programa generado por `cargo new` no puede procesar los
argumentos que le damos. Algunas bibliotecas existentes en
[Crates.io](https://crates.io/) pueden ayudarlo a escribir un programa que
acepte argumentos de línea de comando, pero debido a que recién está
aprendiendo este concepto, implementemos esta capacidad nosotros mismos.

### Leyendo los valores de argumento

Para permitir que `minigrep` lea los valores de los argumentos de línea de
comando que le pasamos, necesitaremos una función provista en la biblioteca
estándar de Rust, que es `std::env::args`. Esta función devuelve un
*iterador* de los argumentos de la línea de comandos que se le dieron a
`minigrep`. Todavía no hemos discutido los iteradores (los trataremos
completamente en el Capítulo 13), pero por ahora, solo necesita conocer dos
detalles sobre los iteradores: los iteradores producen una serie de valores,
y podemos llamar al método `collect` en un iterador para convertirlo en una
colección, como un vector, que contiene todos los elementos que produce el
iterador.

Use el código en el listado 12-1 para permitir que su programa `minigrep` lea
cualquier argumento de línea de comando que se le pase y luego recopile los
valores en un vector.

<span class="filename">Filename: src/main.rs</span>

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();
    println!("{:?}", args);
}
```

<span class="caption">Listado 12-1: Recolectar los argumentos de la línea de
comando en un vector e imprimirlos</span>

Primero, traemos el módulo `std::env` dentro del alcance con una declaración
`use` para que podamos usar su función `args`. Observe que la función
`std::env::args` está anidada en dos niveles de módulos. Como discutimos en
el Capítulo 7, en los casos en que la función deseada está anidada en más de
un módulo, es convencional llevar el módulo principal al alcance en lugar de
a la función. Al hacerlo, podemos usar fácilmente otras funciones de
`std::env`. También es menos ambiguo que agregar `use std::env::args` y
llamar a la función con `args`, porque `args` puede confundirse fácilmente
con una función que está definida en el módulo actual.

> ### La función `args` y Unicode inválido
>
> Tenga en cuenta que `std::env::args` entrará en pánico si algún argumento
> contiene Unicode no válido. Si su programa necesita aceptar argumentos que
> contengan Unicode no válido, usa `std::env::args_os` en su lugar. Esa
> función devuelve un iterador que produce valores `OsString` en lugar de
> valores `String`. Hemos elegido usar `std::env::args` aquí para simplificar
> porque los valores `OsString` difieren según la plataforma y son más
> complejos para trabajar que los valores `String`.

En la primera línea de `main`, llamamos `env::args`, e inmediatamente usamos
`collect` para convertir el iterador en un vector que contiene todos los
valores producidos por el iterador. Podemos usar la función `collect` para
crear muchos tipos de colecciones, por lo que anotamos explícitamente el tipo
de `args` para especificar que queremos un vector de cadenas. Aunque muy
raramente necesitamos anotar tipos en Rust, `collect` es una función que a
menudo necesita anotar porque Rust no puede inferir el tipo de colección que
desea.

Finalmente, imprimimos el vector usando el formateador de depuración, `:?`.
Intentemos ejecutar el código primero sin argumentos y luego con dos
argumentos:

```text
$ cargo run
--snip--
["target/debug/minigrep"]

$ cargo run needle haystack
--snip--
["target/debug/minigrep", "needle", "haystack"]
```

Tenga en cuenta que el primer valor en el vector es
`"target/debug/minigrep "`, que es el nombre de nuestro binario. Esto
coincide con el comportamiento de la lista de argumentos en C, permitiendo
que los programas usen el nombre con el que se invocaron en su ejecución. A
menudo es conveniente tener acceso al nombre del programa en caso de que
desee imprimirlo en mensajes o cambiar el comportamiento del programa en
función de qué alias de línea de comando se utilizó para invocar el programa.
Pero a los fines de este capítulo, lo ignoraremos y guardaremos solo los dos
argumentos que necesitamos.

### Guardar los valores de argumento en variables

Al imprimir el valor del vector de argumentos se ilustró que el programa
puede acceder a los valores especificados como argumentos de línea de
comando. Ahora necesitamos guardar los valores de los dos argumentos en
variables para poder usar los valores en el resto del programa. Hacemos eso
en el listado 12-2.

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let filename = &args[2];

    println!("Searching for {}", query);
    println!("In file {}", filename);
}
```

<span class="caption">Listado 12-2: Creación de variables para contener el
argumento de consulta y el argumento de nombre de archivo</span>

Como vimos cuando imprimimos el vector, el nombre del programa toma el primer
valor en el vector en `args[0]`, por lo que estamos comenzando en el índice
`1`. El primer argumento que toma `minigrep` es la cadena que estamos
buscando, por lo que ponemos una referencia al primer argumento en la
variable `query`. El segundo argumento será el nombre de archivo, por lo que
ponemos una referencia al segundo argumento en la variable `filename`.

Imprimimos temporalmente los valores de estas variables para probar que el
código está funcionando como queremos. Ejecutamos este programa nuevamente
con los argumentos `test` y `sample.txt`:

```text
$ cargo run test sample.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep test sample.txt`
Searching for test
In file sample.txt
```

¡Genial, el programa está funcionando! Los valores de los argumentos que
necesitamos se guardan en las variables correctas. Más adelante, agregaremos
algún tipo de manejo de errores para tratar ciertas situaciones erróneas
potenciales, como cuando el usuario no proporciona ningún argumento; por
ahora, ignoraremos esa situación y trabajaremos en agregar capacidades de
lectura de archivos.
