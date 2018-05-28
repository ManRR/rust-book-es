## `mod` y el sistema de archivos

Comenzaremos nuestro ejemplo de módulo haciendo un nuevo proyecto con Cargo,
pero en lugar de crear una *binary crate*, crearemos una *library crate*: un
proyecto que otras personas pueden incorporar a sus proyectos como una
dependencia. Por ejemplo, la caja `rand` discutida en el Capítulo 2 es una
*library crate* que usamos como una dependencia en el proyecto del juego de
adivinanzas.

Crearemos un esqueleto de una *library* (*biblioteca*) que proporciona alguna
funcionalidad general de red; nos concentraremos en la organización de los
módulos y funciones, pero no nos preocuparemos por qué código va en los
cuerpos de funciones. Llamaremos a nuestra biblioteca `communicator`. Para
crear una biblioteca, pase la opción `--lib` en lugar de `--bin`:

```text
$ cargo new communicator --lib
$ cd communicator
```

Tenga en cuenta que Cargo generó *src/lib.rs* en lugar de *src/main.rs*. Dentro de *src/lib.rs* encontraremos lo siguiente:

<span class="filename">Filename: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

Cargo crea una prueba de ejemplo para ayudarnos a iniciar nuestra biblioteca,
en lugar del binario “Hello, world!” Que obtenemos cuando usamos la opción
`--bin`. Veremos la sintaxis `#[]` y `mod tests` en la sección “Uso de `super`
para acceder a un módulo principal“ más adelante en este capítulo, pero por
ahora, deje este código en la parte inferior de *src/lib.rs *.

Como no tenemos un archivo *src/main.rs*, Cargo no puede ejecutar nada con el
comando `cargo run`. Por lo tanto, usaremos el comando `cargo build` para
compilar el código de nuestro cajón de biblioteca.

Veremos diferentes opciones para organizar el código de su biblioteca que
serán adecuadas en una variedad de situaciones, dependiendo de la intención
del código.

### Definiciones de módulos

Para nuestra biblioteca de redes `communicator`, primero definiremos un módulo
llamado `network` que contiene la definición de una función llamada `connect`.
Cada definición de módulo en Rust comienza con la palabra clave `mod`. Agregue
este código al principio del archivo *src/lib.rs*, encima del código de
prueba:

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}
```

Después de la palabra clave `mod`, colocamos el nombre del módulo,`network`,
y luego un bloque de código entre llaves. Todo dentro de este bloque está
dentro de la `red` del espacio de nombres. En este caso, tenemos una sola
función, `connect`. Si quisiéramos llamar a esta función desde el código que
está fuera del módulo `network`, necesitaríamos especificar el módulo y usar
la sintaxis del espacio de nombres `::` como así: `network::connect()`.

También podemos tener varios módulos, uno al lado del otro, en el mismo
archivo *src/lib.rs*. Por ejemplo, para tener también un módulo `cliente`
que tiene una función llamada` connect`, podemos agregarlo como se muestra en
el Listado 7-1.

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }
}

mod client {
    fn connect() {
    }
}
```

<span class="caption">Listing 7-1: El módulo `network` y el módulo `client`
definidos uno al lado del otro en *src/lib.rs*</span>

Ahora tenemos una función `network :: connect` y una función
`client::connect`. Estos pueden tener una funcionalidad completamente
diferente, y los nombres de las funciones no entran en conflicto entre sí
porque están en módulos diferentes.

En este caso, debido a que estamos creando una biblioteca, el archivo que
sirve como punto de entrada para construir nuestra biblioteca es *src/lib.rs*.
Sin embargo, con respecto a la creación de módulos, no hay nada especial sobre
*src/lib.rs*. También podríamos crear módulos en *src/main.rs* para
una *binary crate* de la misma manera que estamos creando módulos en
*src/lib.rs* para la *library crate*. De hecho, podemos poner módulos dentro
de los módulos, que pueden ser útiles a medida que crecen sus módulos para
mantener la funcionalidad relacionada organizada y separar por separado la
funcionalidad. La forma en que elija organizar su código depende de cómo
piense acerca de la relación entre las partes de su código. Por ejemplo, el
código `client` y su función `connect` podrían tener más sentido para los
usuarios de nuestra biblioteca si estuvieran dentro del espacio de nombres
`network`, como en el Listado 7-2.

