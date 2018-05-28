## Publicando una Crate en Crates.io

Hemos utilizado paquetes de [crates.io](https://crates.io) <!-- ignore -->
como dependencias de nuestro proyecto, pero también puede compartir su código
con otras personas publicando sus propios paquetes. El registro de *crate* en
[crates.io](https://crates.io) <!-- ignore --> distribuye el código fuente de
sus paquetes, por lo que principalmente aloja código que es de código abierto.

Rust y Cargo tiene características que ayudan a que el paquete publicado sea
más fácil de usar y de encontrar para la gente en primer lugar. Luego
hablaremos de algunas de estas características y luego explicaremos cómo
publicar un paquete.

### Hacer comentarios útiles sobre la documentación

Documentar con precisión sus paquetes ayudará a otros usuarios a saber cómo y
cuándo usarlos, por lo que vale la pena invertir el tiempo para escribir la
documentación. En el Capítulo 3, discutimos cómo comentar el código de Rust
usando dos barras, `//`. Rust también tiene un tipo particular de comentario
para la documentación, conocido convenientemente como un *comentario de
documentación*, que generará documentación en HTML. El HTML muestra el
contenido de los comentarios de la documentación para los elementos API
públicos destinados a los programadores interesados en saber cómo usar
*su* *crate* en lugar de cómo se implementa el *crate*.

Los comentarios de documentación utilizan tres barras inclinadas, `///`, en
lugar de dos y admiten la notación de reducción para formatear el texto.
Coloque los comentarios de documentación justo antes del artículo que están
documentando. El listado 14-1 muestra los comentarios de la documentación
para una función `add_one` en una *crate* llamada `my_crate`:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let five = 5;
///
/// assert_eq!(6, my_crate::add_one(5));
/// ```
pub fn add_one(x: i32) -> i32 {
    x + 1
}
```

<span class="caption">Listado 14-1: un comentario de documentación para una
función</span>

Aquí, damos una descripción de lo que hace la función `add_one`, comenzamos
una sección con el encabezado `Examples`, y luego proporcionamos un código
que demuestra cómo usar la función `add_one`. Podemos generar la
documentación HTML a partir de este comentario de documentación ejecutando
`cargo doc`. Este comando ejecuta la herramienta `rustdoc` distribuida con
Rust y coloca la documentación HTML generada en el directorio *target/doc*.

Para mayor comodidad, ejecutar `cargo doc --open` creará el HTML para la
documentación de su *crate* actual (así como la documentación de todas las
dependencias de su *crate*) y abrirá el resultado en un navegador web.
Navegue a la función `add_one` y verá cómo se representa el texto en los
comentarios de la documentación, como se muestra en la Figura 14-1:

<img alt="Rendered HTML documentation for the `add_one` function of `my_crate`" src="img/trpl14-01.png" class="center" />

<span class="caption">Figura 14-1: documentación HTML para la función
`add_one`</span>

#### Secciones de uso común

Usamos el encabezado Markdown `# Examples` en el listado 14-1 para crear una
sección en el HTML con el título “Examples”. Estas son algunas otras
secciones que los autores de *crate* usan comúnmente en su documentación:

* **Pánico**: los escenarios en los que la función documentada podría entrar
 en pánico. Los llamantes de la función que no desean que sus programas
 entren en pánico deben asegurarse de que no llamen a la función en estas situaciones.
* **Errores**: si la función devuelve un `Result`, describir los tipos de
 errores que pueden ocurrir y las condiciones que pueden provocar que se
 devuelvan estos errores puede ser útil para las personas que llaman para que
 puedan escribir código para manejar los diferentes tipos de errores de
 diferentes maneras.
* **Seguridad**: si la función es `unsafe` para llamar (hablamos de
 inseguridad en el Capítulo 19), debe haber una sección que explique por qué
 la función no es segura y que cubra las invariantes que la función espera
 que defiendan las personas que llaman.

La mayoría de los comentarios de la documentación no necesitan todas estas secciones, pero esta es una buena lista de verificación para recordarle los aspectos de su código que las personas que llaman a su código estarán interesadas en conocer.

#### Comentarios de la documentación como pruebas

Agregar bloques de código de ejemplo en los comentarios de su documentación
puede ayudar a demostrar cómo usar su biblioteca, y hacerlo tiene una ventaja
adicional: ejecutar `cargo test` ejecutará los ejemplos de código en su
documentación como pruebas. Nada es mejor que la documentación con ejemplos.
Pero nada es peor que los ejemplos que no funcionan porque el código ha
cambiado desde que se escribió la documentación. Si ejecutamos `cargo test`
con la documentación para la función `add_one` del Listado 14-1, veremos una
sección en los resultados de prueba de esta manera:

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Ahora si cambiamos la función o el ejemplo para que `assert_eq!` En el
ejemplo pánico y ejecutemos `cargo test` nuevamente, veremos que las pruebas
de doc detectan que el ejemplo y el código no están sincronizados entre sí.

#### Comentar los elementos contenidos

Otro estilo de comentario de documento, `//!`, agrega documentación al
elemento que contiene los comentarios en lugar de agregar documentación a los
elementos que siguen a los comentarios. Normalmente utilizamos estos
comentarios de documento dentro del archivo raíz del *crate* (*src/lib.rs*
por convención) o dentro de un módulo para documentar el *crate* o el módulo
como un todo.

Por ejemplo, si queremos agregar documentación que describa el propósito del
*crate* `my_crate` que contiene la función `add_one`, podemos agregar
comentarios de documentación que comiencen con `//!` Al comienzo de
*src/lib.rs* archivo, como se muestra en el listado 14-2:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
//! # My Crate
//!
//! `my_crate` is a collection of utilities to make performing certain
//! calculations more convenient.

/// Adds one to the number given.
// --snip--
```

<span class="caption">Listado 14-2: Documentación para el *crate* `my_crate`
como un todo</span>

Observe que no hay ningún código después de la última línea que comienza con
`//!`. Como comenzamos los comentarios con `//!` en lugar de `///`, estamos
documentando el elemento que contiene este comentario en lugar de un elemento
que sigue a este comentario. En este caso, el elemento que contiene este
comentario es el archivo *src/lib.rs*, que es la raíz del *crate*. Estos
comentarios describen todo el *crate*.

Cuando ejecutamos `cargo doc --open`, estos comentarios se mostrarán en la
página principal de la documentación de `my_crate` sobre la lista de
elementos públicos en el *crate*, como se muestra en la Figura 14-2:

<img alt="Rendered HTML documentation with a comment for the crate as a whole" src="img/trpl14-02.png" class="center" />

<span class="caption">Figura 14-2: documentación generada para `my_crate`,
incluido el comentario que describe el *crate* como un todo</span>

Los comentarios de documentación dentro de los items son útiles para
describir *crates* y módulos especialmente. Úselos para explicar el propósito
general del contenedor para ayudar a los usuarios a entender la organización
del *crate*.

### Exportación de una API pública convenientemente con `pub use`

En el Capítulo 7, cubrimos cómo organizar nuestro código en módulos usando la
palabra clave `mod`, cómo hacer que los elementos sean públicos usando la
palabra clave `pub`, y cómo incluir elementos en un ámbito con la palabra
clave `use`. Sin embargo, la estructura que tiene sentido para usted mientras
desarrolla un *crate* puede no ser muy conveniente para sus usuarios. Es
posible que desee organizar sus estructuras en una jerarquía que contenga
varios niveles, pero las personas que quieran usar un tipo que haya definido
en lo más profundo de la jerarquía pueden tener problemas para descubrir que
ese tipo existe. También pueden molestarse al tener que ingresar `use`
`my_crate::some_module::another_module::UsefulType;` en lugar de `use`
`my_crate::UsefulType;`.

La estructura de su API pública es una consideración importante cuando se
publica un *crate*. Las personas que utilizan su *crate* están menos
familiarizadas con la estructura que usted y pueden tener dificultades para
encontrar las piezas que desean utilizar si su *crate* tiene una gran
jerarquía de módulos.

La buena noticia es que si la estructura *no es* conveniente para que otros
la utilicen en otra biblioteca, no tiene que reorganizar su organización
interna: en su lugar, puede volver a exportar elementos para crear una estructura pública diferente a la suya. estructura privada mediante el uso de
`pub`. La reexportación toma un elemento público en una ubicación y lo hace
público en otra ubicación, como si estuviera definido en la otra ubicación.

Por ejemplo, digamos que creamos una biblioteca llamada `art` para modelar
conceptos artísticos. Dentro de esta biblioteca hay dos módulos: un módulo
`clases` que contiene dos enumeraciones llamadas `PrimaryColor` y
`SecondaryColor` y un módulo `utils` que contiene una función llamada `mix`,
como se muestra en el Listado 14-3:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
//! # Art
//!
//! A library for modeling artistic concepts.

pub mod kinds {
    /// The primary colors according to the RYB color model.
    pub enum PrimaryColor {
        Red,
        Yellow,
        Blue,
    }

    /// The secondary colors according to the RYB color model.
    pub enum SecondaryColor {
        Orange,
        Green,
        Purple,
    }
}

pub mod utils {
    use kinds::*;

    /// Combines two primary colors in equal amounts to create
    /// a secondary color.
    pub fn mix(c1: PrimaryColor, c2: PrimaryColor) -> SecondaryColor {
        // --snip--
    }
}
```

<span class="caption">Listado 14-3: Una biblioteca `art` con elementos
organizados en módulos `types` y `utils`</span>

La figura 14-3 muestra cómo se vería la página principal de la documentación
para esta caja generada por `cargo doc`:


<img alt="Rendered documentation for the `art` crate that lists the `kinds` and `utils` modules" src="img/trpl14-03.png" class="center" />

<span class="caption">Figura 14-3: Front page de la documentación para
`art` que enumera los módulos `clases` y `utils`</span>

Tenga en cuenta que los tipos `PrimaryColor` y `SecondaryColor` no aparecen
en la página principal, ni tampoco la función `mix`. Tenemos que hacer clic
en `types` y `utils` para verlos.

Otro *crate* que depende de esta biblioteca necesitaría declaraciones `use`
que importen los elementos de `art`, especificando la estructura del módulo
que está actualmente definida. El listado 14-4 muestra un ejemplo de un
*crate* que usa los elementos `PrimaryColor` y `mix` de `art` crate:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate art;

use art::kinds::PrimaryColor;
use art::utils::mix;

fn main() {
    let red = PrimaryColor::Red;
    let yellow = PrimaryColor::Yellow;
    mix(red, yellow);
}
```

<span class="caption">Listado 14-4: Una *crate* que utiliza los items del *crate* `art` con su estructura interna exportada</span>

El autor del código en el listado 14-4, que usa el *crate* `art`, tuvo que
descubrir que `PrimaryColor` está en el módulo `kinds` y `mix` está en el
módulo `utils`. La estructura del módulo del *crate* de `art` es más
relevante para los desarrolladores que trabajan en el *crate* `art` que para
los desarrolladores que usan el *crate* `art`. La estructura interna que
organiza las partes del *crate* en el módulo `kinds` y el módulo `utils` no
contiene ninguna información útil para alguien que intente comprender cómo
usar el *crate* `art`. En cambio, la estructura del módulo `art` crate causa
confusión porque los desarrolladores tienen que averiguar dónde mirar, y la
estructura es inconveniente porque los desarrolladores deben especificar los
nombres de los módulos en las declaraciones `use`.

Para eliminar la organización interna de la API pública, podemos modificar el
código del *crate* `art` en el Listado 14-3 para agregar las declaraciones
`pub use` para reexportar los elementos en el nivel superior, como se muestra
en el Listado 14-5:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
//! # Art
//!
//! A library for modeling artistic concepts.

pub use kinds::PrimaryColor;
pub use kinds::SecondaryColor;
pub use utils::mix;

pub mod kinds {
    // --snip--
}

pub mod utils {
    // --snip--
}
```

<span class="caption">Listado 14-5: Agregar declaraciones `pub use` para
reexportar ítems</span>

La documentación API que `cargo doc` genera para este *crate* ahora mostrará
una lista y vinculará las reexportaciones en la página principal, como se
muestra en la Figura 14-4, haciendo que los tipos `PrimaryColor` y
`SecondaryColor` y la función `mix` sean más fáciles encontrar.

<img alt="Rendered documentation for the `art` crate with the re-exports on the front page" src="img/trpl14-04.png" class="center" />

<span class="caption">Figura 14-4: La página principal de la documentación
para `art` que enumera las reexportaciones</span>

Los usuarios de `art` *crate* pueden ver y usar la estructura interna del
Listado 14-3 como se muestra en el Listado 14-4, o pueden usar la estructura
más conveniente en el Listado 14-5, como se muestra en el Listado 14-6:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate art;

use art::PrimaryColor;
use art::mix;

fn main() {
    // --snip--
}
```

<span class="caption">Listado 14-6: Un programa que utiliza los elementos
reexportados del *crate* `art`</span>

En los casos en que hay muchos módulos anidados, la reexportación de los
tipos en el nivel superior con `uso de pub` puede marcar una diferencia
significativa en la experiencia de las personas que usan el *crate*.

Crear una estructura de API pública útil es más un arte que una ciencia, y
puede iterar para encontrar la API que funcione mejor para sus usuarios.
Elegir `pub use` le da flexibilidad en la forma de estructurar su *crate*
internamente y desacopla esa estructura interna de lo que presenta a sus
usuarios. Observe algunos de los códigos de *crates* que ha instalado para
ver si su estructura interna difiere de su API pública.

### Configurando una cuenta Crates.io

Antes de poder publicar cualquier *crate*, necesita crear una cuenta en
[crates.io](https://crates.io)<!-- ignore --> y obtener un token de API.
Para hacerlo, visite la página de inicio en
[crates.io](https://crates.io<!-- ignore --> e inicie sesión a través de una
cuenta de GitHub.
(La cuenta de GitHub es actualmente un requisito, pero el sitio podría
respaldar otras formas de crear una cuenta en el futuro). Una vez que haya
iniciado sesión, visite la configuración de su cuenta en
[https://crates.io/me/]( https://crates.io/me/)<!-- ignore --> y recupere su
clave API. Luego ejecute el comando `cargo login` con su clave API, de esta
manera:

```text
$ cargo login abcdefghijklmnopqrstuvwxyz012345
```

Este comando informará a Cargo de su token API y lo almacenará localmente en
*~/.carg/credentials*. Tenga en cuenta que este token es *secreto*: no lo
comparta con nadie más. Si lo comparte con alguien por cualquier motivo, debe
revocarlo y generar un nuevo token en
[crates.io](https://crates.io)<!-- ignore -->.

### Agregar metadatos a un nuev *crate*

Ahora que tiene una cuenta, digamos que tiene un *crate* que desea publicar.
Antes de publicar, deberá agregar algunos metadatos a su *crate* agregándolos
a la sección `[package]` del archivo *Cargo.toml* del *crate*.

Su *crate* necesitará un nombre único. Mientras trabajas en un *crate*
localmente, puedes nombrar un *crate* como quieras. Sin embargo, los nombres
de los *crates* en [crates.io](https://crates.io)<!-- ignore --> se asignan
por orden de llegada. Una vez que se toma el nombre de una *crate*, nadie más
puede publicar un *crate* con ese nombre. Busque el nombre que desea utilizar
en el sitio para averiguar si se ha utilizado. Si no lo ha hecho, edite el
nombre en el archivo *Cargo.toml* debajo de `[package]` para usar el nombre
para publicar, así:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

Incluso si ha elegido un nombre único, cuando ejecuta `cargo publish` para
publicar el *crate* en este punto, recibirá una advertencia y luego un error:

```text
$ cargo publish
    Updating registry `https://github.com/rust-lang/crates.io-index`
warning: manifest has no description, license, license-file, documentation,
homepage or repository.
--snip--
error: api errors: missing or empty metadata fields: description, license.
```

La razón es que falta información crucial: se requiere una descripción y una
licencia para que las personas sepan qué hace su *crate* y bajo qué términos
pueden usarla. Para rectificar este error, debe incluir esta información en
el archivo *Cargo.toml*.

Agregue una descripción que sea solo una oración o dos, porque aparecerá con
su *crate* en los resultados de búsqueda. Para el campo `license`, debe
proporcionar un *valor de identificador de licencia*
(*license identifier value*). El [Linux Foundation’s Software Package Data
Exchange (SPDX)][spdx] enumera los identificadores que puede usar para este valor. Por ejemplo, para especificar que ha licenciado su *crate* usando la
licencia MIT, agregue el identificador `MIT`:

[spdx]: http://spdx.org/licenses/

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

Si desea utilizar una licencia que no aparece en el SPDX, debe colocar el
texto de esa licencia en un archivo, incluir el archivo en su proyecto y
luego usar `license-file` para especificar el nombre de esa licencia. archivo
en lugar de usar la clave `license`.

La orientación sobre qué licencia es apropiada para su proyecto está más allá
del alcance de este libro. Muchas personas en la comunidad de Rust licencian
sus proyectos de la misma manera que Rust usando una licencia dual de
`MIT OR Apache-2.0`. Esta práctica demuestra que también puede especificar
múltiples identificadores de licencia separados por `OR` para tener múltiples
licencias para su proyecto.

Con un nombre único, la versión, el autor detalla que `cargo new` agregó
cuando creó el *crate*, su descripción y una licencia agregada, el archivo
*Cargo.toml* para un proyecto que está listo para publicar podría verse así:

<span class="filename">Filename: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
description = "A fun game where you guess what number the computer has chosen."
license = "MIT OR Apache-2.0"

[dependencies]
```

[Cargo’s documentation](https://doc.rust-lang.org/cargo/) describe otros
metadatos que puede especificar para asegurarse de que otros puedan descubrir
y usar su *crate* más fácilmente.

### Publicando en Crates.io

Ahora que ha creado una cuenta, ha guardado su token de API, ha elegido un
nombre para su *crate* y ha especificado los metadatos necesarios, ¡está
listo para publicar! al publicar un *crate*, se carga una versión específica
a [crates.io](https://crates.io) <!-- ignore --> para que otros la usen.

Tenga cuidado al publicar un *crate* porque una publicación es *permanente*.
La versión nunca se puede sobrescribir, y el código no se puede eliminar. Uno
de los principales objetivos de [crates.io](https://crates.io)<!-- ignore -->
es actuar como un archivo permanente de código para que las construcciones de
todos los proyectos que dependen de los *crates* de
[crates.io ](https://crates.io) <!-- ignore --> continuará funcionando.
Permitir eliminaciones de versiones haría imposible cumplir ese objetivo. Sin
embargo, no hay límite para la cantidad de versiones de *crates* que puede
publicar.

Ejecute el comando `cargo publish` nuevamente. Debería tener éxito ahora:

```text
$ cargo publish
 Updating registry `https://github.com/rust-lang/crates.io-index`
Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
 Finished dev [unoptimized + debuginfo] target(s) in 0.19 secs
Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
```

¡Felicitaciones! Ahora has compartido tu código con la comunidad Rust, y
cualquiera puede agregar fácilmente tu *crate* como una dependencia de su
proyecto.

### Publicar una nueva versión de un *crate* existente

Cuando haya realizado cambios en su *crate* y esté listo para lanzar una
nueva versión, usted cambiará el valor de `version` especificado en su
archivo *Cargo.toml* y volverá a publicar. Utilice las
[Reglas de control de versiones semánticas][semver] para decidir cuál es el
número de la próxima versión apropiada según los tipos de cambios que haya
realizado. Luego ejecute `cargo publish` para cargar la nueva versión.

[semver]: http://semver.org/

### Eliminando versiones de Crates.io con `cargo yank`

Aunque no puede eliminar versiones anteriores de un *crate*, puede evitar que
futuros proyectos las agreguen como una nueva dependencia. Esto es útil
cuando una versión de *crate* está rota por una razón u otra. En tales
situaciones, Cargo admite *yanking* una versión del *crate*.

El uso de una versión de Yankea impide que los proyectos nuevos comiencen a
depender de esa versión y permite que todos los proyectos existentes que
dependen de ella continúen descargándose y dependan de esa versión.
Esencialmente, un *yank* significa que todos los proyectos con un
*Cargo.lock* no se romperán, y cualquier futuro *Cargo.lock* generado no
usará la versión *yanked*.

Para *yank* una versión de un *crate*, ejecute `cargo yank` y especifique qué
versión para *yank*:

```text
$ cargo yank --vers 1.0.1
```

Al agregar `--undo` al comando, también puede deshacer un *yank* y permitir
que los proyectos comiencen dependiendo de una versión nuevamente:

```text
$ cargo yank --vers 1.0.1 --undo
```

Un *yank* *no* borra ningún código. Por ejemplo, la función *yank* no está
diseñada para borrar secretos cargados accidentalmente. Si eso sucede, debe
reajustar esos secretos de inmediato.
