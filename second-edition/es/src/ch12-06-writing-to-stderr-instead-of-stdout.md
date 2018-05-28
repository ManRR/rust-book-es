## Escribir mensajes de error a *Standard Error* en lugar de *Standard Output*

Por el momento, estamos escribiendo toda nuestra salida al terminal usando la
función `println!`. La mayoría de los terminales proporcionan dos tipos de
resultados: *salida estándar* (`stdout`) para información general y
*error estándar* (`stderr`) para mensajes de error. Esta distinción permite a
los usuarios elegir dirigir la salida exitosa de un programa a un archivo
pero aún así imprimir mensajes de error en la pantalla.

La función `println!` solo es capaz de imprimir en salida estándar, por lo
que tenemos que usar algo más para imprimir a un a *error estándar*.

### Comprobando dónde se escriben los errores

Primero, observemos cómo el contenido impreso por `minigrep` se está
escribiendo actualmente en la salida estándar, incluyendo cualquier mensaje
de error que deseemos escribir en error estándar. Lo haremos redireccionando
la secuencia de salida estándar a un archivo y, a la vez, provocando un error
intencionalmente. No redirigiremos la secuencia de error estándar, por lo que
cualquier contenido enviado a un error estándar continuará apareciendo en la
pantalla.

Se espera que los programas de línea de comando envíen mensajes de error al
flujo de error estándar para que podamos ver los mensajes de error en la
pantalla incluso si redirigimos el flujo de salida estándar a un archivo.
Nuestro programa actualmente no se comporta bien: ¡estamos a punto de ver que
en su lugar, guarda el resultado del mensaje de error en un archivo!

La forma de demostrar este comportamiento es ejecutando el programa con `>` y
el nombre de archivo, *output.txt*, al que queremos redirigir la secuencia de
salida estándar. No pasaremos ningún argumento, que debería causar un error:

```text
$ cargo run > output.txt
```

La sintaxis `>` le dice al shell que escriba el contenido del resultado
estándar en *output.txt* en lugar de la pantalla. No vimos el mensaje de
error que esperábamos que se imprimiera en la pantalla, lo que significa que
debe haber terminado en el archivo. Esto es lo que *output.txt* contiene:

```text
Problem parsing arguments: not enough arguments
```

Sí, nuestro mensaje de error se está imprimiendo en la salida estándar. Es
mucho más útil que los mensajes de error como este se impriman en un error
estándar para que solo los datos de una ejecución exitosa terminen en el
archivo. Cambiaremos eso.

### Impresión de errores en el error estándar

Usaremos el código en el listado 12-24 para cambiar cómo se imprimen los
mensajes de error. Debido a la refactorización que hicimos anteriormente en
este capítulo, todo el código que imprime mensajes de error está en una
función, `main`. La biblioteca estándar proporciona la macro `eprintln!` Que
se imprime en la secuencia de error estándar, así que cambiemos los dos
lugares a los que estábamos llamando `println!` Para imprimir los errores
para usar `eprintln!` En su lugar.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    if let Err(e) = minigrep::run(config) {
        eprintln!("Application error: {}", e);

        process::exit(1);
    }
}
```

<span class="caption">Listado 12-24: Escribir mensajes de error a error
estándar en lugar de salida estándar usando `eprintln!`</span>

Después de cambiar `println!` a `eprintln!`, Ejecutemos el programa
nuevamente de la misma manera, sin ningún argumento y redirigiendo la salida
estándar con `>`:

```text
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Ahora vemos el error en pantalla y *output.txt* no contiene nada, que es el
comportamiento que esperamos de los programas de línea de comandos.

Ejecutamos el programa de nuevo con argumentos que no causan un error, pero
que redirigen la salida estándar a un archivo, de la siguiente manera:

```text
$ cargo run to poem.txt > output.txt
```

No veremos ningún resultado en el terminal, y *output.txt* contendrá nuestros
resultados:

<span class="filename">Filename: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

Esto demuestra que ahora estamos usando salida estándar para salida exitosa y
error estándar para salida de error según corresponda.

## Resumen

Este capítulo resumió algunos de los principales conceptos que ha aprendido
hasta ahora y cubrió cómo realizar operaciones de E/S comunes en Rust. Al
usar argumentos de línea de comandos, archivos, variables de entorno y la
macro `eprintln!` Para errores de impresión, ahora está preparado para
escribir aplicaciones de línea de comandos. Al utilizar los conceptos de
capítulos anteriores, su código estará bien organizado, almacenará los datos
de manera efectiva en las estructuras de datos apropiadas, manejará los
errores de manera adecuada y se pondrá a prueba.

A continuación, exploraremos algunas características de Rust que se vieron
influenciadas por los lenguajes funcionales: cierres e iteradores.