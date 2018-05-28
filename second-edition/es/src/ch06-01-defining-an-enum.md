## Definiendo un *Enum*

Veamos una situación que podríamos querer expresar en el código y veamos por qué
las enumeraciones (*Enum*) son útiles y más apropiadas que las estructuras en este caso.
Digamos que necesitamos trabajar con direcciones IP. Actualmente, se usan dos estándares
principales para las direcciones IP: la versión cuatro y la versión seis. Estas son las
únicas posibilidades para una dirección IP que nuestro programa encontrará: podemos *enumerar*
todos los valores posibles, que es donde la enumeración obtiene su nombre.

Cualquier dirección IP puede ser una versión cuatro o una versión seis, pero no ambas al
mismo tiempo. Esa propiedad de las direcciones IP hace que la estructura de datos enum
sea apropiada, porque los valores enum solo pueden ser una de las variantes. Ambas direcciones,
la versión cuatro y la versión seis, siguen siendo fundamentalmente direcciones IP, por lo
que deben tratarse del mismo tipo cuando el código maneja situaciones que se aplican a
cualquier tipo de dirección IP.

Podemos expresar este concepto en código definiendo una enumeración `IpAddrKind` y
enumerando los tipos posibles que una dirección IP puede ser, `V4` y `V6`. Estas son
conocidas como las *variantes* de la enumeración:

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

`IpAddrKind` ahora es un tipo de datos personalizado que podemos usar en cualquier parte de nuestro código.

### Valores *Enum*

Podemos crear instancias de cada una de las dos variantes de `IpAddrKind` de esta manera:

```rust
# enum IpAddrKind {
#     V4,
#     V6,
# }
#
let four = IpAddrKind::V4;
let six = IpAddrKind::V6;
```

Tenga en cuenta que las variantes de la enumeración son espacios de nombre bajo su identificador,
y usamos dos puntos dobles para separar los dos. La razón por la que esto es útil es que
ahora ambos valores `IpAddrKind::V4` y `IpAddrKind::V6` son del mismo tipo: `IpAddrKind`.
Entonces podemos, por ejemplo, definir una función que tome cualquier `IpAddrKind`:

```rust
# enum IpAddrKind {
#     V4,
#     V6,
# }
#
fn route(ip_type: IpAddrKind) { }
```

Y podemos llamar a esta función con cualquiera de las variantes:

```rust
# enum IpAddrKind {
#     V4,
#     V6,
# }
#
# fn route(ip_type: IpAddrKind) { }
#
route(IpAddrKind::V4);
route(IpAddrKind::V6);
```

Usar enumeraciones tiene aún más ventajas. Pensando más acerca de nuestro tipo de dirección IP,
en este momento no tenemos una forma de almacenar la dirección IP *datos*; solo sabemos qué *tipo* es.
Dado que acaba de aprender sobre las estructuras en el Capítulo 5, puede abordar este problema como
se muestra en el Listado 6-1.

```rust
enum IpAddrKind {
    V4,
    V6,
}

struct IpAddr {
    kind: IpAddrKind,
    address: String,
}

let home = IpAddr {
    kind: IpAddrKind::V4,
    address: String::from("127.0.0.1"),
};

let loopback = IpAddr {
    kind: IpAddrKind::V6,
    address: String::from("::1"),
};
```

<span class="caption">Listing 6-1: Almacenar los datos y la variante `IpAddrKind`
de una dirección IP usando un `struct`</span>

Aquí, hemos definido una estructura `IpAddr` que tiene dos campos: un campo `kind`
que es de tipo `IpAddrKind` (la enumeración que definimos previamente) y un campo
`address` de tipo `String`. Tenemos dos instancias de esta estructura. El primero,
`home`, tiene el valor `IpAddrKind::V4` como su `kind` con los datos de dirección
asociados de `127.0.0.1`. La segunda instancia, `loopback`, tiene la otra variante
de `IpAddrKind` como su valor `kind`,`V6`, y tiene la dirección `::1` asociada a
ella. Hemos utilizado una estructura para agrupar los valores `kind` y `address`,
por lo que ahora la variante está asociada con el valor.

