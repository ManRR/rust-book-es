## Apéndice D: Macros

Hemos utilizado macros como `println!` a lo largo de este libro, pero no
hemos explorado completamente qué es una macro y cómo funciona. Este apéndice
explica las macros de la siguiente manera:

* Qué son las macros y cómo se diferencian de las funciones
* Cómo definir una macro declarativa para hacer metaprogramación
* Cómo definir una macro de procedimiento para crear *traits* `derive`
 personalizados

Estamos cubriendo los detalles de las macros en un apéndice porque todavía
están evolucionando en Rust. Las macros han cambiado y, en un futuro próximo,
cambiarán a un ritmo más rápido que el resto del lenguaje y la biblioteca
estándar desde Rust 1.0, por lo que es más probable que esta sección quede
desactualizada que el resto del libro. Debido a las garantías de estabilidad
de Rust, el código que se muestra aquí seguirá funcionando con versiones
futuras, pero puede haber capacidades adicionales o formas más sencillas de
escribir macros que no estaban disponibles en el momento de esta publicación.
Tenga esto en cuenta cuando intente implementar algo de este apéndice.

### La diferencia entre macros y funciones

Fundamentalmente, las macros son una forma de escribir código que escribe
otro código, que se conoce como *metaprogramación*. En el Apéndice C,
discutimos el atributo `derive`, que genera una implementación de varios
rasgos para usted. También utilizamos las macros `println!` Y `vec!`. En todo
el libro. Todas estas macros *se expanden* para producir más código que el
código que ha escrito manualmente.

La metaprogramación es útil para reducir la cantidad de código que tiene que
escribir y mantener, que también es uno de los roles de las funciones. Sin
embargo, las macros tienen algunos poderes adicionales que las funciones no
tienen.

Una firma de función debe declarar el número y tipo de parámetros que tiene
la función. Las macros, por otro lado, pueden tomar un número variable de
parámetros: podemos llamar `println!("Hello")` con un argumento o
`println!("hello {}", name)` con dos argumentos. Además, las macros se
expanden antes de que el compilador interprete el significado del código, por
lo que una macro puede, por ejemplo, implementar un *trait* en un tipo dado.
Una función no puede, porque se llama en el tiempo de ejecución y un *trait*
debe implementarse en tiempo de compilación.

La desventaja de implementar una macro en lugar de una función es que las
definiciones de macro son más complejas que las definiciones de función
porque estás escribiendo el código de Rust que escribe el código de Rust.
Debido a esta indirección, las definiciones de macro generalmente son más
difíciles de leer, comprender y mantener que las definiciones de funciones.

Otra diferencia entre las macros y las funciones es que las definiciones de
macro no son espacios de nombres dentro de los módulos, como son las
definiciones de funciones. Para evitar conflictos de nombres inesperados al
usar *crate* externas, tiene que incluir explícitamente las macros en el
alcance de su proyecto al mismo tiempo que lleva el *crate* externo dentro
del alcance, usando la anotación `#[macro_use]`. El siguiente ejemplo traerá
todas las macros definidas en el *crate* `serde` dentro del alcance del
*crate* actual:

```rust,ignore
#[macro_use]
extern crate serde;
```

Si `extern crate` fuera capaz de traer macros al alcance de forma
predeterminada sin esta anotación explícita, no se le permitiría usar dos
*crates* que le permitieron definir macros con el mismo nombre. En la
práctica, este conflicto no ocurre a menudo, pero cuantas más *crates* uses,
más posibilidades habrá.

Existe una última diferencia importante entre las macros y las funciones:
debe definir o incluir macros en el alcance *antes* de que las llame en un
archivo, mientras que puede definir funciones en cualquier lugar y llamarlas
a cualquier lugar.

### Macros declarativas con `macro_rules!` para la metaprogramación general

La forma más utilizada de macros en Rust son *macros declarativas*. A veces
también se les conoce como *macros por ejemplo*, *`macro_rules!` macros*, o
simplemente simples *macros*. En esencia, las macros declarativas le permiten
escribir algo similar a una expresión de `match` de Rust. Como se discutió en
el Capítulo 6, las expresiones `match` son estructuras de control que toman
una expresión, comparan el valor resultante de la expresión con patrones y
luego ejecutan el código asociado con el patrón coincidente. Las macros
también comparan un valor con patrones que tienen código asociado a ellos; en
esta situación, el valor es el código fuente literal de Rust que se pasa a la
macro, los patrones se comparan con la estructura de ese código fuente, y el
código asociado con cada patrón es el código que reemplaza el código pasado a
la macro. Todo esto sucede durante la compilación.

