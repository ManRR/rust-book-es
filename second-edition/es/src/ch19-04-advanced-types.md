## Advanced Types

El sistema de tipo Rust tiene algunas características que hemos mencionado en
este libro pero que aún no hemos discutido. Comenzaremos analizando los tipos
nuevos en general al examinar por qué los tipos nuevos son útiles como tipos.
Luego pasaremos a escribir *alias*, una característica similar a *newtypes*
pero con semántica ligeramente diferente. También discutiremos el tipo `!` Y
los tipos de tamaño dinámico.

> Nota: La siguiente sección supone que ha leído la sección anterior  “*The Newtype*
> Patrón para implementar *traits* externos en tipos externos.”

### Uso del patrón *Newtype* para la seguridad y la abstracción de tipos

El patrón de tipo nuevo es útil para tareas más allá de las que hemos
discutido hasta ahora, incluida la aplicación estática de que los valores
nunca se confundan y que indiquen las unidades de un valor. Viste un ejemplo
del uso de *newtypes* para indicar unidades en el listado 19-23: recuerde que
las estructuras `Millimeters` y `Meters` envolvieron los valores `u32` en un
nuevo tipo. Si escribimos una función con un parámetro de tipo `Millimeters`,
no podríamos compilar un programa que accidentalmente intentó llamar a esa
función con un valor de tipo `Meters` o simplemente `u32`.

Otro uso del patrón *newtype* es abstraer algunos detalles de implementación
de un tipo: el nuevo tipo puede exponer una API pública que es diferente de
la API del tipo interno privado si usamos el nuevo tipo directamente para
restringir la funcionalidad disponible, para ejemplo.

*Newtypes* también puede ocultar la implementación interna. Por ejemplo,
podríamos proporcionar un tipo `People` para ajustar un
`HashMap<i32, String>` que almacena la identificación de una persona asociada
con su nombre. El código que utiliza `People` solo interactuaría con la API
pública que proporcionamos, como un método para agregar un *string* de nombre
a la colección `People`; ese código no necesitaría saber que asignamos una
identificación `i32` a los nombres internamente. El nuevo patrón de tipo es
una forma ligera de lograr la encapsulación para ocultar los detalles de
implementación, que discutimos en la sección “Encapsulación que oculta
detalles de implementación” del Capítulo 17.

### Creación de sinónimos de tipo con alias de tipo

Junto con el patrón de *newtype*, Rust proporciona la capacidad de declarar
un *type alias* para dar a un tipo existente otro nombre. Para esto, usamos
la palabra clave `type`. Por ejemplo, podemos crear el alias `Kilometers` a
`i32` de la siguiente manera:

```rust
type Kilometers = i32;
```

Ahora, el alias `Kilometers` es un *sinónimo* para `i32`; a diferencia de los
tipos `Millimeters` y `Meters` que creamos en el listado 19-23, `Kilometers`
no es un tipo nuevo e independiente. Los valores que tienen el tipo
`Kilometers` se tratarán de la misma manera que los valores de tipo `i32`:

```rust
type Kilometers = i32;

let x: i32 = 5;
let y: Kilometers = 5;

println!("x + y = {}", x + y);
```

Debido a que `Kilometers` y `i32` son del mismo tipo, podemos agregar valores
de ambos tipos y podemos pasar valores `Kilometers` a las funciones que toman
los parámetros `i32`. Sin embargo, al usar este método, no obtenemos los
beneficios de verificación de tipo que obtenemos del nuevo patrón de tipo
discutido anteriormente.

El principal caso de uso para los sinónimos de tipo es reducir la repetición.
Por ejemplo, podríamos tener un tipo largo como este:

```rust,ignore
Box<Fn() + Send + 'static>
```

Escribir este tipo largo en las firmas de función y como anotaciones de tipo
en todo el código puede ser molesto y propenso a errores. Imagine tener un
proyecto lleno de código como ese en el listado 19-32.