<span class="filename">Filename: src/lib.rs</span>

```rust
mod network {
    fn connect() {
    }

    mod client {
        fn connect() {
        }
    }
}
```

<span class="caption">Listing 7-2: Mover el módulo `client` dentro del módulo
`network`</span>

En su archivo *src/lib.rs*, reemplace las definiciones existentes
`mod network` y `mod client` con las del Listado 7-2, que tienen el módulo
`client` como un módulo interno de `network`. Las funciones
`network:: connect` y `network::client::connect` se denominan `connect`, pero
no entran en conflicto entre ellas porque se encuentran en espacios de
nombres diferentes.

De esta manera, los módulos forman una jerarquía. Los contenidos de
*src/lib.rs* están en el nivel superior, y los submódulos están en niveles
más bajos. Aquí se muestra cómo se ve la organización de nuestro ejemplo en
el listado 7-1 cuando se considera una jerarquía:

```text
communicator
 ├── network
 └── client
```

Y aquí está la jerarquía correspondiente al ejemplo en el Listado 7-2:

```text
communicator
 └── network
     └── client
```

La jerarquía muestra que en el listado 7-2, `client` es un elemento
secundario del módulo `network` en lugar de un *sibling*. Los proyectos más
complicados pueden tener muchos módulos, y deberán estar organizados
lógicamente para que pueda seguirlos. Lo que “lógicamente” significa en su
proyecto depende de usted y depende de cómo usted y los usuarios de su
biblioteca piensen sobre el dominio de su proyecto. Use las técnicas que se
muestran aquí para crear módulos lado a lado y módulos anidados en la
estructura que desee.

### Mover módulos a otros archivos

Los módulos forman una estructura jerárquica, muy similar a otra estructura
en la informática a la que estás acostumbrado: sistemas de archivos. Podemos
usar el sistema de módulos de Rust junto con varios archivos para dividir los
proyectos de Rust, de modo que no todo esté en *src/lib.rs* o *src/main.rs*.
Para este ejemplo, comencemos con el código en el Listado 7-3.

<span class="filename">Filename: src/lib.rs</span>

```rust
mod client {
    fn connect() {
    }
}

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

<span class="caption">Listing 7-3: Tres módulos, `client`,`network`, y
`network::server`, todos definidos en *src/lib.rs *</span>

El archivo *src/lib.rs* tiene esta jerarquía de módulo:

```text
communicator
 ├── client
 └── network
     └── server
```

Si estos módulos tuvieran muchas funciones, y esas funciones fueran cada vez
más largas, sería difícil desplazarse por este archivo para encontrar el
código con el que queríamos trabajar. Debido a que las funciones están
anidadas dentro de uno o más bloques `mod`, las líneas de código dentro de
las funciones también comenzarán a ser largas. Estas serían buenas razones
para separar los módulos `client`, `network` y `server` de *src/lib.rs* y
colocarlos en sus propios archivos.

Primero, reemplacemos el código del módulo `cliente` con solo la declaración
del módulo `cliente` para que *src/lib.rs* se vea como el código que se
muestra en el Listado 7-4.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
mod client;

mod network {
    fn connect() {
    }

    mod server {
        fn connect() {
        }
    }
}
```

<span class="caption">Listing 7-4: Extrayendo el contenido del módulo
`client` pero dejando la declaración en *src/lib.rs*</span>

Todavía estamos *declarando* el módulo `cliente` aquí, pero al reemplazar el
bloque con un punto y coma, le estamos diciendo a Rust que busque en otra
ubicación el código definido dentro del alcance del módulo `cliente`. En
otras palabras, la línea `mod client;` significa esto:

```rust,ignore
mod client {
    // contents of client.rs
}
```