Para definir una macro, utiliza la construcción `macro_rules!`. ¡Exploremos
cómo usar `macro_rules!`. Mirando cómo se define la macro `vec!`. El Capítulo
8 cubrió cómo podemos usar la macro `vec!`. Para crear un nuevo vector con
valores particulares. Por ejemplo, la siguiente macro crea un nuevo vector
con tres enteros dentro:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

También podríamos usar la macro `vec!` para hacer un vector de dos enteros o
un vector de cinco *string slices*. No podríamos usar una función para hacer
lo mismo porque no sabríamos la cantidad o el tipo de valores por adelantado.

Veamos una definición ligeramente simplificada de la macro `vec!` en el
Listado D-1.

```rust
#[macro_export]
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
```

<span class="caption">Listado D-1: Una versión simplificada de la definición
de macro `vec!`</span>

> Nota: La definición real de la macro `vec!` en la biblioteca estándar
> incluye un código para preasignar la cantidad correcta de memoria por
> adelantado. Ese código es una optimización que no incluimos aquí para
> simplificar el ejemplo.

La anotación `#[macro_export]` indica que esta macro debe estar disponible
siempre que se importe el *crate* en la que estamos definiendo la macro. Sin
esta anotación, incluso si alguien que dependa de este *crate* usa la
anotación `#[macro_use]`, la macro no se incluiría en el alcance.

Luego comenzamos la definición de macro con `macro_rules!` y el nombre de la
macro que estamos definiendo *sin* el signo de exclamación. El nombre, en
este caso `vec`, es seguido por llaves que denotan el cuerpo de la definición
de macro.

La estructura en el cuerpo `vec!` es similar a la estructura de una expresión
`match`. Aquí tenemos un brazo con el patrón `( $( $x:expr ),* )`, seguido de
`=>` y el bloque de código asociado con este patrón. Si el patrón coincide,
se emitirá el bloque de código asociado. Dado que este es el único patrón en
esta macro, solo hay una forma válida de concordar; cualquier otro será un
error. Las macros más complejas tendrán más de un brazo.

La sintaxis de patrón válida en las definiciones de macro es diferente de la
sintaxis de patrón cubierta en el Capítulo 18 porque los patrones de macro se
hacen coincidir con la estructura de código de Rust en lugar de los valores.
Veamos qué significan las piezas del patrón del Listado D-1; para ver la
sintaxis completa del patrón de macro, vea [la referencia].

[la referencia]: ../../reference/macros.html

Primero, un conjunto de paréntesis abarca todo el patrón. Luego viene un
signo de dólar (`$`) seguido de un conjunto de paréntesis, que captura
valores que coinciden con el patrón entre paréntesis para usar en el código
de reemplazo. Dentro de `$()` es `$x:expr`, que coincide con cualquier
expresión de Rust y le da a la expresión el nombre `$ x`.

La coma que sigue a `$()` indica que un carácter literal de separador de coma
podría aparecer opcionalmente después del código que coincide con el código
capturado en `$()`. El `*` que sigue a la coma especifica que el patrón
coincide con cero o más de lo que precede al `*`.

Cuando llamamos a esta macro con `vec![1, 2, 3];`, el patrón `$ x` coincide
tres veces con las tres expresiones `1`, `2` y `3`.

Ahora veamos el patrón en el cuerpo del código asociado a este brazo: el
código `temp_vec.push()` dentro de la parte `$()*` se genera para cada parte
que coincide con `$()` en el patrón , cero o más veces, según cuántas veces
coincida el patrón. El `$ x` se reemplaza con cada expresión coincidente.
Cuando llamamos a esta macro con `vec! [1, 2, 3];`, el código generado que
reemplaza esta llamada de macro será el siguiente:

```rust,ignore
let mut temp_vec = Vec::new();
temp_vec.push(1);
temp_vec.push(2);
temp_vec.push(3);
temp_vec
```

Hemos definido una macro que puede tomar cualquier cantidad de argumentos de
cualquier tipo y puede generar código para crear un vector que contenga los
elementos especificados.

Dado que la mayoría de los programadores de Rust *utilizarán* macros más que
*write* macros, no discutiremos `macro_rules!`. Para obtener más información
sobre cómo escribir macros, consulte la documentación en línea u otros
recursos, como [“The Little Book of Rust Macros”][tlborm].

[tlborm]: https://danielkeep.github.io/tlborm/book/index.html

### Macros de procedimiento para personalizar `derive`

