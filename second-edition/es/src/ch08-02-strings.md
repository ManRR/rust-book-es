## Almacenamiento de texto codificado en UTF-8 con *Strings*

Hablamos sobre *strings* de caracteres en el Capítulo 4, pero las veremos con
más profundidad ahora. Los nuevos Rustaceans comúnmente se atascan en los
*strings* por una combinación de tres razones: la propensión de Rust a
exponer posibles errores, los *strings* son una estructura de datos más
complicada de lo que muchos programadores les dan crédito, y UTF-8. Estos
factores se combinan de una manera que puede parecer difícil cuando proviene
de otros lenguajes de programación.

Es útil analizar los *strings* en el contexto de las colecciones porque los
*strings* se implementan como una colección de bytes, además de algunos
métodos para proporcionar una funcionalidad útil cuando esos bytes se
interpretan como texto. En esta sección, hablaremos sobre las operaciones en
`String` que tiene cada tipo de colección, como crear, actualizar y leer.
También discutiremos las formas en que `String` es diferente de las otras
colecciones, es decir, cómo la indexación en un `String` se complica por las
diferencias entre la forma en que las personas y las computadoras interpretan
los datos de `String`.

### ¿Qué es un *String*?

Primero definiremos lo que queremos decir con el término *string*. Rust solo
tiene un tipo de *string* en el nucleo del lenguaje, que es el segmento de
*string* `str` que se ve generalmente en su forma prestada `& str`. En el
Capítulo 4, hablamos sobre *string slices*, que son referencias a algunos
datos de *string* codificados en UTF-8 almacenados en otro lugar. *String*
literales, por ejemplo, se almacenan en la salida binaria del programa y son
por lo tanto, secciones de *string*.

El tipo `String`, proporcionado por la biblioteca estándar de Rust en lugar de
codificado en el lenguaje central, es un código UTF-8 creable, mutable, de
propiedad, tipo de *string* codificado. Cuando los Rustaceos se refieren a
*string* en Rust, generalmente se refieren a lo tipos `String` y los *string
slice* `&str`, no solo uno de esos tipos.
Aunque esta sección trata principalmente sobre `String`, ambos tipos se usan
mucho en la biblioteca estándar de Rust, y tanto `String` como las *string
slice* están codificadas en UTF-8.

La biblioteca estándar de Rust también incluye varios otros tipos de *string*,como `OsString`, `OsStr`, `CString`, y `CStr`. Los *library crates* pueden
proporcionar incluso más opciones para almacenar datos de *string*. ¿Ve cómo todos esos nombres terminan en `String` o `Str`? Se refieren a variantes
propias y prestadas, al igual que el tipo `String` y `str` que has visto
anteriormente. Estos tipos de *string* pueden almacenar texto en diferentes
codificaciones o ser representado en la memoria de una manera diferente, por
ejemplo. No discutiremos estos otros tipos de *string* en este capítulo; ver
su documentación API para obtener más información sobre cómo usarlos y cuándo
es cada uno apropiado.

### Creando un nuevo *string*

Muchas de las mismas operaciones disponibles con `Vec <T>` están disponibles con `String` también, comenzando con la función `new` para crear una *string* que se muestra en el Listado 8-11.

```rust
let mut s = String::new();
```

<span class="caption">Listing 8-11: Creando un nuevo `String` vacío </span>

Esta línea crea un nuevo *string* vacío llamado `s`, en el que luego podemos
cargar datos. A menudo, tendremos algunos datos iniciales con los que
queremos comenzar el *string*. Para eso, usamos el método `to_string`, que
está disponible en cualquier tipo que implemente el *trait* `Display`, como
lo hacen los literales *string*. El Listado 8-12 muestra dos ejemplos.

```rust
let data = "initial contents";

let s = data.to_string();

// el método también funciona en un literal directamente:
let s = "initial contents".to_string();
```

<span class="caption">Listing 8-12: Usando el método `to_string` para crear
un `String` a partir de un *string* literal </span>

Este código crea un *string* que contiene `initial contents`.

También podemos usar la función `String::from` para crear un `String` a
partir de un *string* literal. El código en el Listado 8-13 es equivalente al
código del Listado 8-12 que usa `to_string`.

```rust
let s = String::from("initial contents");
```

<span class="caption">Listing 8-13: Usando la función `String::from` para
crear un `String` a partir de un *string* literal</span>

Como los *string* se utilizan para muchas cosas, podemos usar muchas API
genéricas diferentes para *string*, lo que nos brinda muchas opciones.
Algunos de ellos pueden parecer redundantes, ¡pero todos tienen su lugar! En
este caso, `String::from` y `to_string` hacen lo mismo, por lo que eligir uno u otro es una cuestión de estilo.

