## Errores recuperables con `Result`

La mayoría de los errores no son lo suficientemente graves como para requerir
que el programa se detenga por completo. A veces, cuando una función falla,
es por una razón que puede interpretar y responder fácilmente. Por ejemplo,
si intenta abrir un archivo y esa operación falla porque el archivo no existe
es posible que desee crear el archivo en lugar de finalizar el proceso.

Recuerde “[Manejo de la falla potencial con el tipo `Result`][handle_failure] <!-- ignore -->” en el Capítulo 2 que la enumeración `Result` se define como que tiene dos variantes, `Ok` y `Err`, como sigue:

[handle_failure]: ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-the-result-type

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

El `T` y `E` son parámetros de tipo genérico: discutiremos los genéricos con
más detalle en el Capítulo 10. Lo que necesita saber ahora es que `T`
representa el tipo del valor que se devolverá en un éxito caso dentro de la
variante `Ok`, y `E` representa el tipo de error que se devolverá en un caso
de falla dentro de la variante `Err`. Como `Result` tiene estos parámetros de
tipo genérico, podemos utilizar el tipo `Result` y las funciones que la
biblioteca estándar ha definido en él en muchas situaciones diferentes en las
que el valor correcto y el valor de error que queremos devolver pueden
diferir.

Llamemos a una función que devuelve un valor `Result` porque la función
podría fallar. En el Listado 9-3 tratamos de abrir un archivo.

<span class="filename">Filename: src/main.rs</span>

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");
}
```

<span class="caption">Listing 9-3: Apertura de un archivo</span>

¿Cómo sabemos que `File::open` devuelve un `Result`? ¡Podríamos mirar la
documentación estándar de la API de la biblioteca, o podríamos preguntarle al
compilador! Si le damos a `f` una anotación de tipo que sabemos que es *no* el tipo de retorno de la función y luego tratamos de compilar el código, el compilador nos dirá que los tipos no coinciden. El mensaje de error nos dirá
qué tipo de `f` *es*. ¡Vamos a intentarlo! Sabemos que el tipo de devolución
de `File::open` no es del tipo `u32`, así que cambiemos la declaración `let f` a esto:

```rust,ignore
let f: u32 = File::open("hello.txt");
```

Intentar compilar ahora nos da el siguiente resultado:

```text
error[E0308]: mismatched types
 --> src/main.rs:4:18
  |
4 |     let f: u32 = File::open("hello.txt");
  |                  ^^^^^^^^^^^^^^^^^^^^^^^ expected u32, found enum
`std::result::Result`
  |
  = note: expected type `u32`
             found type `std::result::Result<std::fs::File, std::io::Error>`
```

Esto nos dice que el tipo de retorno de la función `File::open` es
`Result <T, E>`. El parámetro genérico `T` se ha rellenado aquí con el tipo
de valor de éxito,`std::fs::File`, que es un manejador de archivo. El tipo de
`E` utilizado en el valor de error es `std::io::Error`.

Este tipo de devolución significa que la llamada a `File::open` puede tener
éxito y devolver un manejador de archivo que podemos leer o escribir en. La
llamada a la función también puede fallar: por ejemplo, es posible que el
archivo no exista o que no tengamos permiso para acceder al archivo. La
función `File::open` necesita tener una forma de decirnos si tuvo éxito o no,
y al mismo tiempo proporcionarnos el identificador del archivo o la
información del error. Esta información es exactamente lo que transmite el
enum de `Result`.

En el caso donde `File::open` tiene éxito, el valor en la variable `f` será
una instancia de `Ok` que contiene un manejador de archivo. En el caso donde
falla, el valor en `f` será una instancia de `Err` que contiene más
información sobre el tipo de error que ocurrió.

Necesitamos agregar al código en el Listado 9-3 para tomar diferentes
acciones dependiendo del valor `File: open` returns. El Listado 9-4 muestra
una forma de manejar el `Result` utilizando una herramienta básica, la
expresión `match` que analizamos en el Capítulo 6.

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("There was a problem opening the file: {:?}", error)
        },
    };
}
```

<span class="caption">Listing 9-4: Usar una expresión `match` para manejar las variantes `Result` que pueden ser devueltas</span>

Tenga en cuenta que, al igual que la enumeración `Option`, el enum `Result` y
sus variantes se han importado en el preludio, por lo que no es necesario que
especifique `Result::` antes de las variantes `Ok` y `Err` en los brazos del `match`.

Aquí le decimos a Rust que cuando el resultado es `Ok`, devuelve el valor
`file` interno de la variante `Ok`, y luego asignamos ese valor de manejo de
archivo a la variable `f`. Después del `match`, podemos usar el manejador del
archivo para leer o escribir.