Ahora necesitamos crear el archivo externo con ese nombre de módulo. Cree un
archivo *client.rs* en su directorio *src/* y ábralo. Luego ingrese lo
siguiente, que es la función `connect` en el módulo `client` que eliminamos
en el paso anterior:

<span class="filename">Filename: src/client.rs</span>

```rust
fn connect() {
}
```

Tenga en cuenta que no necesitamos una declaración `mod` en este archivo
porque ya hemos declarado el módulo `client` con `mod` en *src/lib.rs*. Este
archivo solo proporciona los *contenidos* del módulo `cliente`. Si colocamos
un `mod client` aquí, le daremos al módulo `client` su propio submódulo
llamado `client`.

Rust solo sabe buscar *src/lib.rs* de manera predeterminada. Si queremos
agregar más archivos a nuestro proyecto, debemos decirle a Rust en
*src/lib.rs* que busque en otros archivos; esta es la razón por la cual
`mod client` necesita definirse en *src/lib.rs* y no puede definirse en
*src/client.rs*.

Ahora el proyecto debe compilarse con éxito, aunque recibirá algunas
advertencias. Recuerde utilizar `cargo build` en lugar de `cargo run` porque
tenemos una *library crate* en lugar de una *binary crate*:

```text
$ cargo build
   Compiling communicator v0.1.0 (file:///projects/communicator)
warning: function is never used: `connect`
 --> src/client.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/lib.rs:4:5
  |
4 | /     fn connect() {
5 | |     }
  | |_____^

warning: function is never used: `connect`
 --> src/lib.rs:8:9
  |
8 | /         fn connect() {
9 | |         }
  | |_________^
```

Estas advertencias nos dicen que tenemos funciones que nunca se usan. No te
preocupes por estas advertencias por el momento; los abordaremos más adelante
en este capítulo en la sección “Controlando la visibilidad con `pub`“. La
buena noticia es que solo son advertencias; nuestro proyecto construido con
éxito!

Luego, extraigamos el módulo `network` en su propio archivo usando el mismo
patrón. En *src/lib.rs*, elimine el cuerpo del módulo `network` y agregue un
punto y coma a la declaración, así:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
mod client;

mod network;
```

Luego, cree un nuevo archivo *src/network.rs* e ingrese lo siguiente:

<span class="filename">Filename: src/network.rs</span>

```rust
fn connect() {
}

mod server {
    fn connect() {
    }
}
```

Tenga en cuenta que todavía tenemos una declaración `mod` dentro de este
archivo de módulo; esto es porque todavía queremos que `server` sea un
submódulo de `network`.

Ejecute `cargo build` de nuevo. ¡Éxito! Tenemos un módulo más para extraer:
`server`. Debido a que es un submódulo, es decir, un módulo dentro de un
módulo, nuestra táctica actual de extraer un módulo en un archivo con el
nombre de ese módulo no funcionará. Intentaremos de todos modos para que
pueda ver el error. En primer lugar, cambie *src/network.rs* para que tenga
`mod server;` en lugar del contenido del módulo `server`:

<span class="filename">Filename: src/network.rs</span>

```rust,ignore
fn connect() {
}

mod server;
```

Luego cree un archivo *src/server.rs* e ingresa el contenido del módulo
`server` que extrajimos:

<span class="filename">Filename: src/server.rs</span>

```rust
fn connect() {
}
```

Cuando intentemos ejecutar `cargo build`, obtendremos el error que se muestra
en el Listado 7-5.

```text
$ cargo build
   Compiling communicator v0.1.0 (file:///projects/communicator)
error: cannot declare a new module at this location
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
  |
note: maybe move this module `src/network.rs` to its own directory via `src/network/mod.rs`
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
note: ... or maybe `use` the module `server` instead of possibly redeclaring it
 --> src/network.rs:4:5
  |
4 | mod server;
  |     ^^^^^^
```

<span class="caption">Listing 7-5: Error al intentar extraer el submódulo
`server` en *src/server.rs *</span>

El error dice que `no se puede declarar un nuevo módulo en esta ubicación` y
apunta a la línea `mod server;`en *src/network.rs*. Entonces
*src/network.rs* es diferente de *src/lib.rs* de alguna manera: sigue leyendo
para entender por qué.

La nota en el medio del Listado 7-5 es realmente muy útil porque señala algo
que todavía no hemos hablado:

```text
note: maybe move this module `network` to its own directory via
`network/mod.rs`
```

En lugar de continuar siguiendo el mismo patrón de denominación de archivos
que usamos anteriormente, podemos hacer lo que sugiere la nota:

1. Cree un nuevo *directorio* llamado *red*, el nombre del módulo principal.
2. Mueva el archivo *src/network.rs* al nuevo directorio *network* y cámbiele
    el nombre *src/network/mod.rs*.
3. Mueva el archivo de submódulo *src/server.rs* al directorio *network*.

Aquí hay comandos para llevar a cabo estos pasos:

```text
$ mkdir src/network
$ mv src/network.rs src/network/mod.rs
$ mv src/server.rs src/network
```

Ahora, cuando intentemos ejecutar `cargo build`, la compilación funcionará
(aunque todavía tendremos advertencias). Nuestro diseño de módulo todavía se
ve exactamente igual que cuando teníamos todo el código en *src/lib.rs* en el
Listado 7-3:

```text
communicator
 ├── client
 └── network
     └── server
```

El diseño de archivo correspondiente ahora se ve así:

```text
└── src
    ├── client.rs
    ├── lib.rs
    └── network
        ├── mod.rs
        └── server.rs
```

Entonces, cuando quisimos extraer el módulo `network::server`, ¿por qué
también teníamos que cambiar el archivo *src/network.rs* al archivo
*src/networ/mod.rs* y poner el código para `network::server` en el directorio
*network* en *src/network/server.rs*? ¿Por qué no podríamos simplemente
extraer el módulo `network::server` en *src/server.rs*? La razón es que Rust
no podría reconocer que se suponía que `server` era un submódulo de `network`
si el archivo *server.rs* estaba en el directorio *src*. Para aclarar el
comportamiento de Rust aquí, consideremos un ejemplo diferente con la
siguiente jerarquía de módulos, donde todas las definiciones están en
*src/lib.rs*:

```text
communicator
 ├── client
 └── network
     └── client
```

En este ejemplo, tenemos tres módulos de nuevo: `client`,`network`, y
`network::client`. Siguiendo los mismos pasos que hicimos anteriormente para
extraer módulos en archivos, creamos *src/client.rs* para el módulo `client`.
Para el módulo `network`, creamos *src/network.rs*. Pero no podríamos extraer
el módulo `network::client` en un archivo *src/client.rs* porque eso ya
existe para el módulo `client` de nivel superior. Si pudiéramos poner el
código para los módulos *ambos* `client` y `network::client` en el archivo
*src/client.rs*, Rust no tendría forma de saber si el código era para
`client` o para `network::client`.

Por lo tanto, para extraer un archivo para el submódulo `network::client` del
módulo` network`, necesitamos crear un directorio para el módulo `network` en
lugar de un archivo *src/network.rs*. El código que está en el módulo
`network` luego entra en el archivo *src/network/mod.rs*, y el submódulo
`network::client` puede tener su propio archivo *src/networ/client.rs*.
Ahora, el nivel superior *src/client.rs* es inequívocamente el código que
pertenece al módulo `client`.

### Reglas de los sistemas de archivos del módulo

Vamos a resumir las reglas de los módulos con respecto a los archivos:

* Si un módulo llamado `foo` no tiene submódulos, debe poner las
   declaraciones de `foo` en un archivo llamado *foo.rs*.
* Si un módulo llamado `foo` tiene submódulos, debe poner las declaraciones
   para `foo` en un archivo llamado *foo/mod.rs*.

Estas reglas se aplican recursivamente, por lo que si un módulo llamado `foo`
tiene un submódulo llamado `bar` y `bar` no tiene submódulos, debe tener los
siguientes archivos en su directorio *src*:

```text
└── foo
    ├── bar.rs (contains the declarations in `foo::bar`)
    └── mod.rs (contains the declarations in `foo`, including `mod bar`)
```

Los módulos deben declararse en el archivo de su módulo padre con la palabra
clave `mod`.

A continuación, hablaremos de la palabra clave `pub` y eliminaremos esas
advertencias.