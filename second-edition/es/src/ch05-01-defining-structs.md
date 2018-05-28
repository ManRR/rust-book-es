## Definir e Instanciar *Structs*

Los *structs* son similares a las *tuples*, que se discutieron en el Capítulo 3.
Al igual que las *tuples*, las piezas de un *structs* pueden ser de tipos diferentes.
A diferencia de las *tuples*, nombrará cada dato para que quede claro lo que significan
los valores. Como resultado de estos nombres, los *structs* son más flexibles que las
*tuples*: no tiene que depender del orden de los datos para especificar o acceder a
los valores de una instancia.

Para definir un *structs*, ingresamos la palabra clave `struct` y nombramos el
*structs* completa. El nombre de un *structs* debe describir el significado de
las piezas de datos que se agrupan. Luego, dentro de las llaves, definimos los nombres
y tipos de los datos, que llamamos *campos*. Por ejemplo, el Listado 5-1 muestra
un *structs* que almacena información sobre una cuenta de usuario.

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

<span class="caption">Listing 5-1: Definición del *struct* `User`</span>

Para usar un *struct* después de haberlo definido, creamos una *instancia*
de ese *struct* especificando valores concretos para cada uno de los campos.
Creamos una instancia indicando el nombre del *struct* y luego agregamos
llaves que contienen pares `key: value`, donde las claves son los nombres de los
campos y los valores son los datos que queremos almacenar en esos campos. No
tenemos que especificar los campos en el mismo orden en que los declaramos en
el *struct*. En otras palabras, la definición de un *structs* es como una plantilla
general para el tipo, y las instancias completan esa plantilla con datos particulares
para crear valores del tipo. Por ejemplo, podemos declarar un usuario particular
como se muestra en el Listado 5-2.

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
let user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};
```

<span class="caption">Listing 5-2: Creando una instancia de `User`
struct</span>

Para obtener un valor específico de un *struct*, podemos usar la notación de
puntos. Si quisiéramos solo la dirección de correo electrónico de este usuario,
podríamos usar `user1.email` donde quisiéramos utilizar este valor. Si la instancia
es mutable, podemos cambiar un valor usando la notación de puntos y la asignación
en un campo particular. El listado 5-3 muestra cómo cambiar el valor en el campo
`email` de una instancia `User` mutable.

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

user1.email = String::from("anotheremail@example.com");
```

<span class="caption">Listing 5-3: Cambiar el valor en el campo `email` de una instancia `User`</span>

Tenga en cuenta que toda la instancia debe ser mutable; Rust no nos permite
marcar solo ciertos campos como mutables. Al igual que con cualquier expresión,
podemos construir una nueva instancia del *struct* como la última expresión
en el cuerpo de la función para devolver implícitamente esa nueva instancia.

El listado 5-4 muestra una función `build_user` que devuelve una instancia `User`
con el correo electrónico y el nombre de usuario dados. El campo `activo` obtiene
el valor de `true`, y `sign_in_count` obtiene un valor de `1`.

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
fn build_user(email: String, username: String) -> User {
    User {
        email: email,
        username: username,
        active: true,
        sign_in_count: 1,
    }
}
```

<span class="caption">Listing 5-4: Una función `build_user` que toma un correo electrónico y un nombre de usuario y devuelve una instancia `User`</span>

Tiene sentido nombrar los parámetros de la función con el mismo nombre que
los campos de un *struct*, pero tener que repetir los nombres y variables de
campo `email` y` username` es un poco tedioso. Si el *struct* tuviera más
campos, repetir cada nombre sería aún más molesto. Afortunadamente,
¡hay una taquigrafía conveniente!

### Uso de la abreviatura *Field Init* cuando las variables y los campos tienen el mismo nombre

Debido a que los nombres de los parámetros y los nombres de los campos *struct* son
exactamente los mismos en el Listado 5-4, podemos usar la sintaxis *field init shorthand*
para reescribir `build_user` para que se comporte exactamente igual pero no tenga la repetición
`email` y `username`, como se muestra en el Listado 5-5.

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
fn build_user(email: String, username: String) -> User {
    User {
        email,
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

<span class="caption">Listing 5-5: Una función `build_user` que usa *field init shorthand*
porque los parámetros` email` y `username` tienen el mismo nombre
que los campos struct</span>

Aquí, estamos creando una nueva instancia de la estructura (*struct*) `User`, que tiene un campo
llamado `email`. Queremos establecer el valor del campo `email` al valor en el parámetro
`email` de la función `build_user`. Debido a que el campo `email` y el parámetro `email`
tienen el mismo nombre, solo necesitamos escribir `email` en lugar de `email: email`.

### Crear instancias desde otras instancias con la sintaxis de actualización de Struct

A menudo es útil crear una nueva instancia de una estructura que utiliza la mayoría de
los valores de una instancia anterior, pero cambia algunos. Harás esto usando
*struct update syntax*.

Primero, el Listado 5-6 muestra cómo creamos una nueva instancia `User` en `user2`
sin la sintaxis de actualización. Establecimos nuevos valores para `email` y `username`
pero de lo contrario usamos los mismos valores de `user1` que creamos en el Listado 5-2.

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
# let user1 = User {
#     email: String::from("someone@example.com"),
#     username: String::from("someusername123"),
#     active: true,
#     sign_in_count: 1,
# };
#
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    active: user1.active,
    sign_in_count: user1.sign_in_count,
};
```