La segunda forma de macros se llama *procedural macros* (*macros de
procedimiento*) porque se asemejan más a funciones (que son un tipo de
procedimiento). Las macros de procedimiento aceptan algún código de Rust como
entrada, operan en ese código y producen algún código de Rust como salida en
lugar de coincidir con patrones y reemplazar el código con otro código como
lo hacen las macros declarativas. En el momento de escribir estas líneas,
solo puede definir macros de procedimiento para permitir que sus *traits* se
implementen en un tipo especificando el nombre de *trait* en una anotación
`derivar`.

Crearemos un *crate* llamada `hello_macro` que define un *trait* llamado
`HelloMacro` con una función asociada llamada `hello_macro`. En lugar de
hacer que nuestros usuarios del *crate* implementen el *trait* `HelloMacro`
para cada uno de sus tipos, proporcionaremos una macro de procedimientos para
que los usuarios puedan anotar su tipo con `#[derive(HelloMacro)]` para
obtener una implementación predeterminada de `hello_macro` función. La
implementación predeterminada imprimirá `Hello, Macro! My name is TypeName!`
donde `TypeName` es el nombre del tipo en el que se ha definido este *trait*.
En otras palabras, escribiremos un *crate* que permita a otro programador
escribir código como el Listado D-2 usando nuestro *crate*.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate hello_macro;
#[macro_use]
extern crate hello_macro_derive;

use hello_macro::HelloMacro;

#[derive(HelloMacro)]
struct Pancakes;

fn main() {
    Pancakes::hello_macro();
}
```

<span class="caption">Listado D-2: El código que un usuario de nuestro *crate*
podrá escribir cuando utilice nuestra macro de procedimientos</span>

Este código imprimirá `Hello, Macro! My name is Pancakes!` cuando hayamos
terminado. El primer paso es crear un nueva *library crate*, como esta:

```text
$ cargo new hello_macro --lib
```

A continuación, definiremos el *trait* `HelloMacro` y su función asociada:

<span class="filename">Filename: src/lib.rs</span>

```rust
pub trait HelloMacro {
    fn hello_macro();
}
```

Tenemos un *trait* y su función. En este punto, nuestro usuario del *trait*
podría implementar el *trait* para lograr la funcionalidad deseada, así:

```rust,ignore
extern crate hello_macro;

use hello_macro::HelloMacro;

struct Pancakes;

impl HelloMacro for Pancakes {
    fn hello_macro() {
        println!("Hello, Macro! My name is Pancakes!");
    }
}

fn main() {
    Pancakes::hello_macro();
}
```

Sin embargo, tendrían que escribir el bloque de implementación para cada tipo
que quisieran usar con `hello_macro`; queremos evitar que tengan que hacer
este trabajo.

Además, todavía no podemos proporcionar una implementación predeterminada
para la función `hello_macro` que imprimirá el nombre del tipo en el que se
implementa el *trait*: Rust no tiene capacidades de reflexión, por lo que no
puede buscar el nombre del tipo en tiempo de ejecución necesitamos una macro
para generar código en tiempo de compilación.

El siguiente paso es definir la macro de procedimiento. En el momento de
escribir estas líneas, las macros de procedimiento deben estar en su propio
*crate*. Eventualmente, esta restricción podría ser levantada. La convención
para estructurar *crates* y macrofolios es la siguiente: para un *crate*
llamada `foo`, un *crate* macro de procedimientos derivada personalizada se
llama `foo_derive`. Comencemos un nuevo *crate* llamada `hello_macro_derive`
dentro de nuestro proyecto `hello_macro`:

```text
$ cargo new hello_macro_derive --lib
```

Nuestros dos *crates* están estrechamente relacionadas, por lo que creamos la
macro de procedimiento dentro del directorio de nuestro *crate*
`hello_macro`. Si cambiamos la definición de *trait* en `hello_macro`,
también tendremos que cambiar la implementación de la macro de procedimiento
en `hello_macro_derive`. Los dos *crates* tendrán que publicarse por separado
y los programadores que utilicen estos *crates* tendrán que agregar ambas
como dependencias y ponerlas a ambas en el alcance. Podríamos, en cambio, hacer que el *crates* `hello_macro` use `hello_macro_derive` como una
dependencia y reexportar el código macro de procedimiento. Pero la forma en
que hemos estructurado el proyecto hace posible que los programadores usen `hello_macro` incluso si no quieren la funcionalidad `derivar`.

Necesitamos declarar el *crate* `hello_macro_derive` como una macro *crate*
de procedimientos. También necesitaremos la funcionalidad de los *crates*
`syn` y `quote`, como verá en un momento, por lo que debemos agregarlas como
dependencias. Agregue lo siguiente al archivo *Cargo.toml* para
`hello_macro_derive`:

<span class="filename">Filename: hello_macro_derive/Cargo.toml</span>

```toml
[lib]
proc-macro = true