```rust
let f: Box<Fn() + Send + 'static> = Box::new(|| println!("hi"));

fn takes_long_type(f: Box<Fn() + Send + 'static>) {
    // --snip--
}

fn returns_long_type() -> Box<Fn() + Send + 'static> {
    // --snip--
#     Box::new(|| ())
}
```

<span class="caption">Listado 19-32: Usar un *tipo largo* (*long type*) en
muchos lugares</span>

Un tipo *alias* hace que este código sea más manejable al reducir la
repetición. En el listado 19-33, hemos introducido un *alias* llamado `Thunk`
para el tipo detallado y podemos reemplazar todos los usos del tipo con el
*alias* más corto `Thunk`.

```rust
type Thunk = Box<Fn() + Send + 'static>;

let f: Thunk = Box::new(|| println!("hi"));

fn takes_long_type(f: Thunk) {
    // --snip--
}

fn returns_long_type() -> Thunk {
    // --snip--
#     Box::new(|| ())
}
```

<span class="caption">Listado 19-33: Presentamos un tipo de *alias* `Thunk`
para reducir la repetición</span>

¡Este código es mucho más fácil de leer y escribir! Elegir un nombre
significativo para un tipo *alias* también puede ayudarlo a comunicar su
intención (*thunk* es una palabra para evaluar el código más adelante, por lo
que es un nombre apropiado para un *closure* que se almacena).

Los *alias* de tipo también se usan comúnmente con el tipo `Result <T, E>`
para reducir la repetición. Considere el módulo `std::io` en la biblioteca
estándar. Las operaciones de E/S a menudo devuelven un `Result<T, E>` para
manejar situaciones cuando las operaciones no funcionan. Esta biblioteca
tiene una estructura `std::io::Error` que representa todos los posibles
errores de E/S. Muchas de las funciones en `std::io` devolverán
`Result <T, E>` donde `'E` es `std::io::Error`, como estas funciones en el
*trait* `Write`:

```rust
use std::io::Error;
use std::fmt;

pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize, Error>;
    fn flush(&mut self) -> Result<(), Error>;

    fn write_all(&mut self, buf: &[u8]) -> Result<(), Error>;
    fn write_fmt(&mut self, fmt: fmt::Arguments) -> Result<(), Error>;
}
```

El `Result<..., Error>` se repite mucho. Como tal, `std::io` tiene este tipo
de declaración de *alias*:

```rust,ignore
type Result<T> = Result<T, std::io::Error>;
```

Como esta declaración está en el módulo `std::io`, podemos usar el *alias*
totalmente calificado `std::io::Result <T>`-es decir, un `Result <T, E>` con
el `E` rellenado como `std::io::Error`. Las firmas de funciones de *traits*
`Write` terminan pareciéndose a esto:

```rust,ignore
pub trait Write {
    fn write(&mut self, buf: &[u8]) -> Result<usize>;
    fn flush(&mut self) -> Result<()>;

    fn write_all(&mut self, buf: &[u8]) -> Result<()>;
    fn write_fmt(&mut self, fmt: Arguments) -> Result<()>;
}
```

El *alias* tipo ayuda de dos maneras: hace que el código sea más fácil de
escribir *y* nos da una interfaz consistente en todo el `std::io`. Como es un
*alias*, es simplemente otro `Result <T, E>`, lo que significa que podemos
usar cualquier método que funcione en `Result <T, E>` con él, así como una
sintaxis especial como el operador `?`.

### The Never Type that Never Returns

Rust tiene un tipo especial llamado `!` que se conoce en la jerga de teoría
de tipos como *tipo vacío* (*empty type*) porque no tiene valores. Preferimos
llamarlo *never type* porque se encuentra en el lugar del tipo de devolución
cuando una función nunca volverá. Aquí hay un ejemplo:

```rust,ignore
fn bar() -> ! {
    // --snip--
}
```