Recuerde que los *string* están codificadas en UTF-8, por lo que podemos
incluir cualquier información codificada correctamente en ellas, como se
muestra en el Listado 8-14.

```rust
let hello = String::from("السلام عليكم");
let hello = String::from("Dobrý den");
let hello = String::from("Hello");
let hello = String::from("שָׁלוֹם");
let hello = String::from("नमस्ते");
let hello = String::from("こんにちは");
let hello = String::from("안녕하세요");
let hello = String::from("你好");
let hello = String::from("Olá");
let hello = String::from("Здравствуйте");
let hello = String::from("Hola");
```

<span class="caption">Listing 8-14: Almacenando de saludos en diferentes
idiomas en *string*</span>

Todos estos son valores válidos de `String`.

### Actualizar un *String*

Un `String` puede crecer en tamaño y su contenido puede cambiar, al igual que
el contenido de un `Vec <T>`, si inserta más datos en él. Además, puede usar
convenientemente el operador `+` o la macro `format!` Para concatenar los
valores `String`.

#### Añadiendo a un *String* con `push_str` y `push`

Podemos hacer crecer un `String` utilizando el método `push_str` para añadir
un segmento de *string*, como se muestra en el Listado 8-15.

```rust
let mut s = String::from("foo");
s.push_str("bar");
```

<span class="caption">Listing 8-15: Agregar un segmento de *string* a un
`String` usando el método `push_str`</span>

Después de estas dos líneas, `s` contendrá `foobar`. El método `push_str`
toma un segmento de un *string* porque no necesariamente queremos tomar
posesión del parámetro. Por ejemplo, el código en el Listado 8-16 muestra que
sería desafortunado si no pudiéramos usar `s2` después de agregar sus
contenidos a `s1`.

```rust
let mut s1 = String::from("foo");
let s2 = "bar";
s1.push_str(s2);
println!("s2 is {}", s2);
```

<span class="caption">Listing 8-16: Usando un segmento de *string* después de
añadir su contenido a un `String`</span>

Si el método `push_str` se apropiara de `s2`, no podríamos imprimir su valor
en la última línea. Sin embargo, este código funciona como esperábamos.

El método `push` toma un solo carácter como parámetro y lo agrega a `String`.
El Listado 8-17 muestra un código que agrega la letra *l* a un `String`
utilizando el método `push`.

```rust
let mut s = String::from("lo");
s.push('l');
```

<span class="caption">Listing 8-17: Agregar un carácter a un valor `String`
con `push`</span>

Como resultado de este código, `s` contendrá `lol`.

#### Concatenación con el operador `+` o la Macro `format!`

A menudo, querrás combinar dos *string* existentes. Una forma es usar el operador `+`, como se muestra en el Listado 8-18.

```rust
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // nota s1 se ha movido aquí y ya no se puede usar
```

<span class="caption">Listing 8-18: Usando el operador `+` para combinar dos
valores `String` en un nuevo valor 'String`</span>

El *string* `s3` contendrá `Hello, world!` como resultado de este código. El
motivo `s1` ya no es válido después de la adición y el motivo por el que
usamos una referencia a `s2` tiene que ver con la firma del método que se
llama cuando usamos el operador `+`. El operador `+` usa el método `add`,
cuya firma se ve más o menos así:

```rust,ignore
fn add(self, s: &str) -> String {
```

Esta no es la firma exacta que está en la biblioteca estándar: en la
biblioteca estándar, `add` se define usando genéricos. Aquí, estamos viendo
la firma de `add` con tipos de concreto sustituidos por los genéricos, que es
lo que sucede cuando llamamos a este método con valores `String`.
Discutiremos los genéricos en el Capítulo 10. Esta firma nos da las pistas
que necesitamos para comprender las partes difíciles del operador `+`.

Primero, `s2` tiene un `&`, lo que significa que estamos agregando una
*referencia* de el segundo *string* a la primera *string* debido al parámetro `s`
en la función `add`: solo podemos agregar un `& str`a un `String`; no podemos
agregar dos valores `String` juntos. Pero espera, el tipo de `&s2` es
`& String`, no `&str`, como se especifica en el segundo parámetro para `add`. Entonces, ¿por qué compila el Listado 8-18?

La razón por la que podemos usar `&s2` en la llamada a `add` es que el
compilador puede *forzar* el argumento `&String` en un `&str`. Cuando
llamamos al método `add`, Rust usa una coerción *deref*, que aquí convierte
`&s2` en `&s2[..]`. Analizaremos la coerción de *deref* con más profundidad
en el Capítulo 15. Como `add` no toma posesión del parámetro `s`, `s2`
seguirá siendo un `String` válido después de esta operación.

