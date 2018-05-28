## Leyendo un archivo

Ahora agregaremos funcionalidad para leer el archivo que se especifica en el
argumento de línea de comando `filename`. Primero, necesitamos un archivo de
muestra para probarlo: el mejor tipo de archivo para asegurarse de que
`minigrep` funcione es uno con una pequeña cantidad de texto en varias líneas
con algunas palabras repetidas. ¡El listado 12-3 tiene un poema de Emily
Dickinson que funcionará bien! Crea un archivo llamado *poem.txt* en el nivel
raíz de tu proyecto e ingresa el poema “I’m Nobody! Who are you?”

<span class="filename">Filename: poem.txt</span>

```text
I'm nobody! Who are you?
Are you nobody, too?
Then there's a pair of us - don't tell!
They'd banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

<span class="caption">Listado 12-3: Un poema de Emily Dickinson es un buen
caso de prueba</span>

Con el texto en su lugar, edite *src/main.rs* y agregue código para abrir el
archivo, como se muestra en el Listado 12-4.

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
use std::env;
use std::fs::File;
use std::io::prelude::*;

fn main() {
#     let args: Vec<String> = env::args().collect();
#
#     let query = &args[1];
#     let filename = &args[2];
#
#     println!("Searching for {}", query);
    // --snip--
    println!("In file {}", filename);

    let mut f = File::open(filename).expect("file not found");

    let mut contents = String::new();
    f.read_to_string(&mut contents)
        .expect("something went wrong reading the file");

    println!("With text:\n{}", contents);
}
```

<span class="caption">Listado 12-4: Lectura del contenido del archivo
especificado por el segundo argumento</span>

Primero, agregamos algunas declaraciones `use` más para traer partes
relevantes de la biblioteca estándar: necesitamos `std::fs::File` para
manejar archivos, y `std::io::prelude::*` contiene varios *traits* útiles
para hacer E/S, incluida la E/S de archivos. De la misma manera que Rust
tiene un preludio general que pone ciertos tipos y funciones en el alcance
automáticamente, el módulo `std::io` tiene su propio preludio de los tipos y
funciones comunes que necesitará al trabajar con E/S. A diferencia del
preludio predeterminado, debemos agregar explícitamente una instrucción `use`
para el preludio de `std::io`.

En `main`, hemos agregado tres enunciados: primero, obtenemos un manejador
mutable al archivo llamando a la función `File::open` y pasándole el valor de
la variable `filename`. En segundo lugar, creamos una variable llamada
`contents` y la configuramos como mutable, vacía `String`. Esto mantendrá el
contenido del archivo después de que lo hayamos leído. En tercer lugar,
llamamos `read_to_string` en nuestro identificador de archivo y pasamos una
referencia mutable a `contents` como argumento.

Después de esas líneas, hemos agregado nuevamente una declaración temporal
`println!` Que imprime el valor de `contents` después de que se lee el
archivo, para que podamos verificar que el programa esté funcionando hasta el
momento.

Vamos a ejecutar este código con cualquier cadena como el primer argumento de
línea de comando (porque aún no hemos implementado la parte de búsqueda) y el
archivo *poem.txt* como segundo argumento:

```text
$ cargo run the poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep the poem.txt`
Searching for the
In file poem.txt
With text:
I'm nobody! Who are you?
Are you nobody, too?
Then there's a pair of us - don't tell!
They'd banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

¡Estupendo! El código leyó y luego imprimió el contenido del archivo. Pero el
código tiene algunos defectos. La función `main` tiene múltiples
responsabilidades: generalmente, las funciones son más claras y fáciles de
mantener si cada función es responsable de una sola idea. El otro problema es
que no estamos manejando los errores tan bien como pudimos. El programa aún
es pequeño, por lo que estos defectos no son un gran problema, pero a medida
que el programa crezca, será más difícil solucionarlos de manera limpia. Es
una buena práctica comenzar a refactorizar al principio cuando se desarrolla
un programa, porque es mucho más fácil refactorizar cantidades más pequeñas
de código. Lo haremos a continuación.