Este código se lee como “la función `bar` nunca retorna”. Las funciones que
retornan nunca se llaman *funciones divergentes*. No podemos crear valores
del tipo `!` para que `bar` nunca pueda regresar.

Pero, ¿para qué sirve un tipo para el que nunca se pueden crear valores?.
Recordar el código del listado 2-5; hemos reproducido una parte aquí en el
listado 19-34.

```rust
# let guess = "3";
# loop {
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};
# break;
# }
```

<span class="caption">Listado 19-34: Un `match` con un brazo que termina en
`continue`</span>

En ese momento, omitimos algunos detalles en este código. En el Capítulo 6 de
la sección “El operador de flujo de control `match`”, discutimos que los
brazos del `match` deben devolver el mismo tipo. Entonces, por ejemplo, el
siguiente código no funciona:

```rust,ignore
let guess = match guess.trim().parse() {
    Ok(_) => 5,
    Err(_) => "hello",
}
```

El tipo de `guess` en este código debería ser un entero *y* un *string*, y
Rust requiere que `guess` tenga solo un tipo. Entonces, ¿qué devuelve
`continue`?.Cómo se nos permitió devolver un `u32` de un brazo y tener otro
brazo que termina con `continue` en el Listado 19-34?

Como habrás adivinado, `continue` tiene un valor `!`. Es decir, cuando Rust
calcula el tipo de `guess`, mira ambos brazos del *match*, el primero con un
valor de `u32` y el último con un valor `!`. Debido a que `!` nunca puede
tener un valor, Rust decide que el tipo de `guess` es `u32`.

La forma formal de describir este comportamiento es que las expresiones de
tipo `!` pueden forzarse en cualquier otro tipo. Podemos terminar este brazo
`match` con `continue` porque `continue` no devuelve un valor; en su lugar,
mueve el control de nuevo a la parte superior del ciclo, por lo que en el
caso de `Err`, nunca asignamos un valor a `guess`.

El tipo nunca es útil con la macro `panic!` También. ¿Recuerda la función
`unwrap` que llamamos a los valores `Option<T>` para producir un valor o
pánico?. Aquí está su definición:

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap(self) -> T {
        match self {
            Some(val) => val,
            None => panic!("called `Option::unwrap()` on a `None` value"),
        }
    }
}
```

En este código, sucede lo mismo que en el `match` en el Listado 19-34: Rust
ve que `val` tiene el tipo `T` y `panic!` tiene el tipo `!`, por lo que el
resultado de la expresión general de `match` es `T`. Este código funciona
porque `panic!` No produce un valor; termina el programa. En el caso `None`,
no devolveremos un valor de `unwrap`, por lo que este código es válido.

Una expresión final que tiene el tipo `!` es un `loop`:

```rust,ignore
print!("forever ");