En segundo lugar, podemos ver en la firma que `add` toma posesión de `self`,
porque `self` *no* tiene un `&`. Esto significa que `s1` en el Listado 8-18
se moverá a la llamada `add` y ya no será válido después de eso. Entonces,
aunque `let s3 = s1 + &s2;` parece que copiará ambos *strings* y creará una
nueva, esta declaración toma posesión de `s1`, agrega una copia del contenido
de `s2`, y luego devuelve la propiedad de el resultado. En otras palabras,parece que está haciendo muchas copias pero no lo está; la implementación es más eficiente que la copia.

Si necesitamos concatenar varias *strings*, el comportamiento del operador `+`
se vuelve difícil de manejar:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = s1 + "-" + &s2 + "-" + &s3;
```

En este punto, `s` será `tic-tac-toe`. Con todos los caracteres `+` y `"`,
es difícil ver qué está sucediendo. Para una combinación de *string* más
complicada, podemos usar la macro `format!`:

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let s = format!("{}-{}-{}", s1, s2, s3);
```

Este código también establece `s` en `tic-tac-toe`. La macro `format!`
funciona de la misma manera que `println!`, pero en lugar de imprimir la
salida a la pantalla, devuelve un `String` con los contenidos. La versión del
código que utiliza `format!` es mucho más fácil de leer y no toma posesión de
ninguno de sus parámetros.

### Indexación en *Strings*

En muchos otros lenguajes de programación, el acceso a caracteres
individuales en un *string* haciendo referencia a ellos por índice es una
operación válida y común. Sin embargo, si intenta acceder a partes de un
`String` usando la sintaxis de indexación en Rust, obtendrá un error.
Considere el código inválido en el Listado 8-19.

```rust,ignore
let s1 = String::from("hello");
let h = s1[0];
```

<span class="caption">Listing 8-19: Intentando utilizar la sintaxis de indexación con un String</span>

Este código dará como resultado el siguiente error:

```text
error[E0277]: the trait bound `std::string::String: std::ops::Index<{integer}>` is not satisfied
 -->
  |
3 |     let h = s1[0];
  |             ^^^^^ the type `std::string::String` cannot be indexed by `{integer}`
  |
  = help: the trait `std::ops::Index<{integer}>` is not implemented for `std::string::String`
```

El error y la nota cuentan la historia: los *string* de Rust no son
compatibles con la indexación. ¿Pero por qué no? Para responder a esa
pregunta, debemos analizar cómo Rust almacena *strings*en la memoria.

#### Representación interna

Un `String` es un contenedor sobre un `Vec <u8>`. Veamos algunas de nuestros
*string* de ejemplo UTF-8 codificadas correctamente del Listado 8-14. Primero
este:

```rust
let len = String::from("Hola").len();
```

En este caso, `len` será 4, lo que significa que el vector que almacena el
*string* “Hola” tiene 4 bytes de longitud. Cada una de estas letras toma 1
byte cuando está codificada en UTF-8. Pero, ¿qué pasa con la siguiente línea?
(Tenga en cuenta que este *string* comienza con la letra cirílica mayúscula Ze,
no el número árabe 3.)

```rust
let len = String::from("Здравствуйте").len();
```

Si se pregunta, cuán largo es el *string*, podría decir 12. Sin embargo,la respuesta de Rust es 24: ese es el número de bytes que se necesita para
codificar "Здравствуйте" en UTF-8, ya que cada valor escalar Unicode toma 2
bytes de almacenamiento. Por lo tanto, un índice en los bytes del *string* no
siempre se correlacionará con un valor escalar Unicode válido. Para demostrar,considere este código de Rust no válido:

```rust,ignore
let hello = "Здравствуйте";
let answer = &hello[0];
```

¿Cuál debería ser el valor de `answer`? ¿Debería ser `З`, la primera letra?
Cuando está codificado en UTF-8, el primer byte de `З` es `208` y el segundo
es `151`, por lo que `answer` debería ser `208`, pero `208` no es un carácter
válido por sí mismo. Devolver `208` probablemente no sea lo que un usuario
desearía si pidieran la primera letra de este *string*; sin embargo, esos son
los únicos datos que -Rust tiene un índice de bytes 0-. Los usuarios
generalmente no desean que se devuelva el valor del byte, incluso si el
*string* solo contiene letras latinas: si `&"hello"[0]` eran códigos válidos
que devolvían el valor de byte, devolvería `104`, no `h`. Para evitar
devolver un valor inesperado y causar errores que podrían no descubrirse de
inmediato, Rust no compila este código en absoluto y evita malos entendidos
al principio del proceso de desarrollo.

#### ¡Bytes y valores escalares y clusters de Grafema! ¡Oh mi!

Otro punto acerca de UTF-8 es que en realidad hay tres formas relevantes de
mirar los *string* desde la perspectiva de Rust: como bytes, valores
escalares y clusters de grafemas (lo más parecido a lo que llamaríamos
*letras*).

Si miramos la palabra Hindi "नमस्ते" escrita en la secuencia de comandos
Devanagari, se almacena como un vector de valores 'u8` que se ve así:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

