## El tipo Slice

Otro tipo de datos que no tiene propiedad es el *slice*. Las divisiones le
permiten hacer referencia a una secuencia contigua de elementos en una
colección en lugar de a toda la colección.

Aquí hay un pequeño problema de programación: escriba una función que toma
una cadena y devuelve la primera palabra que encuentra en esa cadena. Si la
función no encuentra un espacio en la cadena, toda la cadena debe ser una
palabra, por lo que se debe devolver toda la cadena.

Pensemos en la firma de esta función:

```rust,ignore
fn first_word(s: &String) -> ?
```

Esta función, `first_word`, tiene un `&String` como parámetro. No queremos
ser propietarios (*ownership*), así que está bien. Pero, ¿qué deberíamos devolver?
Realmente no tenemos una forma de hablar sobre *parte* de una cadena. Sin embargo,
podríamos devolver el índice del final de la palabra. Probemos eso, como se
muestra en el Listado 4-7.

<span class="filename">Filename: src/main.rs</span>

```rust
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

<span class="caption">Listing 4-7: La función `first_word` que devuelve un valor de
índice de bytes en el parámetro `String`</span>

Debido a que necesitamos pasar por el elemento `String` por elemento y
comprobar si un valor es un espacio, convertiremos nuestro `String`
a una matriz de bytes utilizando el método `as_bytes`:

```rust,ignore
let bytes = s.as_bytes();
```

A continuación, creamos un iterador sobre la matriz de bytes utilizando el método `iter`:

```rust,ignore
for (i, &item) in bytes.iter().enumerate() {
```

Analizaremos iteradores con más detalle en el Capítulo 13. Por ahora, sepa
que `iter` es un método que devuelve cada elemento de una colección y que
`enumerate` ajusta el resultado de `iter` y devuelve cada elemento como
parte de un tupla en su lugar. El primer elemento de la tupla devuelta de
`enumerate` es el índice, y el segundo elemento es una referencia al elemento.
Esto es un poco más conveniente que calcular el índice nosotros mismos.

Debido a que el método `enumerate` devuelve una tupla, podemos usar patrones
para desestructurar esa tupla, al igual que en cualquier otro lugar en Rust.
Entonces en el ciclo `for`, especificamos un patrón que tiene `i` para el
índice en la tupla y `&item` para el byte simple en la tupla. Como obtenemos
una referencia al elemento de `.iter (). Enumerate ()`, usamos `&` en el patrón.

Dentro del bucle `for`, buscamos el byte que representa el espacio usando la sintaxis
literal de byte. Si encontramos un espacio, devolvemos la posición. De lo contrario,
devolvemos la longitud de la cadena usando `s.len ()`:

```rust,ignore
    if item == b' ' {
        return i;
    }
}

s.len()
```

Ahora tenemos una forma de averiguar el índice del final de la primera palabra en la cadena,
pero hay un problema. Estamos devolviendo un `usize` por sí mismo, pero es solo un número
significativo en el contexto de `&String`. En otras palabras, como es un valor separado del
`String`, no hay garantía de que siga siendo válido en el futuro. Considere el programa en
el Listado 4-8 que usa la función `first_word` del Listado 4-7.

<span class="filename">Filename: src/main.rs</span>

```rust
# fn first_word(s: &String) -> usize {
#     let bytes = s.as_bytes();
#
#     for (i, &item) in bytes.iter().enumerate() {
#         if item == b' ' {
#             return i;
#         }
#     }
#
#     s.len()
# }
#
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // palabra obtendrá el valor 5

    s.clear(); // esto vacía el String, haciéndolo igual a ""

    // la palabra todavía tiene el valor 5 aquí, pero no hay más string
    // con las que podamos usar significativamente el valor 5. ¡la palabra ahora es totalmente inválida!
}
```

<span class="caption">Listing 4-8: Almacenar el resultado de llamar a la función `first_word`
y luego cambiar el contenido de `String`</span>

Este programa compila sin ningún error y también lo haría si usáramos `word`
después de llamar a `s.clear ()`. Debido a que `word` no está conectado al estado
de `s` en absoluto, `word` todavía contiene el valor `5`. Podríamos usar ese valor
`5` con la variable `s` para intentar extraer la primera palabra, pero esto sería un
error porque el contenido de `s` ha cambiado desde que guardamos `5` en `word`.

¡Tener que preocuparse de que el índice en `word` esté fuera de sincronía con los
datos en `s` es tedioso y propenso a errores! La gestión de estos índices es aún
más frágil si escribimos una función `second_word`. Su firma debería verse así:

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

Ahora estamos rastreando un índice de inicio *y* final, y tenemos
aún más valores que se calcularon a partir de datos en un estado particular,
pero que no están vinculados a ese estado en absoluto. Ahora tenemos tres variables
no relacionadas flotando alrededor que deben mantenerse sincronizadas.

Afortunadamente, Rust tiene una solución a este problema: string slices.

### String Slices

Un *string slice* es una referencia a parte de un `String`, y se ve así:

```rust
let s = String::from("hello world");