loop {
    print!("and ever ");
}
```

Aquí, el ciclo nunca termina, entonces `!` es el valor de la expresión. Sin
embargo, esto no sería cierto si incluyéramos un `break`, porque el ciclo
terminaría cuando llegara al `break`.

### Tipos dinámicamente dimensionados y el *Trait* `Sized`

Debido a la necesidad de Rust de conocer ciertos detalles, como la cantidad
de espacio para asignar un valor de un tipo particular, hay una esquina de su
sistema de tipo que puede ser confusa: el concepto de *tipos de tamaño
dinámico*. A veces denominados *DSTs* o *unsized types*, estos tipos nos
permiten escribir código utilizando valores cuyo tamaño solo podemos conocer
en tiempo de ejecución.

Vamos a profundizar en los detalles de un tipo de tamaño dinámico llamado
`str`, que hemos estado utilizando a lo largo del libro. Así es, no `&str`,
sino `str` en sí mismo, es un DST. No podemos saber cuánto tiempo dura la
*string* hasta el tiempo de ejecución, lo que significa que no podemos crear
una variable de tipo `str`, ni podemos tomar un argumento de tipo `str`.
Considere el siguiente código, que no funciona:

```rust,ignore
let s1: str = "Hello there!";
let s2: str = "How's it going?";
```

Rust necesita saber cuánta memoria asignar para cualquier valor de un tipo
particular, y todos los valores de un tipo deben usar la misma cantidad de
memoria. Si Rust nos permitiera escribir este código, estos dos valores `str`
necesitarían ocupar la misma cantidad de espacio. Pero tienen diferentes
longitudes: `s1` necesita 12 bytes de almacenamiento y `s2` necesita 15. Es
por eso que no es posible crear una variable que tenga un tipo de tamaño
dinámico.

¿Asi que que hacemos?. En este caso, ya conoce la respuesta: hacemos los
tipos de `s1` y `s2` a `&str` en lugar de `str`. Recuerde que en la sección
String Slices” del Capítulo 4, dijimos que la estructura de datos de sectores
almacena la posición inicial y la longitud del sector.

Entonces, aunque un `&T` es un valor único que almacena la dirección de
memoria de donde el `T` se encuentra, un `&str` es *dos* valores: la
dirección del `str` y su longitud. Como tal, podemos conocer el tamaño de un
valor `&str` en tiempo de compilación: es el doble de la longitud de un
`usize`. Es decir, siempre sabemos el tamaño de un `&str`, no importa cuánto
tiempo se refiere al *string* a la que se refiere. En general, este es el
camino en qué tipos de tamaño dinámico se usan en Rust: tienen un poco más de
metadatos que almacena el tamaño de la información dinámica. La regla de oro
de tipos de tamaño dinámico es que siempre debemos poner valores de tamaño
dinámico tipos detrás de un puntero de algún tipo.

Podemos combinar `str` con todo tipo de punteros: por ejemplo,`Box<str>` o
`Rc<str>`. De hecho, ya has visto esto antes, pero con una dinámica diferente
tipo de tamaño: *traits*. Cada característica es un tipo de tamaño dinámico
al que podemos hacer referencia usando el nombre del *trait*. En el Capítulo
17 en la sección “Uso de *Trait Objects* que permiten valores de diferentes
tipos” mencionamos que para usar los *traits* como *trait objects*, debemos
ponerlos detrás de un puntero, como `&Trait` o `Box<Trait>`
(`Rc<Trait>` también funcionaría).

Para trabajar con los DST, Rust tiene un *trait* particular llamado el
*trait* `Sized` para determinar si se conoce o no el tamaño de un tipo en el
momento de la compilación. Este *trait* es implementado automáticamente para
todo cuyo tamaño se conoce en tiempo de compilación. Además, Rust agrega
implícitamente un límite en `Sized` a cada función genérica.
Es decir, una definición de función genérica como esta:

```rust,ignore
fn generic<T>(t: T) {
    // --snip--
}
```

en realidad se trata como si hubiéramos escrito esto:

```rust,ignore
fn generic<T: Sized>(t: T) {
    // --snip--
}
```

Por defecto, las funciones genéricas solo funcionarán en los tipos que tienen
un tamaño conocido en el momento de la compilación. Sin embargo, puede usar
la siguiente sintaxis especial para relajar esta restricción:

```rust,ignore
fn generic<T: ?Sized>(t: &T) {
    // --snip--
}
```

Un *trait* ligado en `?Sized` es el opuesto de un *trait* ligado en `Sized`:
leemos esto como “`T` puede o no ser `Sized`”.Esta sintaxis solo está
disponible para `Sized`, no cualquier otro *trait*.

También tenga en cuenta que cambiamos el tipo del parámetro `t` de `T` a
`&T`. Debido a que el tipo puede no ser `Sized`, tenemos que usarlo detrás de
algún tipo de puntero. En este caso, hemos elegido una referencia.

¡A continuación, hablaremos sobre funciones y closures!