[dependencies]
syn = "0.11.11"
quote = "0.3.15"
```

Para comenzar a definir la macro de procedimiento, coloque el código en el
Listado D-3 en su archivo *src/lib.rs* para el *crate* `hello_macro_derive`.
Tenga en cuenta que este código no se compilará hasta que agreguemos una
definición para la función `impl_hello_macro`.

<span class="filename">Filename: hello_macro_derive/src/lib.rs</span>

```rust,ignore
extern crate proc_macro;
extern crate syn;
#[macro_use]
extern crate quote;

use proc_macro::TokenStream;

#[proc_macro_derive(HelloMacro)]
pub fn hello_macro_derive(input: TokenStream) -> TokenStream {
    // Construct a string representation of the type definition
    let s = input.to_string();

    // Parse the string representation
    let ast = syn::parse_derive_input(&s).unwrap();

    // Build the impl
    let gen = impl_hello_macro(&ast);

    // Return the generated impl
    gen.parse().unwrap()
}
```

<span class="caption">Listado D-3: Código que la mayoría de los
*macro crates* de procedimiento deberán tener para procesar el código de
Rust</span>

Observe la forma en que hemos dividido las funciones en D-3; esto será el
mismo para casi todos los *macro crate* de procedimiento que ve o crea,
porque hace que escribir un macro de procedimiento sea más conveniente. Lo
que elija hacer en el lugar donde se llama a la función `impl_hello_macro`
será diferente dependiendo del propósito de su macro de procedimiento.

Hemos introducido tres *crates* nuevos: `proc_macro`, [`syn`], y [`quote`].
El *crate* `proc_macro` viene con Rust, por lo que no fue necesario agregarlo a las dependencias en *Cargo.toml*. El *crate* `proc_macro` nos permite
convertir el código Rust en un *string* que contiene ese código Rust. El
`syn` crate analiza el código de Rust de un *string* en una estructura de
datos en la que podemos realizar operaciones. El *crate* `quote` toma las
estructuras de datos `syn` y las convierte nuevamente en código Rust. Estos
*crates* hacen que sea mucho más simple analizar cualquier tipo de código
Rust que podamos querer manejar: escribir un analizador completo para el
código Rust no es tarea fácil.

[`syn`]: https://crates.io/crates/syn
[`quote`]: https://crates.io/crates/quote

Se llamará a la función `hello_macro_derive` cuando un usuario de nuestra
biblioteca especifique `#[derive(HelloMacro)]` en un tipo. La razón es que
hemos anotado la función `hello_macro_derive` aquí con `proc_macro_derive` y
hemos especificado el nombre, `HelloMacro`, que coincide con nuestro nombre
de *trait*; esa es la convención que siguen la mayoría de las macros de
procedimiento.

Esta función primero convierte la `input` de un `TokenStream` en un `String`
llamando a `to_string`. Este `String` es una representación de *string* del
código Rust para el cual derivamos `HelloMacro`. En el ejemplo del Listado
D-2, `s` tendrá el valor `String` `struct Pancakes;` porque ese es el código
Rust al que agregamos la anotación `#[derive(HelloMacro)]`.

> Nota: En el momento de escribir esto, solo puedes convertir un
> `TokenStream` en un *string*. Una API más rica existirá en el futuro.

