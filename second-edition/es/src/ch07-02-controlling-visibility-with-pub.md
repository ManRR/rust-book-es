## Controlando la visibilidad con `pub`

Resolvimos los mensajes de error que se muestran en el Listado 7-5 moviendo
el código `network` y `network::server` en los archivos *src/network/mod.rs*
y *src/network/server.rs*, respectivamente. En ese momento, `cargo build` fue
capaz de construir nuestro proyecto, pero todavía recibimos mensajes de
advertencia que dicen que las funciones `client::connect`, `network::connect`,
y `network::server::connect` son no ser usado.

Entonces, ¿por qué recibimos estas advertencias? Después de todo, estamos
construyendo una biblioteca con funciones que están destinadas a ser
utilizadas por nuestros *usuarios*, no necesariamente por nosotros dentro de
nuestro propio proyecto, por lo que no debería importar que estas funciones
`connect` no se utilicen. El objetivo de crearlos es que serán utilizados por
otro proyecto, no el nuestro.

Para comprender por qué este programa invoca estas advertencias, intentemos
usar la biblioteca `communicator` de otro proyecto, llamándolo externamente.
Para hacer eso, crearemos una *binary crate* en el mismo directorio que
nuestra *library crate* haciendo un archivo *src/main.rs* que contenga este
código:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate communicator;

fn main() {
    communicator::client::connect();
}
```

Usamos el comando `extern crate` para llevar el *library crate*
`communicator` dentro del alcance. Nuestro paquete ahora contiene *dos*
*crates*. Cargo trata *src/main.rs* como el archivo raíz de una
*binary crate*, que está separada de la *library  crate* existente cuyo
archivo raíz es *src/lib.rs*. Este patrón es bastante común para los
proyectos ejecutables: la mayoría de las funcionalidades se encuentran en una
*library crate*, y la *binary crate* usa la *library crate*. Como resultado,
otros programas también pueden usar la *library crate*, y es una buena
separación de preocupaciones.

Desde el punto de vista de una *crate* fuera de la biblioteca de
`comunicador`, todos los módulos que hemos estado creando están dentro de un
módulo que tiene el mismo nombre que la *crate*, `comunicador`. Llamamos al
módulo de nivel superior de una *crate* el *módulo raíz*.

También tenga en cuenta que incluso si estamos usando una *crate* externa
dentro de un submódulo de nuestro proyecto, la `extern crate` debe ir en
nuestro módulo raíz (por lo tanto, en *src/main.rs* o *src/ ib.rs*). Luego,
en nuestros submódulos, podemos referirnos a elementos de *crates* externas
como si los elementos fueran módulos de nivel superior.

En este momento, nuestra *binary crate* simplemente llama a la función
`connect` de nuestra biblioteca desde el módulo `client`. Sin embargo,
invocando `cargo build` ahora nos dará un error después de las advertencias:

```text
error[E0603]: module `client` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

¡Ah, ja! Este error nos dice que el módulo `client` es privado, que es el
quid de las advertencias. También es la primera vez que nos encontramos con
los conceptos de *public* y *private* en el contexto de Rust. El estado
predeterminado de todo el código en Rust es privado: nadie más tiene permiso
para usar el código. Si no usa una función privada dentro de su programa,
porque su programa es el único código permitido para usar esa función, Rust
le advertirá que la función no se utilizó.

Después de especificar que una función como `client::connect` es pública, no
solo se permitirá su llamada a esa función desde su *binary crate*, sino que
también desaparecerá la advertencia de que la función no se utiliza. Marcar
una función como pública le permite a Rust saber que la función será
utilizada por código fuera de su programa. Rust considera el uso externo
teórico que ahora es posible como la función “en uso”. Por lo tanto, cuando
una función se marca como pública, Rust no requerirá que se use en su
programa y dejará de advertir que la función no se usa.

### Haciendo una función pública

Para decirle a Rust que haga pública una función, agregamos la palabra clave
`pub` al inicio de la declaración. Nos centraremos en corregir la advertencia
que indica que `client::connect` no se usó por ahora, así como que el módulo
`cliente` es un error privado de nuestra *binary crate*. Modifique
*src/lib.rs* para hacer que el módulo `client` sea público, así:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub mod client;

mod network;
```

La palabra clave `pub` se coloca justo antes de `mod`. Intentemos construir
de nuevo:

```text
error[E0603]: function `connect` is private
 --> src/main.rs:4:5
  |
4 |     communicator::client::connect();
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

¡Hurra! Tenemos un error diferente! Sí, los diferentes mensajes de error son
motivo de celebración. El nuevo error muestra `` function `connect` is private ``, así que editemos *src/client.rs* para hacer que `client::connect`
sea público:

<span class="filename">Filename: src/client.rs</span>

```rust
pub fn connect() {
}
```

Ahora ejecute `cargo build` de nuevo:

```text
warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

¡El código compilado y la advertencia de que `client::connect` no se está
utilizando ha desaparecido!

Las advertencias de código no utilizadas no siempre indican que un elemento
de su código debe hacerse público: si *no* desea que estas funciones formen
parte de su API pública, las advertencias de código no utilizadas podrían
alertarlo para que deje de codificarlo. necesita que pueda eliminar de forma
segura. También podrían alertarlo sobre un error si hubiera eliminado
accidentalmente todos los lugares de su biblioteca donde se llama a esta
función.

Pero en este caso, *queremos* que las otras dos funciones formen parte de la
API pública de nuestro *crate*, así que también vamos a marcarlas como `pub`
para eliminar las advertencias restantes. Modifique *src/network mod.rs* para
que se vea como el siguiente:

<span class="filename">Filename: src/network/mod.rs</span>

```rust,ignore
pub fn connect() {
}

mod server;
```

Luego compile el código:

```text
warning: function is never used: `connect`
 --> src/network/mod.rs:1:1
  |
1 | / pub fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default

warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
```

Hmmm, todavía recibimos una advertencia de función no utilizada, a pesar de
que `network::connect` está establecido en `pub`. La razón es que la función
es pública dentro del módulo, pero el módulo `network` en el que reside la
función no es público. Esta vez estamos trabajando desde el interior de la
biblioteca, mientras que con `client::connect` trabajamos desde el exterior.
Necesitamos cambiar *src/lib.rs* para hacer `network` público también, como:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub mod client;

pub mod network;
```

Ahora cuando compilamos, esa advertencia se ha ido:

```text
warning: function is never used: `connect`
 --> src/network/server.rs:1:1
  |
1 | / fn connect() {
2 | | }
  | |_^
  |
  = note: #[warn(dead_code)] on by default
```

Solo queda una advertencia: ¡intente arreglarlo usted solo!

### Reglas de privacidad

En general, estas son las reglas para la visibilidad del elemento:

- Si un elemento es público, se puede acceder a él a través de cualquiera de
   sus módulos principales.
- Si un elemento es privado, solo se puede acceder mediante su módulo padre
   inmediato y cualquiera de los módulos hijos del padre.

### Ejemplos de privacidad

Veamos algunos ejemplos más de privacidad para obtener algo de práctica. Cree
un nuevo proyecto de biblioteca e ingrese el código en el Listado 7-6 en el
*src/lib.rs* de su nuevo proyecto.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
mod outermost {
    pub fn middle_function() {}

    fn middle_secret_function() {}

    mod inside {
        pub fn inner_function() {}

        fn secret_function() {}
    }
}

fn try_me() {
    outermost::middle_function();
    outermost::middle_secret_function();
    outermost::inside::inner_function();
    outermost::inside::secret_function();
}
```

<span class="caption">Listing 7-6: Ejemplos de funciones privadas y públicas,
algunas de las cuales son incorrectas</span>

Antes de intentar compilar este código, adivine qué líneas de la función
`try_me` tendrán errores. Luego, intente compilar el código para ver si tenía
razón, ¡y siga leyendo para ver los errores!

#### Mirando los errores

La función `try_me` está en el módulo raíz de nuestro proyecto. El módulo
llamado `outermost` es privado, pero la segunda regla de privacidad establece
que la función `try_me` puede acceder al módulo `outermost` porque
`outermost` está en el módulo actual (root), al igual que `try_me`.

La llamada a `outermost::middle_function` funcionará porque `middle_function`
es pública y `try_me` está accediendo a `middle_function` a través de su
módulo padre `outermost`. Ya determinamos que este módulo es accesible.

La llamada a `outer::middle_secret_function` causará un error de compilación.
Como `middle_secret_function` es privado, se aplica la segunda regla. El
módulo raíz no es ni el módulo actual de `middle_secret_function`
(`outermost` is), ni es un módulo secundario del módulo actual de
`middle_secret_function`.

El módulo llamado `inside` es privado y no tiene módulos secundarios, por lo
que solo se puede acceder mediante su módulo actual `outermost`. Eso
significa que la función `try_me` no puede llamar
`outermost::inside::inner_function` o `outermost::inside::secret_function`.

#### Reparar los errores

Aquí hay algunas sugerencias para cambiar el código en un intento de corregir
los errores. Adivine si corregirá los errores antes de probar cada uno. Luego
compila el código para ver si tienes razón o no, usando las reglas de
privacidad para entender por qué. ¡Siéntase libre de diseñar más experimentos
y probarlos!

* ¿Qué pasa si el módulo `inside` fuera público?
* ¿Qué pasa si `outermost` fuera público y `inside` fuera privado?
* ¿Qué pasaría si, en el cuerpo de `inner_function`, llamaras a
   `::outermost ::middle_secret_function()`? (Los dos dos puntos al principio
   significan que queremos referirnos a los módulos a partir del módulo raíz).

Luego, hablemos acerca de traer elementos al alcance con la palabra clave
`use`.