El otro brazo del `match` maneja el caso donde obtenemos un valor `Err` de
`File::open`. En este ejemplo, hemos elegido llamar a la macro `panic!`. Si
no hay un archivo llamado *hello.txt* en nuestro directorio actual y
ejecutamos este código, veremos el siguiente resultado de la macro `panic!`:

```text
thread 'main' panicked at 'There was a problem opening the file: Error { repr:
Os { code: 2, message: "No such file or directory" } }', src/main.rs:9:12
```

Como de costumbre, este resultado nos dice exactamente qué salió mal.

### Coincidencia en diferentes errores

El código en el Listado 9-4 entrará en `panic!`. No importa por qué
`File::open` falló. Lo que queremos hacer en su lugar es tomar diferentes
acciones por diferentes motivos de falla: si `File::open` falló porque el
archivo no existe, queremos crear el archivo y devolver el manejador al nuevo
archivo. Si `File::open` falló por alguna otra razón, por ejemplo, porque no
teníamos permiso para abrir el archivo, aún queremos que el código se `panic!` de la misma manera que en el Listado 9-4 . Mire el Listado 9-5, que agrega otro brazo al `match`.

<span class="filename">Filename: src/main.rs</span>

<!-- ignore esta prueba porque de lo contrario crea hello.txt que causa
que otras pruebas fallen lol -->

```rust,ignore
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(ref error) if error.kind() == ErrorKind::NotFound => {
            match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => {
                    panic!(
                        "Tried to create file but there was a problem: {:?}",
                        e
                    )
                },
            }
        },
        Err(error) => {
            panic!(
                "There was a problem opening the file: {:?}",
                error
            )
        },
    };
}
```

<span class="caption">Listing 9-5: Manejando diferentes tipos de errores de
diferentes maneras</span>

El tipo del valor que `File::open` devuelve dentro de la variante `Err` es
`io::Error`, que es una estructura proporcionada por la biblioteca estándar.
Esta estructura tiene un método `kind` al que podemos llamar para obtener un
valor `io::ErrorKind`. La enumeración `io::ErrorKind` es proporcionado por la
biblioteca estándar y tiene variantes que representa los diferentes tipos de
errores que pueden resultar de un `io` operación. La variante que
queremos usar es `ErrorKind::NotFound`, que indica el archivo que intentamos abrir aún no existe.

La condición `if error.kind() == ErrorKind::NotFound` se llama
*match guard*: es una condición adicional en un brazo de `match` que refina
aún más el patrón *brazos*. Esta condición debe ser verdadera para que se
ejecute el código de ese brazo; de otra manera, la coincidencia de patrones se moverá para considerar el próximo brazo en el `match`. los
`ref` en el patrón es necesario para que `error` no se mueva a la condición
de guardia pero simplemente es referenciado por él. La razón por la que usas
`ref` para crear una referencia en un patrón en lugar de `&` se tratará en
detalle en el Capítulo 18. En resumen, en el contexto de un patrón, `&`
coincide con una referencia y le da su valor, pero `ref` coincide con un
valor y le da una referencia.

La condición que queremos verificar en el guarda *match* es si el valor devueltopor `error.kind()` es la variante `NotFound` de la enumeración
`ErrorKind`. Si esto es, intentamos crear el archivo con `File::create`.
Sin embargo, como `File:: reate` también podría fallar, también necesitamos
agregar una declaración de interna al `match`. Cuando el archivo no se puede abrir, se imprimirá un mensaje de error diferente. El último brazo del
`match` externa permanece igual, por lo que el programa entra en pánico por
cualquier error además de el error de archivo faltante.

### Atajos para pánico en caso de error: `unwrap` y` expect`

Usar `match` funciona bastante bien, pero puede ser un poco detallado y no
siempre comunica bien el intento. El tipo `Result <T, E>` tiene muchos
métodos auxiliares definidos para realizar diversas tareas. Uno de esos
métodos, llamado `unwrap`, es un método abreviado que se implementa como la
declaración `match` que escribimos en el Listado 9-4. Si el valor `Result` es
la variante `Ok`, `unwrap` devolverá el valor dentro del `Ok`. Si el `Result`
es la variante `Err`, `unwrap` llamará a la macro `panic!` para nosotros.
Aquí hay un ejemplo de `unwrap` en acción:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

Si ejecutamos este código sin un archivo *hello.txt*, veremos un mensaje de
error de la llamada `panic!` que el método `unwrap` hace:

```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: Error {
repr: Os { code: 2, message: "No such file or directory" } }',
src/libcore/result.rs:906:4
```

Otro método, `expect`, que es similar a `unwrap`, nos permite elegir el
mensaje de error `panic!`. Usar `expect` en lugar de `unwrap` y proporcionar
buenos mensajes de error puede transmitir su intención y facilitar el rastreo
del origen de un ataque de pánico. La sintaxis de `expect` se ve así:

<span class="filename">Filename: src/main.rs</span>

```rust,should_panic
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

Usamos `expect` de la misma manera que `unwrap`: para devolver el manejador
de archivo o llamar a la macro `panic!`. El mensaje de error utilizado por
`expect` en su llamada a `panic!` será el parámetro que pasamos a `expect`,
en lugar del mensaje `panic!` Predeterminado que `unwrap` usa. Esto es lo que
parece:

```text
thread 'main' panicked at 'Failed to open hello.txt: Error { repr: Os { code:
2, message: "No such file or directory" } }', src/libcore/result.rs:906:4
```

Como este mensaje de error comienza con el texto que especificamos, `Failed to open hello.txt`, será más fácil encontrar el origen del código en el que
se origina este mensaje de error. Si usamos `unwrap` en varios lugares, puede
llevar más tiempo averiguar exactamente qué `unwrap` está causando el pánico,ya que todas las llamadas `unwrap` que entran en pánico imprimen el mismo
mensaje.

### Propagación de errores

Cuando está escribiendo una función cuya implementación llama a algo que
puede fallar, en lugar de manejar el error dentro de esta función, puede
devolver el error al código de llamada para que pueda decidir qué hacer. Esto
se conoce como *propagar* el error y le da más control al código de llamada,donde podría haber más información o lógica que dicte cómo se debe manejar el
error que lo que tiene disponible en el contexto de su código.

Por ejemplo, el Listado 9-6 muestra una función que lee un nombre de usuario
de un archivo. Si el archivo no existe o no se puede leer, esta función
devolverá esos errores al código que llamó a esta función.

<span class="filename">Filename: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

<span class="caption">Listing 9-6: Una función que devuelve errores al código
de llamada con `match`</span>

Mire primero el tipo de retorno de la función:
`Resultado <String, io::Error>`. Esto significa que la función devuelve un
valor del tipo `Result <T, E>` donde el parámetro genérico `T` se ha
rellenado con el tipo concreto `String` y el tipo genérico `E` se ha
rellenado con el tipo concreto `io::Error`. Si esta función tiene éxito sin
ningún problema, el código que llama a esta función recibirá un valor `Ok` que contiene una `String`, el nombre de usuario que esta función lee del
archivo. Si esta función encuentra algún problema, el código que llama a esta
función recibirá un valor `Err` que contiene una instancia de `io::Error` que
contiene más información sobre cuáles fueron los problemas. Elegimos
`io::Error` como el tipo de retorno de esta función porque ese es el tipo del
valor de error devuelto por las dos operaciones que estamos llamando en el
cuerpo de esta función que puede fallar: la función `File::open` y el método
`read_to_string`.

El cuerpo de la función comienza llamando a la función `File::open`. Entonces
nosotros manejamos el valor `Result` devuelto con un `match` similar al
`match` en Listado 9-4, solo en lugar de llamar `panic!` En el caso de `Err`,
retornamos temprano de esta función y pasamos el valor de error de
`File::open` vuelve al código de llamada como el valor de error de esta
función. Si `File::open` tiene éxito, almacenamos el archivo maneja en la
variable `f` y continúa.

Luego creamos un nuevo `String` en la variable `s` y llamamos `read_to_string`
método en el manejador de archivo en `f` para leer el contenido del archivo
en`s`. El método `read_to_string` también devuelve un `Result` porque puede
fallar, incluso aunque `File::open` tenga éxito. Entonces, necesitamos otro
`match` para manejar eso `Result`: si `read_to_string` tiene éxito, entonces nuestra función ha tenido éxito, y nosotros devuelve el nombre de usuario del
archivo que ahora está en `s` envuelto en un `Ok`. Si `read_to_string` falla,
devolvemos el valor de error de la misma manera que devolvió el valor del
error en el `match` que manejaba el valor de retorno de `File::open`.
Sin embargo, no necesitamos decir explícitamente `return`, porque esto
es la última expresión en la función.

El código que llama a este código manejará obtener un valor `Ok`
que contiene un nombre de usuario o un valor `Err` que contiene un
`io::Error`. Nosotros no sabemos qué hará el código de llamada con esos
valores. Si el código de llamada obtiene un valor `Err`, podría llamar `panic!` y bloquear el programa, usar un nombre de usuario predeterminado, o
busque el nombre de usuario desde otro lugar que no sea un archivo, por
ejemplo. No tenemos suficiente información sobre lo que el código de llamada
realmente intenta hacer, por lo que propagamos toda la información de éxito o
error hacia arriba para que se maneje adecuadamente.

Este patrón de errores de propagación es tan común en Rust que Rust
proporciona el operador de signo de interrogación `?` para hacer esto más
fácil.

#### Un atajo para propagar errores: el operador `?`

El Listado 9-7 muestra una implementación de `read_username_from_file` que
tiene la misma funcionalidad que tenía en el listado 9-6, pero esta
implementación usa el operador `?`.

<span class="filename">Filename: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

<span class="caption">Listing 9-7: Una función que devuelve errores al código de llamada utilizando el operador `?`</span>

El `?` colocado después de que se haya definido un valor de `Result` funciona
casi de la misma manera que las expresiones `match` que definimos para
manejar los valores `Result` en el Listado 9-6. Si el valor del `Result` es
un `Ok`, el valor dentro de `Ok` se devolverá a partir de esta expresión, y el programa continuará. Si el valor es un `Err`, el valor dentro de `Err` se devolverá de la función completa como si hubiéramos usado la palabra clave `return` para que el valor de error se propague al código de llamada.

Hay una diferencia entre lo que hacen la expresión `match` del Listado 9-6 y
lo que hace el operador `?`: los valores de error utilizados con `?` Pasan
por la función `from`, definida en el *trair* `From` en la biblioteca
estándar, que se usa para convertir errores de un tipo a otro. Cuando el
operador `?` llama a la función `from`, el tipo de error recibido se
convierte al tipo de error definido en el tipo de retorno de la función
actual. Esto es útil cuando una función devuelve un tipo de error para
representar todas las formas en que una función puede fallar, incluso si las
partes pueden fallar por muchas razones diferentes. Siempre que cada tipo de
error implemente la función `from` para definir cómo convertirse al tipo de
error devuelto, el operador `?` se ocupará de la conversión automáticamente.

En el contexto del Listado 9-7, el `?` al final de la llamada `File::open`
devolverá el valor dentro de un `Ok` a la variable `f`. Si ocurre un error,
el operador `?` saldrá temprano de toda la función y dará cualquier valor
`Err` al código de llamada. Lo mismo se aplica al operador `?` al final de la
llamada `read_to_string`.

El operador `?` Elimina una gran cantidad de texto repetitivo y simplifica la
implementación de esta función. Incluso podríamos acortar este código aún más
mediante el encadenamiento de llamadas de método inmediatamente después del
`?`, Como se muestra en el Listado 9-8.

<span class="filename">Filename: src/main.rs</span>

```rust
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut s = String::new();

    File::open("hello.txt")?.read_to_string(&mut s)?;

    Ok(s)
}
```

<span class="caption">Listado 9-8: Llamadas al método de encadenamiento
después del operador `?`</span>

Hemos movido la creación del nuevo `String` en `s` al comienzo de la función;
esa parte no ha cambiado. En lugar de crear una variable `f`, hemos
encadenado la llamada a `read_to_string` directamente en el resultado de
`File::open("hello.txt")?`. Todavía tenemos un `?` al final de la llamada
`read_to_string`, y aún devolvemos un valor `Ok` que contiene el nombre de
usuario en `s` cuando tanto `File::open` como `read_to_string` tienen éxito en lugar de devolver errores. La funcionalidad es nuevamente la misma que en el Listado 9-6 y el Listado 9-7; esta es solo una forma diferente y más ergonómica de escribirlo.

#### El operador `?` solo se puede usar en funciones que devuelven `Result`

El operador `?` solo puede usarse en funciones que tienen un tipo de retorno
de `Result`, porque está definido para funcionar de la misma manera que la
expresión `match` que definimos en el Listado 9-6. La parte del `match` que
requiere un tipo de devolución de `Result` es `return Err(e)`, por lo que el
tipo de devolución de la función debe ser un `Result` para que sea compatible
con este `return`.

Veamos qué pasa si usamos el operador `?` en la función `main`, que recordará que tiene un tipo de retorno de `()`:

```rust,ignore
use std::fs::File;

fn main() {
    let f = File::open("hello.txt")?;
}
```

Cuando compilamos este código, recibimos el siguiente mensaje de error:

```text
error[E0277]: the trait bound `(): std::ops::Try` is not satisfied
 --> src/main.rs:4:13
  |
4 |     let f = File::open("hello.txt")?;
  |             ------------------------
  |             |
  |             the `?` operator can only be used in a function that returns
  `Result` (or another type that implements `std::ops::Try`)
  |             in this macro invocation
  |
  = help: the trait `std::ops::Try` is not implemented for `()`
  = note: required by `std::ops::Try::from_error`
```

Este error indica que solo podemos usar el operador `?` en una función que
devuelve `Result`. En las funciones que no devuelven `Result`, cuando llama a
otras funciones que devuelven `Result`, necesitará usar `match` o uno de los
métodos `Result` para manejar `Result` en lugar de usar el operador `?` para
propagar potencialmente el error al código de llamada.

Ahora que hemos discutido los detalles de llamar `panic!` o devolver
`Result`, volvamos al tema de cómo decidir en qué casos cuál es el adecuado.