Ahora tenemos que analizar el código de Rust `String` en una estructura de
datos que luego podemos interpretar y realizar operaciones. Aquí es donde
`syn` entra en juego. La función `parse_derive_input` en `syn` toma `String`
y devuelve una estructura `DeriveInput` que representa el código Rust
analizado. El siguiente código muestra las partes relevantes de la estructura
`DeriveInput` que obtenemos al analizar el *string* `struct Pancakes;`:

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident(
        "Pancakes"
    ),
    body: Struct(
        Unit
    )
}
```

Los campos de esta estructura muestran que el código de Rust que hemos
analizado es una estructura de unidad con el `ident` (identificador, que
significa el nombre) de `Pancakes`. Hay más campos en esta estructura para
describir todo tipo de código Rust; revise la
[`syn` documentation for `DeriveInput`][syn-docs] para más información.

[syn-docs]: https://docs.rs/syn/0.11.11/syn/struct.DeriveInput.html

En este punto, no hemos definido la función `impl_hello_macro`, que es donde
construiremos el nuevo código Rust que queremos incluir. Pero antes de
hacerlo, tenga en cuenta que la última parte de esta función
`hello_macro_derive` usa la función `parse` del *crate* `quote` para convertir
la salida de la función `impl_hello_macro` de nuevo en `TokenStream`. El
`TokenStream` devuelto se agrega al código que escriben nuestros usuarios de
*crates*, por lo que cuando compilan su *crate*, obtendrán la funcionalidad
adicional que proporcionamos.

Es posible que haya notado que estamos llamando `unwrap` al pánico si las
llamadas a las funciones `parse_derive_input` o `analizar` fracasan aquí.
Pánico en los errores es necesario en el código de macro de procedimiento
porque las funciones `proc_macro_derive` deben devolver `TokenStream` en
lugar de `Result` para ajustarse a la API de macro de procedimiento. Elegimos
simplificar este ejemplo usando `unwrap`; en el código de producción, debe
proporcionar mensajes de error más específicos sobre lo que salió mal usando
`panic!` o `expect`.

Ahora que tenemos el código para convertir el código anotado de Rust de un
`TokenStream` en una instancia `String` y `DeriveInput`, generemos el código
que implementa el *trait* `HelloMacro` en el tipo anotado:

<span class="filename">Filename: hello_macro_derive/src/lib.rs</span>

```rust,ignore
fn impl_hello_macro(ast: &syn::DeriveInput) -> quote::Tokens {
    let name = &ast.ident;
    quote! {
        impl HelloMacro for #name {
            fn hello_macro() {
                println!("Hello, Macro! My name is {}", stringify!(#name));
            }
        }
    }
}
```

Obtenemos una instancia de estructura `Ident` que contiene el nombre
(identificador) del tipo anotado usando `ast.ident`. El código en el Listado
D-2 especifica que el `name` será `Ident("Pancakes")`.

La macro `quote!` nos permite escribir el código Rust que queremos devolver y
convertirlo en `quote::Tokens`. Esta macro también proporciona algunas
mecánicas de plantillas muy interesantes; podemos escribir `#name`, y
`quote!` lo reemplazará con el valor en la variable llamada `name`. Incluso
puede hacer una repetición similar a la forma en que funcionan las macros
normales. Consulte [the `quote` crate’s docs][quote-docs] para una
introducción completa.

[quote-docs]: https://docs.rs/quote

Queremos que nuestra macro de procedimientos genere una implementación de
nuestro *trait* `HelloMacro` para el tipo anotado por el usuario, que podemos
obtener usando `#name`.La implementación del *trait* tiene una función,
`hello_macro`, cuyo cuerpo contiene el funcionalidad que queremos
proporcionar: impresión `Hello, Macro! My name is` y luego el nombre del tipo
anotado.

La macro `stringify!` Utilizada aquí está integrada en Rust. Toma una
expresión Rust, como `1 + 2`, y en tiempo de compilación convierte la
expresión en un literal de *string*, como `"1 + 2"`. Esto es diferente de
`format!` o `println!`, que evalúa la expresión y luego convierte el
resultado en `String`. Existe la posibilidad de que la entrada `#name` sea
una expresión para imprimir literalmente, entonces usamos `stringify!`.
Usando `stringify!` también guarda una asignación convirtiendo `#name` a un
*string* literal en tiempo de compilación.

En este punto, `cargo build` debería completarse con éxito en ambos
`hello_macro` y `hello_macro_derive`. Vamos a conectar estos *crates* al
código en el Listado D-2 para ver la macro de procedimiento en acción! cree
un nuevo proyecto binario en su directorio *proyectos* utilizando
`cargo new --bin pancakes`. Necesitamos agregar `hello_macro` y
`hello_macro_derive` como dependencias en los `pancakes`
*crates* *Cargo.toml*. Si publica sus versiones de `hello_macro` y
`hello_macro_derive` a *https://crates.io/*, serían dependencias regulares;
si no, puede especificarlos como dependencias `path` de la siguiente manera:

```toml
[dependencies]
hello_macro = { path = "../hello_macro" }
hello_macro_derive = { path = "../hello_macro/hello_macro_derive" }
```

Coloque el código del Listado D-2 en *src/main.rs*, y ejecute `cargo run`:
debería imprimir `Hello, Macro! My name is Pancakes!` la implementación del
*trait* `HelloMacro` de la macro procedural se incluyó sin que el *crate*
`pancakes` necesitara implementarla; el `#[derive HelloMacro)]` agregó la
implementación del *traits*.

### El futuro de las macros

En el futuro, Rust ampliará las macros declarativas y de procedimiento. Rust
utilizará un mejor sistema de macros declarativas con la palabra clave
`macro` y agregará más tipos de macros de procedimientos para tareas más
potentes que simplemente `derive`. Estos sistemas aún están en desarrollo en
el momento de esta publicación; Consulte la documentación en línea de Rust
para obtener la información más reciente.