Podemos representar el mismo concepto de una manera más concisa usando solo una enumeración,
en lugar de una enumeración dentro de una estructura, al poner los datos directamente
en cada variante *enum*. Esta nueva definición de la enumeración `IpAddr` dice que las
variantes` V4` y `V6` tendrán valores` String` asociados:

```rust
enum IpAddr {
    V4(String),
    V6(String),
}

let home = IpAddr::V4(String::from("127.0.0.1"));

let loopback = IpAddr::V6(String::from("::1"));
```

Adjuntamos datos a cada variante de la enumeración (*enum*) directamente, por lo que
no hay necesidad de una estructura (*struct*) extra.

Otra ventaja es usar una enumeración en lugar de una estructura: cada variante puede
tener diferentes tipos y cantidades de datos asociados. Las direcciones IP de la versión
cuatro siempre tendrán cuatro componentes numéricos que tendrán valores entre 0 y 255. Si
quisiéramos almacenar las direcciones `V4` como cuatro valores `u8` pero aún expresar las
direcciones `V6` como un valor `Cadena`, no podría con una estructura. Los enums manejan
esta caja con facilidad:

```rust
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);

let loopback = IpAddr::V6(String::from("::1"));
```

Hemos mostrado varias formas diferentes de definir estructuras de datos para
almacenar la versión cuatro y la versión seis de las direcciones IP. Sin embargo,
resulta que querer almacenar direcciones IP y codificar de qué clase son es tan
común que [la biblioteca estándar tiene una definición que podemos usar!] [IpAddr]
<!-- ignore --> Veamos cómo el la biblioteca estándar define `IpAddr`: tiene la
enumeración exacta y las variantes que hemos definido y utilizado, pero incorpora
los datos de dirección dentro de las variantes en forma de dos estructuras diferentes,
que se definen de manera diferente para cada variante:

[IpAddr]: ../../std/net/enum.IpAddr.html

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

Este código ilustra que puede poner cualquier tipo de datos dentro de una variante *enum*:
cadenas, tipos numéricos o estructuras, por ejemplo. ¡Incluso puedes incluir otra enumeración!
Además, los tipos de biblioteca estándar a menudo no son mucho más complicados de lo que se te ocurra.

Tenga en cuenta que, aunque la biblioteca estándar contiene una definición para `IpAddr`,
aún podemos crear y usar nuestra propia definición sin conflicto porque no hemos incorporado
la definición de la biblioteca estándar a nuestro alcance. Hablaremos más sobre cómo poner
tipos dentro del alcance en el Capítulo 7.

Veamos otro ejemplo de una enumeración en el Listado 6-2: esta tiene una amplia variedad
de tipos incrustados en sus variantes.

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

<span class="caption">Listing 6-2: *enum* `Message` cuyas variantes almacenan
diferentes cantidades y tipos de valores</span>

Esta enumeración tiene cuatro variantes con diferentes tipos:

* `Salir` no tiene datos asociados a él en absoluto.
* `Move` incluye una estructura anónima dentro de él.
* `Write` incluye un solo `String`.
* `ChangeColor` incluye tres valores `i32`.

Definir una enumeración con variantes como las del Listado 6-2 es similar a
definir diferentes tipos de definiciones de estructura, excepto que *enum* no usa
la palabra clave `struct` y todas las variantes se agrupan bajo el tipo `Mensaje`. Las
siguientes estructuras podrían contener los mismos datos que tienen las variantes
*enum* anteriores:

```rust
struct QuitMessage; // unit struct
struct MoveMessage {
    x: i32,
    y: i32,
}
struct WriteMessage(String); // tuple struct
struct ChangeColorMessage(i32, i32, i32); // tuple struct
```

Pero si usamos las diferentes estructuras, que cada una tiene su propio tipo
no podríamos definir fácilmente una función para tomar cualquiera de estos
tipos de mensajes como lo haríamos con la enumeración `Message` definida en el
Listado 6-2, que es un solo tipo.

Hay una similitud más entre las enumeraciones (*enums*) y las estructuras (*structs*):
del mismo modo que podemos definir los métodos en las estructuras con `impl`,
también podemos definir métodos en las enumeraciones. Aquí hay un método llamado `call`
que podríamos definir en nuestra enumeración `Message`:

```rust
# enum Message {
#     Quit,
#     Move { x: i32, y: i32 },
#     Write(String),
#     ChangeColor(i32, i32, i32),
# }
#
impl Message {
    fn call(&self) {
        // method body would be defined here
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

El cuerpo del método usaría `self` para obtener el valor que llamamos el método.
En este ejemplo, hemos creado una variable `m` que tiene el valor
`Message::Write(String::from("hello"))`, y eso es lo que `self` estará en el cuerpo
del `método call` cuando `m.call()` se ejecuta.

Veamos otra enumeración en la biblioteca estándar que es muy común y útil: `Option`.

### El *Enum* `Option` y sus ventajas sobre valores nulos

En la sección anterior, vimos cómo el enum `IpAddr` nos permite usar el sistema de
tipos de Rust para codificar más información que solo los datos en nuestro programa.
Esta sección explora un caso de estudio `Opción`, que es otra enumeración definida
por la biblioteca estándar. El tipo `Option` se usa en muchos lugares porque codifica
el escenario muy común en el que un valor podría ser algo o podría no ser nada.
Expresar este concepto en términos del sistema de tipos significa que el compilador
puede verificar si usted ha manejado todos los casos que debería manejar; esta
funcionalidad puede prevenir errores que son extremadamente comunes en otros
lenguajes de programación.

El diseño del lenguaje de programación a menudo se considera en términos de las
características que incluye, pero las características que excluye también son importantes.
Rust no tiene la característica *null* que tienen muchos otros lenguajes. *Null*  es un
valor que significa que no hay ningún valor allí. En lenguajes con *null*,
las variables siempre pueden estar en uno de dos estados: *null* o no *null* (nulo o no nulo).

En su presentación de 2009 “Null References: The Billion Dollar Mistake”,
Tony Hoare, el inventor de null, tiene esto que decir:

> Lo llamo mi error billonario. En ese momento, estaba diseñando el primer sistema de
> tipo completo para referencias en un lenguaje orientado a objetos. Mi objetivo era
> garantizar que todo el uso de las referencias sea absolutamente seguro, con una
> verificación realizada automáticamente por el compilador. Pero no pude resistir la
> tentación de poner una referencia nula (*null*), simplemente porque era muy fácil
> de implementar. Esto ha llevado a innumerables errores, vulnerabilidades y fallas en
> el sistema, que probablemente hayan causado mil millones de dolores y daños en los últimos
> cuarenta años.

El problema con los valores nulos es que si intenta usar un valor nulo como un valor no nulo,
obtendrá un error de algún tipo. Como esta propiedad nula o no nula es omnipresente, es
extremadamente fácil cometer este tipo de error.

Sin embargo, el concepto que *null* está tratando de expresar sigue siendo útil: un valor
nulo es actualmente inválido o no existe por algún motivo.

El problema no es realmente con el concepto sino con la implementación particular.
Como tal, Rust no tiene *null*, pero tiene una enumeración que puede codificar el
concepto de un valor presente o ausente. Esta enumeración es `Opción <T>`, y está
[definida por la biblioteca estándar][option] <!-- ignore --> de la siguiente manera:

[option]: ../../std/option/enum.Option.html

```rust
enum Option<T> {
    Some(T),
    None,
}
```

La enumeración `Option <T>` es tan útil que incluso está incluida en el *prelude*; no
es necesario que lo incluya explícitamente en el alcance. Además, también lo son sus
variantes: puede usar `Some` y `None` directamente sin el prefijo `Option::`. La enumeración
`Option <T>` sigue siendo solo una enumeración regular, y `Some(T)` y `None` siguen siendo
variantes de tipo `Option <T>`.

La sintaxis `<T>` es una característica de Rust de la que aún no hemos hablado. Es un parámetro
de tipo genérico, y trataremos los genéricos más detalladamente en el Capítulo 10. Por ahora,
todo lo que necesita saber es que `<T>` significa que la variante `Algunos` de la enumeración
`Option` puede contener una sola pieza de datos de cualquier tipo. Aquí hay algunos ejemplos del
uso de valores `Option` para contener tipos de números y tipos de *string*:

```rust
let some_number = Some(5);
let some_string = Some("a string");

