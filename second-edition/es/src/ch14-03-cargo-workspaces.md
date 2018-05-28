## Espacios de trabajo de Cargo

En el Capítulo 12, creamos un paquete que incluía un *binary crate* y una
*library crate*. A medida que se desarrolle su proyecto, es posible que
descubra que la *library crate* sigue creciendo y desea dividir su paquete
aún más en varias *library crate*. En esta situación, Cargo ofrece una
función llamada *workspaces* que puede ayudar a administrar múltiples
paquetes relacionados que se desarrollan en tándem.

### Creando un espacio de trabajo

Un *workspace* es un conjunto de paquetes que comparten el mismo
*Cargo.lock* y el directorio de salida. Hagamos un proyecto usando un espacio
de trabajo: usaremos un código trivial para poder concentrarnos en la
estructura del espacio de trabajo. Hay varias formas de estructurar un
espacio de trabajo; vamos a mostrar una forma común. Tendremos un espacio de
trabajo que contiene un binario y dos bibliotecas. El binario, que
proporcionará la funcionalidad principal, dependerá de las dos bibliotecas.
Una biblioteca proporcionará una función `add_one`, y una segunda biblioteca
una función `add_two`. Estas tres cajas formarán parte del mismo espacio de
trabajo. Comenzaremos por crear un nuevo directorio para el área de trabajo:

```text
$ mkdir add
$ cd add
```

A continuación, en el directorio *add*, creamos el archivo *Cargo.toml* que
configurará todo el espacio de trabajo. Este archivo no tendrá una sección
`[package]` ni los metadatos que hemos visto en otros archivos
*Cargo.toml*. En cambio, comenzará con una sección `[workspace]` que nos
permitirá agregar miembros al espacio de trabajo al especificar la ruta a
nuestro *binary crate*; en este caso, esa ruta es *adder*:

<span class="filename">Filename: Cargo.toml</span>

```toml
[workspace]

members = [
    "adder",
]
```

A continuación, crearemos el *binary crate* `adder` ejecutando `cargo new`
dentro del directorio *add *:

```text
$ cargo new --bin adder
     Created binary (application) `adder` project
```

En este punto, podemos construir el espacio de trabajo ejecutando
`cargo build`. Los archivos en su directorio *add  deberían verse así:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

El espacio de trabajo tiene un directorio *target* en el nivel superior para
que se coloquen los artefactos compilados; el *crate* `adder` no tiene su
propio directorio *target*. Incluso si tuviéramos que ejecutar
`cargo build` desde el directorio *adder*, los artefactos compilados
terminarían en *add/target* en lugar de *add/adder/target*. Cargo estructura
el directorio *target* en un espacio de trabajo como este porque los *crates*
en un espacio de trabajo dependen uno del otro. Si cada *crate* tuviera su
propio directorio *target*, cada *crate* tendría que recompilar cada una de
los otros *crates* en el espacio de trabajo para tener los artefactos en su
propio directorio *target*. Al compartir un directorio *target*, los *crates*
pueden evitar reconstrucciones innecesarias.

### Crear el segundo *crate* en el espacio de trabajo

Luego, creemos otro *crate* miembros en el espacio de trabajo y llámela
`add-one`. Cambie el nivel superior *Cargo.toml* para especificar la ruta
*add-one* en la lista `members`:

<span class="filename">Filename: Cargo.toml</span>

```toml
[workspace]

members = [
    "adder",
    "add-one",
]
```

Luego genere un nuevo *library crate* llamada `add-one`:

```text
$ cargo new add-one
     Created library `add-one` project
```

Su directorio *add* ahora debería tener estos directorios y archivos:

```text
├── Cargo.lock
├── Cargo.toml
├── add-one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

En el archivo *add-one/src/lib.rs*, agreguemos una función `add_one`:

<span class="filename">Filename: add-one/src/lib.rs</span>

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

Ahora que tenemos un *library crate* en el espacio de trabajo, podemos hacer
que la *binary crate* `adder` dependa de la *library crate* `add-one`.
Primero, necesitaremos agregar una dependencia de ruta en `add-one` a
*adder/Cargo.toml*.

<span class="filename">Filename: adder/Cargo.toml</span>

```toml
[dependencies]

add-one = { path = "../add-one" }
```

Cargo no asume que los *crates* en un espacio de trabajo dependerán entre sí,
por lo que debemos ser explícitos sobre las relaciones de dependencia entre
los *crates*.

Luego, usemos la función `add_one` del *crate* `add-one` en el *crate*
`adder`. Abra el archivo *adder/src/main.rs* y agregue una línea
`extern crate` en la parte superior para poner el nuevo *library crate*
`add-one` en el alcance. Luego cambie la función `main` para llamar a la
función `add_one`, como en el Listado 14-7:

<span class="filename">Filename: adder/src/main.rs</span>

```rust,ignore
extern crate add_one;