Eso es 18 bytes y es la forma en que las computadoras finalmente almacenan
estos datos. Si los vemos como valores escalares Unicode, que son el tipo
`char` de Rust, esos bytes se ven así:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

Aquí hay seis valores `char`, pero el cuarto y el sexto no son letras: son
diacríticos que no tienen sentido por sí mismos. Finalmente, si los vemos
como grupos de grafemas, obtendríamos lo que una persona llamaría las cuatro
letras que componen la palabra hindi:

```text
["न", "म", "स्", "ते"]
```

Rust proporciona diferentes formas de interpretar los datos de *string* sin
formato que almacenan las computadoras para que cada programa pueda elegir la
interpretación que necesita, sin importar en qué idioma humano se encuentren
los datos.

Una razón final por la que Rust no nos permite indexar en un `String` para
obtener un carácter es que las operaciones de indexación siempre toman un
tiempo constante (O (1)). Pero no es posible garantizar ese rendimiento con
un `String`, porque Rust tendría que recorrer el contenido desde el principio
hasta el índice para determinar cuántos caracteres válidos había.

### Cortando *Strings*

La indexación en un *string* a menudo es una mala idea porque no está claro
cuál debería ser el tipo de devolución de la operación de indexación de
*string*: un valor de byte, un carácter, un clúster de grafemas o un segmento
de *string*. Por lo tanto, Rust le pide que sea más específico si realmente
necesita usar índices para crear secciones de *string* (*string slices*).
Para ser más específico en su indexación e indicar que desea un segmento de
*string* (*string slices*), en lugar de indexar usando `[]` con un solo
número, puede usar `[]` con un rango para crear un segmento de *string*
(*string slices*) que contenga bytes particulares:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Aquí, `s` será un `& str` que contiene los primeros 4 bytes del *string*.
Anteriormente, mencionamos que cada uno de estos caracteres tenía 2 bytes, lo
que significa que `s` será `Зд`.

¿Qué pasaría si usáramos `&hello[0..1]`? La respuesta: Rust entraría en
pánico en tiempo de ejecución de la misma manera que si se accede a un índice
no válido en un vector:

```text
thread 'main' panicked at 'byte index 1 is not a char boundary; it is inside 'З' (bytes 0..2) of `Здравствуйте`', src/libcore/str/mod.rs:2188:4
```

Debe usar rangos para crear secciones de *string* con precaución, ya que al hacerlo puede bloquear su programa.

### Métodos para iterar sobre *Strings*

Afortunadamente, puede acceder a elementos en un *string* de otras maneras.

Si necesita realizar operaciones en valores escalares Unicode individuales,
la mejor manera de hacerlo es usar el método `chars`. Llamar `chars` en“नमस्ते” se separa y devuelve seis valores de tipo `char`, y puede iterar sobre el resultado para acceder a cada elemento:

```rust
for c in "नमस्ते".chars() {
    println!("{}", c);
}
```

Este código imprimirá lo siguiente:

```text
न
म
स
्
त
े
```

El método `bytes` devuelve cada byte sin formato, que podría ser apropiado
para su dominio:

```rust
for b in "नमस्ते".bytes() {
    println!("{}", b);
}
```

Este código imprimirá los 18 bytes que componen este `String`:

```text
224
164
// --snip--
165
135
```

Pero asegúrese de recordar que los valores escalares Unicode válidos pueden
estar formados por más de 1 byte.

Obtener clústeres de grafema a partir de *strings*es complejo, por lo que la
biblioteca estándar no proporciona esta funcionalidad. Los *Crates* están
disponibles en [crates.io](https://crates.io) si esta es la funcionalidad que
necesita.

### Los *Strings* no son tan simples

Para resumir, los *string* son complicados. Diferentes lenguajes de
programación toman diferentes decisiones sobre cómo presentar esta
complejidad al programador. Rust ha elegido hacer que el manejo correcto de
los datos de `String` sea el comportamiento predeterminado de todos los
programas de Rust, lo que significa que los programadores tienen que pensar
más en manejar los datos de UTF-8 por adelantado. Este *trade-off* expone más
de la complejidad de los *string* de lo que es evidente en otros lenguajes de
programación, pero evita que tenga que manejar errores que involucren
caracteres no ASCII más adelante en su ciclo de vida de desarrollo.

Cambiemos a algo un poco menos complejo: ¡*hash maps*!
