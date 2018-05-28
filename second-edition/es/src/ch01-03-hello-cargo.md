## Hola, Cargo!

Cargo es el sistema de compilación y el administrador de paquetes de Rust. La
mayoría de los Rustaceans utilizan esta herramienta para administrar sus
proyectos de Rust porque Cargo maneja muchas tareas para usted, como
construir su código, descargar las bibliotecas de las que depende su código y
construir esas bibliotecas. (Llamamos a las bibliotecas su código necesita
*dependencias*).

Los programas más simples de Rust, como el que hemos escrito hasta ahora, no
tienen ninguna dependencia. Entonces, si hubiéramos construido el proyecto
*Hello, world!* con Cargo, solo usaría la parte de Cargo que maneja la
construcción de su código. Al escribir programas de Rust más complejos,
agregará dependencias, y si inicia un proyecto utilizando Cargo, agregar
dependencias será mucho más fácil de hacer.

Debido a que la gran mayoría de los proyectos de Rust usan Cargo, el resto de
este libro asume que también está utilizando Cargo. Cargo viene instalado con
Rust si utilizaste los instaladores oficiales discutidos en la sección
“Instalación”. Si instaló Rust por otros medios, verifique si Cargo se
instala ingresando lo siguiente en su terminal:

```text
$ cargo --version
```

Si ves un número de versión, ¡ya lo tienes!. Si ve un error, como
`command not found`, consulte la documentación de su método de instalación
para determinar cómo instalar Cargo por separado.

### Creando un Proyecto con Cargo

Vamos a crear un nuevo proyecto utilizando Cargo y veamos cómo difiere de
nuestro proyecto original *Hello, world!*. Vuelva a su directorio
*projects* (o donde haya decidido almacenar su código). Luego, en cualquier
sistema operativo, ejecute lo siguiente:

```text
$ cargo new hello_cargo --bin
$ cd hello_cargo
```

El primer comando crea un nuevo ejecutable binario llamado *hello_cargo*. El
argumento `--bin` pasado a `cargo new` crea una aplicación ejecutable (a
menudo simplemente llamada *binary*) en lugar de una biblioteca. Hemos
nombrado nuestro proyecto *hello_cargo*, y Cargo crea sus archivos en un
directorio del mismo nombre.

Vaya al directorio *hello_cargo* y liste los archivos. Verá que Cargo ha
generado dos archivos y un directorio para nosotros: un archivo *Cargo.toml*
y un directorio *src* con un archivo *main.rs* dentro. También ha
inicializado un nuevo repositorio de Git junto con un archivo *.gitignore*.

> Nota: Git es un sistema de control de versiones común. Puede cambiar
> `cargo new` para usar un sistema de control de versiones diferente o ningún
> sistema de control de versiones usando el indicador `--vcs`. Ejecute
> `cargo new --help` para ver las opciones disponibles.

Abra *Cargo.toml* en su editor de texto de su elección. Debería verse simila
al código en el Listado 1-2.

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]

[dependencies]
```

<span class="caption">Listado 1-2: Contenido de *Cargo.toml* generado por
`cargo new`</span>

Este archivo está en formato [*TOML*][toml] <!-- ignore --> (*Tom's Obvious,
Minimal Language*), que es el formato de configuración de Cargo.

[toml]: https://github.com/toml-lang/toml

La primera línea, `[package]`, es un encabezado de sección que indica que las
siguientes declaraciones están configurando un paquete. A medida que
agreguemos más información a este archivo, agregaremos otras secciones.

Las siguientes tres líneas establecen la información de configuración que
Cargo necesita para compilar su programa: el nombre, la versión y quién lo
escribió. Cargo obtiene su nombre e información de correo electrónico de su
entorno, por lo que si esa información no es correcta, corrija la información
ahora y luego guarde el archivo.

La última línea, `[dependencies]`, es el comienzo de una sección para que
usted liste cualquiera de las dependencias de su proyecto. En Rust, los
paquetes de código se conocen como *crates*. No necesitaremos otros *crates*
para este proyecto, pero lo haremos en el primer proyecto en el Capítulo 2,
entonces usaremos esta sección de dependencias.

Ahora abra *src/main.rs* y eche un vistazo:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo ha generado un programa *Hello, world!* para ti, ¡como el que
escribimos en el Listado 1-1! Hasta ahora, las diferencias entre nuestro
proyecto anterior y el proyecto que Cargo genera son que Cargo colocó el
código en el directorio *src*, y tenemos un archivo de configuración
*Cargo.toml* en el directorio superior.

Cargo espera que sus archivos fuente vivan dentro del directorio *src*. El
directorio del proyecto de nivel superior es solo para archivos README,
información de licencia, archivos de configuración y cualquier otra cosa no
relacionada con su código. Usar Cargo te ayuda a organizar tus proyectos. Hay
un lugar para todo, y todo está en su lugar.

Si comenzó un proyecto que no usa Cargo, como lo hicimos con el proyecto
*Hello, world!*, puede convertirlo a un proyecto que sí utiliza Cargo. Mueva
el código del proyecto al directorio *src* y cree un archivo *Cargo.toml*
apropiado.

### Construir y ejecutar un proyecto de Cargo

Ahora veamos qué es diferente cuando construimos y ejecutamos el programa
*Hello, world!* con Cargo. Desde su directorio *hello_cargo*, construya su
proyecto ingresando el siguiente comando:

```text
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