let absent_number: Option<i32> = None;
```

Si usamos `None` en lugar de `Some`, debemos decirle a Rust qué tipo de `Option <T>`
tenemos, porque el compilador no puede inferir el tipo que la variante `Some`
se mantendrá mirando solo a un valor `None`.

Cuando tenemos un valor `Some`, sabemos que un valor está presente y el valor se
mantiene dentro del ` Some`. Cuando tenemos un valor `None`, en cierto sentido,
significa lo mismo que nulo: no tenemos un valor válido. Entonces, ¿por qué tener
`Option <T>` es mejor que tener nulo?

En resumen, debido a que `Option <T>` y `T` (donde `T` puede ser de cualquier tipo)
son diferentes tipos, el compilador no nos permitirá usar un valor `Option <T>` como
si fuera definitivamente un valor válido. Por ejemplo, este código no se compilará
porque está tratando de agregar un `i8` a una `Opción <i8>`:

```rust,ignore
let x: i8 = 5;
let y: Option<i8> = Some(5);

let sum = x + y;
```

Si ejecutamos este código, recibimos un mensaje de error como este:

```text
error[E0277]: the trait bound `i8: std::ops::Add<std::option::Option<i8>>` is
not satisfied
 -->
  |
5 |     let sum = x + y;
  |                 ^ no implementation for `i8 + std::option::Option<i8>`
  |
```

¡Intenso! En efecto, este mensaje de error significa que Rust no entiende cómo
agregar un `i8` y una `Opción <i8>`, porque son tipos diferentes. Cuando tenemos
un valor de un tipo como `i8` en Rust, el compilador se asegurará de que siempre tengamos
un valor válido. Podemos proceder con confianza sin tener que verificar el valor
nulo antes de usar ese valor. Solo cuando tenemos una `Opción <i8>`
(o el tipo de valor con el que estamos trabajando) tenemos que preocuparnos por
posiblemente no tener un valor, y el compilador se asegurará de que manejemos ese
caso antes de usar el valor.

En otras palabras, debes convertir una `Opción <T>` a una `T` antes de poder
realizar operaciones `T` con ella. Generalmente, esto ayuda a detectar uno de los
problemas más comunes con *null*: asumiendo que algo no es nulo cuando realmente es.

No tener que preocuparse por asumir incorrectamente un valor no nulo te ayuda a
tener más confianza en tu código. Para tener un valor que posiblemente sea nulo,
debe optar explícitamente haciendo que el tipo de ese valor sea `Opción <T>`. Luego,
cuando usa ese valor, se le requiere manejar explícitamente el caso cuando el valor es nulo.
En todas partes donde un valor tiene un tipo que no es una `Opción <T>`, *puede*
asumir con seguridad que el valor no es nulo. Esta fue una decisión deliberada
de diseño para que Rust limite la omnipresencia de *null* y aumente la seguridad
del código Rust.

Entonces, ¿cómo se obtiene el valor `T` de una variante `Some` cuando
tiene un valor de tipo `Opción <T>` para que pueda usar ese valor? La enumeración
`Option <T>` tiene una gran cantidad de métodos que son útiles en una variedad de
situaciones; puedes verlos en [su documentación][docs] <!-- ignore -->.
Familiarizarse con los métodos en `Option <T>` será extremadamente útil en su
viaje con Rust.

[docs]: ../../std/option/enum.Option.html

En general, para usar un valor `Option <T>`, desea tener un código que maneje cada
variante. Desea un código que se ejecutará solo cuando tenga un valor `Some (T)`,
y este código podrá usar el `T` interno. Desea que se ejecute algún otro código si
tiene un valor `None`, y ese código no tiene un valor `T` disponible. La expresión
`match` es una construcción de flujo de control que hace justamente esto cuando
se usa con *enums*: ejecutará un código diferente dependiendo de la variante de
la enumeración que tenga, y ese código puede usar los datos dentro del valor
coincidente.