fn main() {
    let num = 10;
    println!("Hello, world! {} plus one is {}!", num, add_one::add_one(num));
}
```

<span class="caption">Listado 14-7: Usando el *library crate* `add-one` del *crate* `adder`</span>

Construyamos el espacio de trabajo ejecutando `cargo build` en el directorio
*add* de nivel superior.

```text
$ cargo build
   Compiling add-one v0.1.0 (file:///projects/add/add-one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.68 secs
```

Para ejecutar el *binary crate* desde el directorio *add*, necesitamos
especificar qué paquete en el área de trabajo queremos usar utilizando el
argumento `-p` y el nombre del paquete con `cargo run`:

```text
$ cargo run -p adder
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

Esto ejecuta el código en *adder/src/main.rs*, que depende del *crate*
`add-one`.

#### Dependiendo de un *crate* externo en un espacio de trabajo

Observe que el espacio de trabajo tiene solo un archivo *Cargo.lock* en el
nivel superior del espacio de trabajo en lugar de tener un *Cargo.lock* en el
directorio de cada *crate*. Esto asegura que todos los *crate* están usando
la misma versión de todas las dependencias. Si agregamos el *crate* `rand` a
los archivos *adder/Cargo.toml* y *add-one/Cargo.toml*, Cargo resolverá los
dos en una versión de `rand` y registrará en la una *Cargo.lock*. Hacer que
todos los *crate* en el espacio de trabajo usen las mismas dependencias
significa que los *crate* en el espacio de trabajo siempre serán compatibles
entre sí. Agreguemos el *crate* `rand` a la sección `[dependencies]` en el
archivo *add-one/Cargo.toml* para poder usar el *crate* `rand` en el `add-one`
*crate*:

<span class="filename">Filename: add-one/Cargo.toml</span>

```toml
[dependencies]

rand = "0.3.14"
```

Ahora podemos agregar `extern crate rand;` al archivo *add-one/src/lib.rs*,
y construir todo el espacio de trabajo ejecutando `cargo build` en el
directorio *add* traerá y compilará el *crate* `rand`:

```text
$ cargo build
    Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading rand v0.3.14
   --snip--
   Compiling rand v0.3.14
   Compiling add-one v0.1.0 (file:///projects/add/add-one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 10.18 secs
```

El *Cargo.lock* de nivel superior ahora contiene información sobre la
dependencia de `add-one` en `rand`. Sin embargo, aunque `rand` se usa en
algún lugar del área de trabajo, no podemos usarlo en otros *crates* en el
área de trabajo a menos que agreguemos `rand` a sus archivos *Cargo.toml*
también. Por ejemplo, si agregamos `extern crate rand;` al archivo
*adder/src/main.rs* para el *crate* `adder`, obtendremos un error:

```text
$ cargo build
   Compiling adder v0.1.0 (file:///projects/add/adder)
error: use of unstable library feature 'rand': use `rand` from crates.io (see
issue #27703)
 --> adder/src/main.rs:1:1
  |
1 | extern crate rand;
```

Para arreglar esto, edite el archivo *Cargo.toml* para el *crate* `adder`
e indique que `rand` también es una dependencia para ese *crate*. Construir
el *crate* `adder` agregará `rand` a la lista de dependencias para `adder` en
*Cargo.lock*, pero no se descargarán copias adicionales de `rand`. Cargo ha
asegurado que cada *crate* en el espacio de trabajo que usa el *crate* `rand`
usará la misma versión. Usar la misma versión de `rand` en el espacio de
trabajo ahorra espacio porque no tendremos copias múltiples y asegura que los
*crates* en el espacio de trabajo serán compatibles entre sí.

#### Agregar una prueba a un espacio de trabajo

Para otra mejora, agreguemos una prueba de la función `add_one::add_one`
dentro del *crate* `add_one`:

<span class="filename">Filename: add-one/src/lib.rs</span>

```rust
pub fn add_one(x: i32) -> i32 {
    x + 1
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(3, add_one(2));
    }
}
```

Ahora ejecute `cargo test` en el directorio *add* de nivel superior:

```text
$ cargo test
   Compiling add-one v0.1.0 (file:///projects/add/add-one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27 secs
     Running target/debug/deps/add_one-f0253159197f7841

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/adder-f88af9d2cc175a5e

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests add-one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

La primera sección del resultado muestra que pasó la prueba `it_works` en el
*crate* `add-one`. La siguiente sección muestra que las pruebas de cero se
encontraron en el *crate* de `adder`, y luego la última sección muestra que
no se encontraron pruebas de documentación en el *crate* `add-one`. Al
ejecutar `cargo test` en un espacio de trabajo estructurado de esta manera,
se ejecutarán las pruebas de todos los *crates* en el área de trabajo.

También podemos ejecutar pruebas para un *crate* en particular en un espacio
de trabajo desde el directorio de nivel superior utilizando la bandera `-p` y
especificando el nombre del *crate* que queremos probar:

```text
$ cargo test -p add-one
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/add_one-b3235fea9a156f74

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests add-one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Este resultado muestra que `cargo test` solo ejecutó las pruebas para el
*crate* `add-one` y no ejecutó las pruebas de *crate* `adder`.

Si publica los *crates* en el espacio de trabajo en *https://crates.io/*,
cada *crate* en el espacio de trabajo deberá publicarse por separado. El
comando `cargo publish` no tiene un indicador `--all` ni un indicador `-p`, por lo que debe cambiar al directorio de cada *crate* y ejecutar
`cargo publish` en cada *crate* en el espacio de trabajo para publicar los
*crates*.

Para práctica adicional, agregue un *crate* `add-two` a este espacio de
trabajo de forma similar a el *crate* `add-one`

A medida que su proyecto crezca, considere utilizar un espacio de trabajo: es
más fácil comprender componentes individuales más pequeños que una gran
cantidad de código. Además, mantener los *crates* en un espacio de trabajo
puede facilitar la coordinación entre ellas si a menudo se cambian al mismo
tiempo.