Este comando crea un archivo ejecutable en *target/debug/hello_cargo*
(o *target\debug\hello_cargo.exe* en Windows) en lugar de en su directorio
actual. Puede ejecutar el ejecutable con este comando:

```text
$ ./target/debug/hello_cargo # or .\target\debug\hello_cargo.exe on Windows
Hello, world!
```

Si todo va bien, `Hello, world!` Debería imprimirse en la terminal. Ejecuta
`cargo build` por primera vez también hace que Cargo cree un nuevo archivo en
el nivel superior: *Cargo.lock*. Este archivo realiza un seguimiento de las
versiones exactas de las dependencias en su proyecto. Este proyecto no tiene
dependencias, por lo que el archivo es un poco escaso. Nunca necesitará
cambiar este archivo manualmente; Cargo maneja sus contenidos por usted.

Acabamos de crear un proyecto con `cargo build` y lo ejecutamos con
`./Target/debug/hello_cargo`, pero también podemos usar `cargo run` para
compilar el código y luego ejecutar el ejecutable resultante en un solo
comando:

```text
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Tenga en cuenta que esta vez no vimos la salida que indica que Cargo estaba
compilando `hello_cargo`. Cargo descubrió que los archivos no habían cambiado
por lo que simplemente ejecutó el binario. Si hubiera modificado su código
fuente, Cargo habría reconstruido el proyecto antes de ejecutarlo, y usted
habría visto esta salida:

```text
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo también proporciona un comando llamado `cargo check`. Este comando
verifica rápidamente su código para asegurarse de que compila pero no produce
un ejecutable:

```text
$ cargo check
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

¿Por qué no quieres un ejecutable?. A menudo, el `cargo check` es mucho más
rápido que `cargo build`, porque se salta el paso de producir un ejecutable.
Si continuamente revisas tu trabajo mientras escribes el código, ¡usar
`cargo check` acelerará el proceso! Como tal, muchos Rustaceas llevan a cabo
un `cargo check` periódicamente mientras escriben su programa para asegurarse
de que compila. Luego ejecutan `cargo build` cuando están listos para usar el
ejecutable.

Repasemos lo que hemos aprendido hasta ahora sobre Cargo:

* Podemos construir un proyecto usando `cargo build` o `cargo check`.
* Podemos construir y ejecutar un proyecto en un solo paso usando `cargo run`.
* En lugar de guardar el resultado de la construcción en el mismo directorio
 que nuestro código, Cargo lo almacena en el directorio *target/debug*.

Una ventaja adicional de usar Cargo es que los comandos son los mismos sin
importar en qué sistema operativo estés trabajando. Entonces, en este punto,
ya no brindaremos instrucciones específicas para Linux y macOS en comparación
con Windows.

### Building for Release

Cuando su proyecto finalmente esté listo para su lanzamiento, puede usar
`cargo build - release` para compilarlo con optimizaciones. Este comando
creará un ejecutable en *target/release* en lugar de *target/debug*. Las
optimizaciones hacen que su código Rust se ejecute más rápido, pero al
encenderlo, se alarga el tiempo que tarda su programa en compilarse. Esta es
la razón por la cual hay dos perfiles diferentes: uno para el desarrollo,
cuando desea reconstruir rápidamente y con frecuencia, y otro para construir
el programa final que le dará a un usuario que no se reconstruirá
repetidamente y que se ejecutará tan rápido como posible. Si está evaluando
el tiempo de ejecución de su código, asegúrese de ejecutar `carga build - release` y *benchmark* con el ejecutable en *target/release*.

### Cargo como Convención

Con proyectos simples, Cargo no le da mucho valor a simplemente usar `rustc`,
pero demostrará su valor a medida que sus programas se vuelven más
intrincados. Con proyectos complejos compuestos de múltiples *crates*, es
mucho más fácil dejar que Cargo coordine la construcción.

Aunque el proyecto `hello_cargo` es simple, ahora usa muchas de las
herramientas reales que usará en el resto de su carrera en Rust. De hecho,
para trabajar en cualquier proyecto existente, puede usar los siguientes
comandos para verificar el código usando Git, cambiar al directorio de ese
proyecto y compilar:

```text
$ git clone someurl.com/someproject
$ cd someproject
$ cargo build
```

Para obtener más información sobre Cargo, consulte [su documentación].

[su documentación]: https://doc.rust-lang.org/cargo/

## Resumen

¡Ya has tenido un gran comienzo en tu viaje a Rust! En este capítulo, has
aprendido a:

* Instale la última versión estable de Rust usando `rustup`
* Actualización a una versión más nueva de Rust
* Abrir documentación instalada localmente
* Escribir y ejecutar un programa *Hello, world!* que usa `rustc` directamente
* Crear y ejecutar un nuevo proyecto utilizando las convenciones de Cargo

Este es un buen momento para crear un programa más sustancial para
acostumbrarse a leer y escribir el código Rust. Entonces, en el Capítulo 2,
construiremos un programa de adivinanzas. Si prefiere comenzar aprendiendo
cómo funcionan los conceptos de programación común en Rust, consulte el
Capítulo 3 y luego regrese al Capítulo 2.