<span class="caption">Listing 5-6: Creando una nueva instancia de `User`
usando algunos de los valores de `user1`</span>

Usando la sintaxis de actualización de *struct*, podemos lograr el mismo efecto
con menos código, como se muestra en el listado 5-7. La sintaxis `..` especifica
que los campos restantes no configurados explícitamente deben tener el mismo valor
que los campos en la instancia dada.

```rust
# struct User {
#     username: String,
#     email: String,
#     sign_in_count: u64,
#     active: bool,
# }
#
# let user1 = User {
#     email: String::from("someone@example.com"),
#     username: String::from("someusername123"),
#     active: true,
#     sign_in_count: 1,
# };
#
let user2 = User {
    email: String::from("another@example.com"),
    username: String::from("anotherusername567"),
    ..user1
};
```

<span class="caption">Listing 5-7: Usar *struct update syntax* para establecer nuevos
valores de `email` y `username` para una instancia `User`, pero use el resto de los
valores de los campos de la instancia en la variable `user1`</span>

El código en el Listado 5-7 también crea una instancia en `user2` que tiene un valor
diferente para `email` y `username` pero tiene los mismos valores para los campos
`active` y `sign_in_count` de `user1`.

### Usar *Tuple Structs* sin campos con nombre para crear diferentes tipos

También puede definir estructuras que se parecen a las tuplas, llamadas
*tuple
structs* (*estructuras de tupla*). Las estructuras Tupla tienen el significado añadido que
proporciona el nombre de la estructura, pero no tienen nombres asociados con sus
campos; más bien, solo tienen los tipos de los campos. Las estructuras Tupla son
útiles cuando se quiere dar un nombre a la tupla completa y hacer que la tupla sea
un tipo diferente que otras tuplas, y nombrar cada campo como en una estructura
regular sería detallado o redundante.

Para definir una estructura de tupla, comience con la palabra clave `struct` y
el nombre de la estructura seguido de los tipos en la tupla. Por ejemplo, aquí
hay definiciones y usos de dos estructuras de tupla denominadas `Color` y `Point`:

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);
```

Tenga en cuenta que los valores `black` y `origin` son de tipos diferentes,
porque son instancias de diferentes estructuras de tupla. Cada estructura que
define es de su propio tipo, aunque los campos dentro de la estructura tienen
los mismos tipos. Por ejemplo, una función que toma un parámetro de tipo `Color`
no puede tomar un `Point` como argumento, aunque ambos tipos están compuestos
por tres valores `i32`. De lo contrario, las instancias de tupla struct se
comportan como tuplas: puede desestructurarlas en sus piezas individuales,
puede usar un `.` seguido del índice para acceder a un valor individual,
y así sucesivamente.

### *Unit-Like* Estructura sin ningún campo

¡También puedes definir estructuras que no tienen ningún campo! Estos se denominan
*unit-like structs* porque se comportan de manera similar a `()`,
el tipo de unidad. *Unit-like structs* pueden ser útiles en situaciones en
las que necesita implementar un rasgo en algún tipo pero no tiene ningún dato que desee
almacenar en el tipo mismo. Discutiremos los rasgos en el Capítulo 10.

> ### Ownership of Struct Data
>
> En la definición de estructura `User` en el listado 5-1, utilizamos el tipo
> `String` propiedad en lugar del tipo de sección de cadena `&str`. Esta es
> una elección deliberada porque queremos que las instancias de esta
> estructura sean propietarias de todos sus datos y que los datos sean
> válidos mientras toda la estructura sea válida.
>
> Es posible que las estructuras almacenen referencias a datos propiedad
> de otra cosa, pero para hacerlo se requiere el uso de *lifetimes*, una
> característica de Rust que discutiremos en el Capítulo 10. Lifetimes
> garantiza que los datos a los que hace referencia una estructura son
> válidos para siempre que la estructura sea Digamos que intenta almacenar
> una referencia en una estructura sin especificar vidas, como esta, que
> no funcionará:
>
> <span class="filename">Filename: src/main.rs</span>
>
> ```rust,ignore
> struct User {
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
>     active: bool,
> }
>
> fn main() {
>     let user1 = User {
>         email: "someone@example.com",
>         username: "someusername123",
>         active: true,
>         sign_in_count: 1,
>     };
> }
> ```
>
> El compilador se quejará de que necesita especificadores *lifetime*:
>
>
> ```text
> error[E0106]: missing lifetime specifier
>  -->
>   |
> 2 |     username: &str,
>   |               ^ expected lifetime parameter
>
> error[E0106]: missing lifetime specifier
>  -->
>   |
> 3 |     email: &str,
>   |            ^ expected lifetime parameter
> ```
>
> En el Capítulo 10, discutiremos cómo solucionar estos errores para que pueda
> almacenar referencias en estructuras, pero por ahora, corregiremos errores como
> estos utilizando tipos propios como `String` en lugar de referencias como `&str`.