let hello = &s[0..5];
let world = &s[6..11];
```

Esto es similar a tomar una referencia a todo el `String` pero con el bit
adicional `[0..5] `. En lugar de una referencia a todo el `String`, es una
referencia a una parte del `String`. La sintaxis `start..end` es un rango
que comienza en `start` y continúa hasta, pero sin incluir, `end`.

Podemos crear sectores utilizando un rango entre paréntesis especificando
`[starting_index..ending_index]`, donde `starting_index` es la primera posición
en el sector y `ending_index` es uno más que la última posición del sector.
Internamente, la estructura de datos de corte almacena la posición de inicio
y la longitud de la división, que corresponde a `índice_determinado`
menos `índice_inicio`. Entonces, en el caso de `let world = & s [6..11];`,
`world` sería una porción que contiene un puntero al sexto byte de `s`
con un valor de longitud de 5.

La figura 4-6 muestra esto en un diagrama.

<img alt="world containing a pointer to the 6th byte of String s and a length 5" src="img/trpl04-06.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-6: String slice refiriéndose a parte de un
`String`</span>

Con la sintaxis del rango `..` de Rust, si desea comenzar en el primer índice (cero),
puede soltar el valor antes de los dos períodos. En otras palabras, estos son iguales:

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

De la misma manera, si su segmento incluye el último byte de `String`, puede soltar el
número final. Eso significa que estos son iguales:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

También puede soltar ambos valores para tomar una porción de la cadena completa.
Entonces estos son iguales:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> Nota: Los índices de rango de *String slice* deben ocurrir en límites de
> caracteres UTF-8 válidos. Si intenta crear un segmento de cadena en el medio
> de un carácter multibyte, su programa saldrá con un error. A los efectos de la
> introducción de segmentos de cadena, asumimos ASCII solo en esta sección;
> una discusión más completa del manejo de UTF-8 se encuentra en la sección
> “Almacenamiento de texto codificado en UTF-8 con Strings” del Capítulo 8.

Con toda esta información en mente, reescribamos `first_word` para devolver un
slice. El tipo que significa “string slice” se escribe como `&str`:

<span class="filename">Filename: src/main.rs</span>

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

Obtenemos el índice para el final de la palabra de la misma manera que lo
hicimos en el Listado 4-7, al buscar la primera aparición de un espacio.
Cuando encontramos un espacio, devolvemos un segmento de cadena usando el
inicio de la cadena y el índice del espacio como los índices inicial y final.

Ahora cuando llamamos a `first_word`, recuperamos un único valor que está
vinculado a los datos subyacentes. El valor se compone de una referencia al
punto inicial de la división y la cantidad de elementos en la división.

Devolver un segmento también funcionaría para una función `second_word`:

```rust,ignore
fn second_word(s: &String) -> &str {
```

Ahora tenemos una API directa que es mucho más difícil de desordenar, porque el
compilador asegurará que las referencias en `String` sigan siendo válidas.
¿Recuerdas el error en el programa en el listado 4-8, cuando obtuvimos el índice
al final de la primera palabra pero luego borramos la cadena para que nuestro índice
no fuera válido? Ese código era lógicamente incorrecto pero no mostró ningún error
inmediato. Los problemas aparecerían más adelante si seguimos intentando usar el
primer índice de palabras con una cadena vacía. Las rebanadas hacen que este error
sea imposible y háganos saber que tenemos un problema con nuestro código mucho antes.
El uso de la versión de corte de `first_word` arrojará un error en tiempo de compilación:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear(); // error!
}
```

Aquí está el error del compilador:

```text
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
 --> src/main.rs:6:5
  |
4 |     let word = first_word(&s);
  |                            - immutable borrow occurs here
5 |
6 |     s.clear(); // error!
  |     ^ mutable borrow occurs here
7 | }
  | - immutable borrow ends here
```

Recuerde de las reglas de préstamo que si tenemos una referencia inmutable a algo,
tampoco podemos tomar una referencia mutable. Como `clear` necesita truncar el `String`,
intenta tomar una referencia mutable, que falla. Rust no solo ha hecho que nuestra API
sea más fácil de usar, ¡sino que también ha eliminado toda una clase de errores en
tiempo de compilación!

#### String Literals Are Slices

Recordemos que hablamos sobre *string* literales que se almacenan dentro
del binario. Ahora que sabemos acerca de las divisiones, podemos entender
correctamente los *string* literales:

```rust
let s = "Hello, world!";
```

El tipo de `s` aquí es `&str`: es un *slice* que apunta a ese punto específico del
binario. Esta es también la razón por la cual los literales de *string* son inmutables;
`&str` es una referencia inmutable.

#### String Slices como parámetros

Saber que puedes tomar porciones de literales y valores de `String` nos lleva
a una mejora más en `first_word`, y esa es su firma:

```rust,ignore
fn first_word(s: &String) -> &str {
```

Un *Rustacean* más experimentado escribiría la firma que se muestra en el Listado 4-9
porque nos permite utilizar la misma función en los valores de `String` y `&str`.

```rust,ignore
fn first_word(s: &str) -> &str {
```

<span class="caption">Listing 4-9: Mejorando la función `first_word` usando un *slice string*
para el tipo del parámetro `s`</span>

Si tenemos un *string slice*, podemos pasarlo directamente. Si tenemos un `String`,
podemos pasar una porción del `String` entero. Definir una función para tomar un segmento
de cadena en lugar de una referencia a un `String` hace que nuestra API sea más general y
útil sin perder ninguna funcionalidad:

<span class="filename">Filename: src/main.rs</span>

```rust
# fn first_word(s: &str) -> &str {
#     let bytes = s.as_bytes();
#
#     for (i, &item) in bytes.iter().enumerate() {
#         if item == b' ' {
#             return &s[0..i];
#         }
#     }
#
#     &s[..]
# }
fn main() {
    let my_string = String::from("hello world");

    // first_word works on slices of `String`s
    let word = first_word(&my_string[..]);

    let my_string_literal = "hello world";

    // first_word works on slices of string literals
    let word = first_word(&my_string_literal[..]);

    // Because string literals *are* string slices already,
    // this works too, without the slice syntax!
    let word = first_word(my_string_literal);
}
```

### Otros Slices

String slices, como puedes imaginar, son específicas de los *strings*.
Pero hay un tipo de *slice* más general, también. Considera esta matriz:

```rust
let a = [1, 2, 3, 4, 5];
```

Del mismo modo que podríamos querer referirnos a una parte de un *string*,
es posible que deseemos referirnos a parte de una matriz. Lo haríamos así:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];
```

Este *slice* tiene el tipo `&[i32]`. Funciona de la misma manera que los *string slices*,
almacenando una referencia al primer elemento y una longitud. Utilizará este
tipo de *slices* para todo tipo de otras colecciones. Hablaremos de estas colecciones en detalle
cuando hablemos de vectores en el Capítulo 8.

## Resumen

Los conceptos de propiedad (*ownership*), endeudamiento (*borrowing*) y slices garantizan
la seguridad de la memoria en los programas de Rust en tiempo de compilación.
El lenguaje Rust le permite controlar el uso de la memoria de la misma manera que otros
lenguajes de programación de sistemas, pero que el propietario de los datos limpie automáticamente
esos datos cuando el propietario se sale del alcance significa que no tiene que escribir
y depurar código adicional para obtener este control

La propiedad afecta el funcionamiento de muchas otras partes de Rust, por lo que hablaremos
de estos conceptos más a lo largo del resto del libro. Pasemos al Capítulo 5 y veamos cómo
agrupar piezas de datos en un `struct`.
