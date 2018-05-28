## Convirtiendo nuestro servidor de un *Single-Threaded* en un servidor Multithreaded

En este momento, el servidor procesará cada solicitud por turno, lo que
significa que no procesará una segunda conexión hasta que la primera termine
de procesarse. Si el servidor recibió más y más solicitudes, esta ejecución
en serie sería cada vez menos óptima. Si el servidor recibe una solicitud
que tarda mucho tiempo en procesarse, las solicitudes posteriores deberán
esperar hasta que finalice la solicitud larga, incluso si las nuevas
solicitudes se pueden procesar rápidamente. Tendremos que arreglar esto,
pero primero, veremos el problema en acción.

### Simular una solicitud lenta en la implementación del servidor actual

Veremos cómo una solicitud de procesamiento lento puede afectar otras
solicitudes realizadas a nuestra implementación actual del servidor. El
listado 20-10 implementa el manejo de una solicitud */sleep* con una
respuesta lenta simulada que hará que el servidor duerma durante 5 segundos
antes de responder.

<span class="filename">Filename: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;
# use std::io::prelude::*;
# use std::net::TcpStream;
# use std::fs::File;
// --snip--

fn handle_connection(mut stream: TcpStream) {
#     let mut buffer = [0; 512];
#     stream.read(&mut buffer).unwrap();
    // --snip--

    let get = b"GET / HTTP/1.1\r\n";
    let sleep = b"GET /sleep HTTP/1.1\r\n";

    let (status_line, filename) = if buffer.starts_with(get) {
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else if buffer.starts_with(sleep) {
        thread::sleep(Duration::from_secs(5));
        ("HTTP/1.1 200 OK\r\n\r\n", "hello.html")
    } else {
        ("HTTP/1.1 404 NOT FOUND\r\n\r\n", "404.html")
    };

    // --snip--
}
```

<span class="caption">Listado 20-10: simulando una solicitud lenta
reconociendo */sleep* y durmiendo durante 5 segundos</span>

Este código es un poco complicado, pero es lo suficientemente bueno para
fines de simulación. Creamos una segunda solicitud `sleep`, cuyos datos
reconoce nuestro servidor. Añadimos un `else if` después del bloque `if`
para verificar la solicitud a */sleep*. Cuando se recibe esa solicitud, el
servidor duerme durante 5 segundos antes de mostrar la página HTML correcta.

Puedes ver cuán primitivo es nuestro servidor: ¡las bibliotecas reales
manejarían el reconocimiento de múltiples solicitudes de una manera mucho
menos detallada!

Inicie el servidor usando `cargo run`. A continuación, abra dos ventanas del
navegador: una para *http://127.0.0.1:7878/* y la otra para
*http://127.0.0.1:7878/sleep*. Si ingresa el */* URI varias veces, como
antes, verá que responde rápidamente. Pero si ingresa */sleep* y luego carga
*/*, verá que */* espera hasta que `sleep` haya dormido durante 5 segundos
antes de cargar.

Hay varias formas en que podemos cambiar la forma en que funciona nuestro
servidor web para evitar tener más solicitudes de respaldo detrás de una
solicitud lenta; el que implementaremos es un *thread pool*.

### mejorar el rendimiento con un *Thread Pool*

Un *thread pool* es un *pool* de hilos generados que están esperando y listos
para manejar una tarea. Cuando el programa recibe una nueva tarea, asigna
uno de los hilos en el *pool* a la tarea, y ese hilo procesará la tarea. los
los hilos restantes en el *pool* están disponibles para manejar cualquier
otra tarea que se presente mientras el primer hilo está procesando. Cuando
se termina el primer hilo procesando su tarea, se devuelve al *thread pool*
inactivos, listo para manejar una nueva tarea. Un *thread pool* le permite
procesar conexiones al mismo tiempo, aumentando el rendimiento de su
servidor.

Limitaremos el número de *threads* en el *pool* a un número pequeño para
protegernos de ataques de denegación de servicio (DoS); si tuviéramos
nuestro programa, cree un nuevo hilo para cada solicitud que entró, alguien
haciendo 10 millones de solicitudes a nuestro servidor podría crear estragos
al detener todos los recursos de nuestro servidor y detener el procesamiento
de las solicitudes.

En lugar de engendrar hilos ilimitados, tendremos un número fijo de hilos
esperando en el *pool*. A medida que llegan las solicitudes, se enviarán al
*pool* para procedimiento. El grupo mantendrá una cola de solicitudes
entrantes. Cada una de las los *threads* del *pool* extraerán una solicitud
de esta cola, manejarán la solicitud, y luego pedirle a la cola otra
solicitud. Con este diseño, podemos procesar solicitudes `N`
concurrentemente, donde `N` es la cantidad de hilos. Si cada *thread*
responde a una solicitud de larga ejecución, las solicitudes posteriores aún
pueden realizar una copia de seguridad en la cola, pero hemos aumentado la
cantidad de solicitudes de larga ejecución que podemos manejar antes de
llegar a ese punto.

Esta técnica es solo una de las muchas formas de mejorar el rendimiento de
un servidor web. Otras opciones que puede explorar son el modelo de
*fork/join* y el modelo de E/S asíncronas de un solo hilo. Si le interes
este tema, puede leer más sobre otras soluciones e intentar implementarlas
en Rust; con un lenguaje de bajo nivel como Rust, todas estas opciones son
posibles.

Antes de comenzar a implementar un *thread pool*, hablemos sobre cómo
debería ser el uso del *pool*. Cuando intenta diseñar código, escribir
primero la interfaz del cliente puede ayudar a guiar su diseño. Escriba la
API del código para que esté estructurado de la manera que desea llamarlo;
luego implemente la funcionalidad dentro de esa estructura en lugar de
implementar la funcionalidad y luego diseñar la API pública.

De manera similar a como utilizamos el desarrollo basado en pruebas en el
proyecto en el Capítulo 12, usaremos el *compiler-driven development* aquí.
Escribiremos el código que llama a las funciones que queremos, y luego
veremos los errores del compilador para determinar qué debemos cambiar luego
para que el código funcione.

#### Estructura del código si pudiéramos *Spawn* un hilo para cada solicitud

Primero, exploremos cómo podría verse nuestro código si creara un nuevo hilo
para cada conexión. Como se mencionó anteriormente, este no es nuestro plan
final debido a los problemas con el potencial de generar un número ilimitado
de hilos, pero es un punto de partida. El listado 20-11 muestra los cambios
que se realizarán en `main` para *spawning* un nuevo hilo para manejar cada
flujo dentro del bucle `for`.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
# use std::thread;
# use std::io::prelude::*;
# use std::net::TcpListener;
# use std::net::TcpStream;
#
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        thread::spawn(|| {
            handle_connection(stream);
        });
    }
}
# fn handle_connection(mut stream: TcpStream) {}
```

<span class="caption">Listado 20-11: Generando un nuevo hilo para cada
flujo</span>

Como aprendió en el Capítulo 16, `thread::spawn` creará un nuevo hilo y
luego ejecutará el código en el *closure* del nuevo hilo. Si ejecuta este
código y carga */sleep* en su navegador, entonces */* en dos pestañas más
del navegador, verá que las solicitudes a */* no tienen que esperar
*/sleep* para finalizar. Pero como mencionamos, esto eventualmente abrumará
al sistema porque estarías creando nuevos hilos sin límite.

#### Crear una interfaz similar para un número finito de hilos

Queremos que nuestro *thread pool* funcione de forma similar y familiar, por
lo que cambiar de *thread* a un *thread pool* no requiere grandes cambios en
el código que usa nuestra API. El listado 20-12 muestra la interfaz
hipotética para una estructura `ThreadPool` que queremos usar en lugar de
`thread::spawn`.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
# use std::thread;
# use std::io::prelude::*;
# use std::net::TcpListener;
# use std::net::TcpStream;
# struct ThreadPool;
# impl ThreadPool {
#    fn new(size: u32) -> ThreadPool { ThreadPool }
#    fn execute<F>(&self, f: F)
#        where F: FnOnce() + Send + 'static {}
# }
#
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }
}
# fn handle_connection(mut stream: TcpStream) {}
```

<span class="caption">Listado 20-12: nuestra interfaz ideal
`ThreadPool`</span>

Usamos `ThreadPool::new` para crear un nuevo *thread pool* con un número
configurable de *threads*, en este cuatro caso. Luego, en el bucle `for`,
`pool.execute` tiene una interfaz similar a `thread::spawn`, ya que requiere
un *closure* para que el pool se ejecute para cada stream. Necesitamos
implementar `pool.execute` para que se lleve a cabo el *closure* y se lo
entregue a un hilo en el *pool* para que se ejecute. Este código aún no se
compilará, pero lo intentaremos para que el compilador pueda guiarnos en
cómo solucionarlo.

#### Construyendo la estructura `ThreadPool` usando *Compiler Driven Development*

Realice los cambios en el listado 20-12 a *src/main.rs*, y luego usemos los
errores del compilador de `cargo check` para impulsar nuestro desarrollo.
Este es el primer error que obtenemos:

```text
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
error[E0433]: failed to resolve. Use of undeclared type or module `ThreadPool`
  --> src\main.rs:10:16
   |
10 |     let pool = ThreadPool::new(4);
   |                ^^^^^^^^^^^^^^^ Use of undeclared type or module
   `ThreadPool`

error: aborting due to previous error
```

¡Estupendo! Este error nos dice que necesitamos un tipo o módulo
`ThreadPool`, así que crearemos uno ahora. Nuestra implementación
`ThreadPool` será independiente del tipo de trabajo que nuestro servidor web
está haciendo. Entonces, cambiemos el *crate* `hello` de un *binary crate* a
un *library crate* para mantener nuestra implementación `ThreadPool`.
Después de cambiar a un *library crate*, también podríamos usar la
biblioteca del *thread pool* por separado para cualquier trabajo que
deseemos con un *thread pool*, no solo para servir solicitudes web.

Cree un *src/lib.rs* que contenga lo siguiente, que es la definición más
simple de una estructura `ThreadPool` que podemos tener por ahora:

<span class="filename">Filename: src/lib.rs</span>

```rust
pub struct ThreadPool;
```

A continuación, cree un nuevo directorio, *src/bin*, y mueva el *binary
crate* enraizada en *src/main.rs* en *src/bin/main.rs*. Hacerlo hará que el
*library crate* cargue el *crate* primario en el directorio *hello*; aún
podemos ejecutar el binario en *src/bin/main.rs* usando `cargo run`. Después
de mover el archivo *main.rs*, edítelo para que aparezca el *library crate*
y coloque `ThreadPool` en el alcance agregando el siguiente código a la
parte superior de *src/bin/main.rs*:

<span class="filename">Filename: src/bin/main.rs</span>

```rust,ignore
extern crate hello;
use hello::ThreadPool;
```

Este código aún no funcionará, pero revisemoslo nuevamente para obtener el
próximo error que necesitamos abordar:

```text
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
error[E0599]: no function or associated item named `new` found for type
`hello::ThreadPool` in the current scope
 --> src/bin/main.rs:13:16
   |
13 |     let pool = ThreadPool::new(4);
   |                ^^^^^^^^^^^^^^^ function or associated item not found in
   `hello::ThreadPool`
```

Este error indica que a continuación debemos crear una función asociada
denominada `new` para `ThreadPool`. También sabemos que `new` necesita tener
un parámetro que pueda aceptar `4` como argumento y debe devolver una
instancia `ThreadPool`. Implementemos la función `new` más simple que tendrá
esas características:

<span class="filename">Filename: src/lib.rs</span>

```rust
pub struct ThreadPool;

impl ThreadPool {
    pub fn new(size: usize) -> ThreadPool {
        ThreadPool
    }
}
```

Elegimos `usize` como el tipo del parámetro `size`, porque sabemos que un
número negativo de hilos no tiene ningún sentido. También sabemos que
usaremos este 4 como el número de elementos en una colección de hilos, que
es para lo que el tipo `usize` es, como se discutió en la sección “Tipos de
enteros” del Capítulo 3.

Revisemos el código nuevamente:

```text
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
warning: unused variable: `size`
 --> src/lib.rs:4:16
  |
4 |     pub fn new(size: usize) -> ThreadPool {
  |                ^^^^
  |
  = note: #[warn(unused_variables)] on by default
  = note: to avoid this warning, consider using `_size` instead

error[E0599]: no method named `execute` found for type `hello::ThreadPool` in the current scope
  --> src/bin/main.rs:18:14
   |
18 |         pool.execute(|| {
   |              ^^^^^^^
```

Ahora recibimos una advertencia y un error. Ignorando la advertencia por un
momento, el error ocurre porque no tenemos un método `execute` en
`ThreadPool`. Recuerde en la sección “Crear una interfaz similar para un
número finito de *Threads*” que decidimos que nuestro *thread pool* debería
tener una interfaz similar a `thread::spawn`. Además, implementaremos la
función `execute` para que tome el *closure* que se le da y lo entregue a un
hilo inactivo en el *pool* para ejecutar.

Definiremos el método `execute` en `ThreadPool` para tomar un *closure* como
parámetro. Recuerde en la sección “Almacenamiento de *closures* con
parámetros genéricos y *traits* de Fn” en el Capítulo 13 que podemos tomar
*closures* como parámetros con tres características diferentes: `Fn`,
`FnMut` y `FnOnce`. Tenemos que decidir qué tipo de *closure* usar aquí.
Sabemos que terminaremos haciendo algo similar a la implementación de la
biblioteca estándar `thread::spawn`, para que podamos ver qué límites tiene
la firma de `thread::spawn` en su parámetro. La documentación nos muestra lo
siguiente:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T + Send + 'static,
        T: Send + 'static
```

El parámetro de tipo `F` es el que nos interesa aquí; el parámetro de tipo
`T` está relacionado con el valor de retorno, y no estamos preocupados por
eso. Podemos ver que `spawn` usa `FnOnce` como el *trait bound* a `F`. Esto
es probablemente lo que queremos también, porque eventualmente pasaremos el
argumento que obtenemos en `execute` a `spawn`. Podemos estar más seguros de
que `FnOnce` es el *trait* que queremos usar, ya que el hilo para ejecutar
una solicitud solo ejecutará el *closure* de esa solicitud una vez, que
coincide con `Once` en `FnOnce`.

El parámetro de tipo `F` también tiene el *trait bound* `Send` y el
*lifetime bound* `'static`, que son útiles en nuestra situación:
necesitamos `Send` para transferir el *closure* de un hilo a otro y
`'static` porque no sabemos cuánto tardará el hilo en ejecutarse. Vamos a
crear un método `execute` en `ThreadPool` que tomará un parámetro genérico
de tipo `F` con estos límites:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub struct ThreadPool;
impl ThreadPool {
    // --snip--

    pub fn execute<F>(&self, f: F)
        where
            F: FnOnce() + Send + 'static
    {

    }
}
```

Todavía usamos el `()` después de `FnOnce` porque este `FnOnce` representa
un *closure* que no toma parámetros y no devuelve un valor. Al igual que las
definiciones de función, el tipo de devolución puede omitirse de la firma,
pero incluso si no tenemos parámetros, aún necesitamos los paréntesis.

De nuevo, esta es la implementación más simple del método `execute`: no hace
nada, pero solo intentamos compilar nuestro código. Veámoslo de nuevo:

```text
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
warning: unused variable: `size`
 --> src/lib.rs:4:16
  |
4 |     pub fn new(size: usize) -> ThreadPool {
  |                ^^^^
  |
  = note: #[warn(unused_variables)] on by default
  = note: to avoid this warning, consider using `_size` instead

warning: unused variable: `f`
 --> src/lib.rs:8:30
  |
8 |     pub fn execute<F>(&self, f: F)
  |                              ^
  |
  = note: to avoid this warning, consider using `_f` instead
```

Estamos recibiendo solo advertencias ahora, ¡lo que significa que compila!
Pero tenga en cuenta que si prueba `cargo run` y realiza una solicitud en el
navegador, verá los errores en el navegador que vimos al principio del
capítulo. ¡Nuestra biblioteca no está llamando al *closure* pasado a `ejecutar` todavía!

> Nota: Un dicho que usted puede escuchar sobre los leenguajes con
> compiladores estrictos, como Haskell y Rust, es “si el código se compila,
> funciona”. Pero este dicho no es universalmente cierto. Nuestro proyecto se
> compila, ¡pero no hace absolutamente nada! Si estuviéramos construyendo un
> proyecto real y completo, este sería un buen momento para comenzar a
> escribir pruebas unitarias para verificar que el código compila *y* tenga
> el comportamiento que queremos.

#### Validar el número de *Threads* en `new`

Seguiremos recibiendo advertencias porque no estamos haciendo nada con los
parámetros para `new` y `execute`. Implementemos los cuerpos de estas
funciones con el comportamiento que queremos. Para empezar, pensemos en
`new`. Anteriormente elegimos un tipo sin signo para el parámetro `size`,
porque un *pool* con un número negativo de *threads* no tiene sentido. Sin
embargo, un *pool* con cero *threads* tampoco tiene sentido, pero cero es un
`usize` perfectamente válido. Agregaremos código para verificar que `size`
sea mayor que cero antes de devolver una instancia `ThreadPool` y pánico al
programa si recibe un cero usando la macro `assert!`, Como se muestra en el
Listado 20-13.

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub struct ThreadPool;
impl ThreadPool {
    /// Create a new ThreadPool.
    ///
    /// The size is the number of threads in the pool.
    ///
    /// # Panics
    ///
    /// The `new` function will panic if the size is zero.
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        ThreadPool
    }

    // --snip--
}
```

<span class="caption">Listado 20-13: Implementando `ThreadPool::new` para
entrar en pánico si `size` es cero</span>

Hemos agregado documentación para nuestro `ThreadPool` con comentarios de
doc. Tenga en cuenta que seguimos buenas prácticas de documentación al
agregar una sección que indica las situaciones en las que nuestra función
puede entrar en pánico, como se discutió en el Capítulo 14. Pruebe ejecutar
`cargo doc --open` y haga clic en la estructura `ThreadPool` para ver lo que
generó documentos para `new`!

En lugar de agregar la macro `assert!` como hemos hecho aquí, podríamos
hacer que `new` devolviera un `Result` como lo hicimos con `Config::new` en
el proyecto de E/S del Listado 12-9. Pero hemos decidido en este caso que
intentar crear un *thread pool* sin ningún *threads* debería ser un error
irrecuperable. Si te sientes ambicioso, intenta escribir una versión de
`new` con la siguiente firma para comparar ambas versiones:

```rust,ignore
pub fn new(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### Crear espacio para almacenar los *Threads*

Ahora que tenemos una manera de saber que tenemos un número válido de hilos
para almacenar en el *pool*, podemos crear esos hilos y almacenarlos en la
estructura `ThreadPool` antes de devolverlos. Pero, ¿cómo “almacenamos” un
hilo? Echemos otro vistazo a la firma `thread::spawn`:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T + Send + 'static,
        T: Send + 'static
```

La función `spawn` devuelve `JoinHandle<T>`, donde `T` es el tipo que
devuelve el *closure*. Tratemos de usar `JoinHandle` también y veamos qué
pasa. En nuestro caso, los *closures* que estamos pasando al *thread pool*
qur manejarán la conexión y no devolverán nada, por lo que `T` será el tipo
de unidad `()`.

El código en el listado 20-14 se compilará pero aún no crea *threads*. Hemos
cambiado la definición de `ThreadPool` para contener un vector de
instancias `thread::JoinHandle<()>`, inicializamos el vector con una
capacidad de `size`, configuramos un bucle `for` que ejecutará algún código
para crear los hilos, y devolvió una instancia `ThreadPool` que los contiene.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
use std::thread;

pub struct ThreadPool {
    threads: Vec<thread::JoinHandle<()>>,
}

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut threads = Vec::with_capacity(size);

        for _ in 0..size {
            // create some threads and store them in the vector
        }

        ThreadPool {
            threads
        }
    }

    // --snip--
}
```

<span class="caption">Listado 20-14: Creando un vector para `ThreadPool`
para contener los hilos</span>

Hemos introducido `std::thread` en el alcance del *library crate*, porque
estamos usando `thread::JoinHandle` como el tipo de elementos en el vector
en `ThreadPool`.

Una vez que se recibe un tamaño válido, nuestro `ThreadPool` crea un nuevo
vector que puede contener elementos `size`. Todavía no hemos utilizado la
función `with_capacity` en este libro, que realiza la misma tarea que
`Vec::new` pero con una diferencia importante: asigna espacio previamente en
el vector. Debido a que sabemos que necesitamos almacenar elementos `size`
en el vector, hacer esta asignación por adelantado es ligeramente más
eficiente que usar `Vec::new`, que se redimensiona a medida que se insertan
los elementos.

Cuando ejecute `cargo check` nuevamente, obtendrá algunas advertencias más,
pero debería tener éxito.

#### Una `Worker` responsable de Enviar el Código desde 'ThreadPool' a un *Thread*

Dejamos un comentario en el bucle `for` en el Listado 20-14 con respecto a
la creación de *threads*. Aquí, veremos cómo realmente creamos hilos. La
biblioteca estándar proporciona `thread::spawn` como una forma de crear
*thread*, y `thread::spawn` espera obtener algún código para que el
*thread* se ejecute tan pronto como se cree el *thread*. Sin embargo, en
nuestro caso, queremos crear los hilos y hacer que *esperen* el código que
enviaremos más tarde. La implementación de hilos de la biblioteca estándar
no incluye ninguna forma de hacerlo; tenemos que implementarlo manualmente.

Implementaremos este comportamiento introduciendo una nueva estructura de
datos entre `ThreadPool` y los hilos que administrarán este nuevo
comportamiento. Llamaremos esta estructura de datos `Worker`, que es un
término común en las implementaciones de *pooling*. Piensa en las personas
que trabajan en la cocina de un restaurante: los trabajadores esperan hasta
que los pedidos lleguen de los clientes, y luego son responsables por tomar
esos pedidos y llenarlos.

En lugar de almacenar un vector de instancias `JoinHandle<()>` en el *thread
pool*, almacenaremos instancias de la estructura `Worker`. Cada
`Worker` almacenará una sola instancia `JoinHandle<()>`. Luego
implementaremos un método en `Worker` que tomar un *closure* de código para
ejecutar y enviarlo al hilo que ya se ejecuta para ejecución. También le
daremos a cada trabajador un `id` para que podamos distinguir entre
los diferentes trabajadores en el *pool* al iniciar sesión o depurar.

Hagamos los siguientes cambios a lo que sucede cuando creamos un
`ThreadPool`. Implementaremos el código que envía el *closure* al hilo
después de que tengamos `Worker` configurado de esta manera:

1. Defina una estructura `Worker` que contenga un `id` y un `JoinHandle<()>`.
2. Cambie `ThreadPool` para contener un vector de instancias `Worker`.
3. Defina una función `Worker::new` que toma un número `id` y devuelve una
 instancia `Worker` que contiene el `id` y un hilo *spawned* con un
 *closure* vacío.
4. En `ThreadPool::new`, use el contador de bucles `for` para generar un
 `id`, cree un nuevo `Worker` con ese `id`, y almacene el `Worker` en el
  vector.

Si está preparado para un desafío, intente implementar estos cambios por su
cuenta antes de mirar el código en el Listado 20-15.

¿Listo?. Aquí está el Listado 20-15 con una forma de hacer las modificaciones
precedentes.

<span class="filename">Filename: src/lib.rs</span>

```rust
use std::thread;

pub struct ThreadPool {
    workers: Vec<Worker>,
}

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id));
        }

        ThreadPool {
            workers
        }
    }
    // --snip--
}

struct Worker {
    id: usize,
    thread: thread::JoinHandle<()>,
}

impl Worker {
    fn new(id: usize) -> Worker {
        let thread = thread::spawn(|| {});

        Worker {
            id,
            thread,
        }
    }
}
```

<span class="caption">Listado 20-15: Modificación de `ThreadPool` para
contener instancias `Worker` en lugar de contener hilos directamente</span>

Hemos cambiado el nombre del campo en `ThreadPool` de` threads` a `workers`
porque ahora contiene instancias `Worker` en lugar de instancias
`JoinHandle<()>`. Usamos el contador en el bucle `for` como argumento para
`Worker::new`, y almacenamos cada `Worker` nuevo en el vector llamado
`workers`.

El código externo (como nuestro servidor en *src/bin/main.rs*) no necesita
conocer los detalles de implementación con respecto al uso de una estructura
`Worker` dentro de `ThreadPool`, por lo que hacemos la estructura `Worker` y
su función privada `new`. La función `Worker::new` usa el `id` que le damos
y almacena una instancia `JoinHandle<()>` que se crea al generar un nuevo
hilo usando un *closure* vacío.

Este código compilará y almacenará el número de instancias `Worker` que
especificamos como argumento para `ThreadPool::new`. Pero *todavía* no
estamos procesando el *closure* que obtenemos en `execute`. Veamos cómo
hacer eso a continuación.

#### Envío de solicitudes a hilos por canales

Ahora abordaremos el problema de que los *closures* dados a
`thread::spawn` no hacen absolutamente nada. Actualmente, obtenemos el
*closure* que queremos ejecutar en el método `execute`. Pero tenemos que
darle a `thread::spawn` un *closure* para que se ejecute cuando creamos
cada `Worker` durante la creación de `ThreadPool`.

Queremos que las estructuras `Worker` que acabamos de crear busquen código
para ejecutar desde una *cola* (*queue*) contenida en `ThreadPool` y envíen
ese código a su hilo para que se ejecute.

En el Capítulo 16, aprendió sobre *canales*, una forma sencilla de
comunicarse entre dos hilos, que sería perfecto para este caso de uso.
Usaremos un canal para funcionar como la cola de trabajos, y `execute`
enviará un trabajo del `ThreadPool` a las instancias `Worker`, que enviará
el trabajo a su hilo. Este es el plan:

1. El `ThreadPool` creará un canal y se mantendrá en el lado de envío del
 canal.
2. Cada `Worker` se mantendrá en el lado receptor del canal.
3. Crearemos una nueva estructura `Job` que contendrá los *closures* que
 queremos enviar al canal.
4. El método `execute` enviará el trabajo que quiere ejecutar por el lado de
 envío del canal.
5. En su hilo, el `Worker` recorrerá su lado de recepción del canal y
 ejecutará los *closures* de cualquier trabajo que reciba.

Comencemos creando un canal en `ThreadPool::new` y manteniendo el lado
emisor en la instancia `ThreadPool`, como se muestra en el Listado 20-16. La
estructura `Job` no contiene nada por ahora, pero será el tipo de elemento
que estamos enviando por el canal.

<span class="filename">Filename: src/lib.rs</span>

```rust
# use std::thread;
// --snip--
use std::sync::mpsc;

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Job>,
}

struct Job;

impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id));
        }

        ThreadPool {
            workers,
            sender,
        }
    }
    // --snip--
}
#
# struct Worker {
#     id: usize,
#     thread: thread::JoinHandle<()>,
# }
#
# impl Worker {
#     fn new(id: usize) -> Worker {
#         let thread = thread::spawn(|| {});
#
#         Worker {
#             id,
#             thread,
#         }
#     }
# }
```

<span class="caption">Listado 20-16: Modificando `ThreadPool` para almacenar
el final de envío de un canal que envía instancias `Job`</span>

En `ThreadPool::new`, creamos nuestro nuevo canal y hacemos que el *pool*
contenga el extremo que envía. Esto compilará con éxito, aún con
advertencias.

Intentemos pasar un extremo de recepción del canal a cada *worker* a medida
que el *thread pool* crea el canal. Sabemos que queremos utilizar el extremo
receptor en el hilo que crea los *workers*, por lo que haremos referencia al
parámetro `receptor` en el *closure*. El código en el listado 20-17 aún no
se compilará del todo.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, receiver));
        }

        ThreadPool {
            workers,
            sender,
        }
    }
    // --snip--
}

// --snip--

impl Worker {
    fn new(id: usize, receiver: mpsc::Receiver<Job>) -> Worker {
        let thread = thread::spawn(|| {
            receiver;
        });

        Worker {
            id,
            thread,
        }
    }
}
```

<span class="caption">Listado 20-17: Pasar el extremo receptor del canal a
los *workers*</span>

Hemos realizado algunos cambios pequeños y directos: pasamos el extremo
receptor del canal a `Worker::new`, y luego lo usamos dentro del *closure*.

Cuando tratamos de verificar este código, obtenemos este error:

```text
$ cargo check
   Compiling hello v0.1.0 (file:///projects/hello)
error[E0382]: use of moved value: `receiver`
  --> src/lib.rs:27:42
   |
27 |             workers.push(Worker::new(id, receiver));
   |                                          ^^^^^^^^ value moved here in
   previous iteration of loop
   |
   = note: move occurs because `receiver` has type
   `std::sync::mpsc::Receiver<Job>`, which does not implement the `Copy` trait
```

El código está tratando de pasar `receiver` a múltiples instancias
`Worker`. Esto no funcionará, como recordará en el Capítulo 16: la
implementación del canal que proporciona Rust es múltiple *productor*, único
*consumidor*. Esto significa que no podemos simplemente clonar el extremo
consumidor del canal para arreglar este código. Incluso si pudiéramos, esa
no es la técnica que quisiéramos usar; en su lugar, queremos distribuir los
*jobs* a través de los hilos al compartir el único `receiver` entre todos
los *workers*.

Además, quitar un *job* (*trabajo*) de la cola del canal implica mutar el
`receiver`,
por lo que los hilos necesitan una forma segura de compartir y modificar
`receiver`; de lo contrario, podríamos obtener condiciones de carrera (como
se describe en el Capítulo 16).

Recuerde los *thread-safe smart pointers* analizados en el Capítulo 16: para
compartir la propiedad entre varios *threads* y permitir que los *threads*
muten el valor, necesitamos usar `Arc<Mutex<T>>`. El tipo `Arc` permitirá
que varios *workers* posean el receptor, y `Mutex` garantizará que solo un
*worker* obtenga un *trabajo* (*job*) del receptor a la vez. El listado
20-18 muestra los cambios que debemos hacer.

<span class="filename">Filename: src/lib.rs</span>

```rust
# use std::thread;
# use std::sync::mpsc;
use std::sync::Arc;
use std::sync::Mutex;
// --snip--

# pub struct ThreadPool {
#     workers: Vec<Worker>,
#     sender: mpsc::Sender<Job>,
# }
# struct Job;
#
impl ThreadPool {
    // --snip--
    pub fn new(size: usize) -> ThreadPool {
        assert!(size > 0);

        let (sender, receiver) = mpsc::channel();

        let receiver = Arc::new(Mutex::new(receiver));

        let mut workers = Vec::with_capacity(size);

        for id in 0..size {
            workers.push(Worker::new(id, Arc::clone(&receiver)));
        }

        ThreadPool {
            workers,
            sender,
        }
    }

    // --snip--
}

# struct Worker {
#     id: usize,
#     thread: thread::JoinHandle<()>,
# }
#
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--
#         let thread = thread::spawn(|| {
#            receiver;
#         });
#
#         Worker {
#             id,
#             thread,
#         }
    }
}
```

<span class="caption">Listado 20-18: Compartir el extremo receptor del canal
entre los *workers* que usan `Arc` y `Mutex`</span>

En `ThreadPool::new`, ponemos el extremo receptor del canal en un `Arc` y un
`Mutex`. Para cada nuevo *worker*, clonamos el `Arc` para aumentar el
recuento de referencias para que los *workers* puedan compartir la propiedad
del extremo receptor.

Con estos cambios, ¡el código compila! ¡Estamos llegando!

#### Implementando el método `execute`

Implementemos finalmente el método `execute` en `ThreadPool`. También
cambiaremos `Job` de una estructura a un alias de tipo para un *trait
object* que contiene el tipo de *closure* que recibe `execute`. Como se
explica en la sección “Creación de sinónimos de tipo con alias de tipo” en
el Capítulo 19, los alias de tipo nos permiten acortar los largos. Mire el
Listado 20-19.

<span class="filename">Filename: src/lib.rs</span>

```rust
// --snip--
# pub struct ThreadPool {
#     workers: Vec<Worker>,
#     sender: mpsc::Sender<Job>,
# }
# use std::sync::mpsc;
# struct Worker {}

type Job = Box<FnOnce() + Send + 'static>;

impl ThreadPool {
    // --snip--

    pub fn execute<F>(&self, f: F)
        where
            F: FnOnce() + Send + 'static
    {
        let job = Box::new(f);

        self.sender.send(job).unwrap();
    }
}

// --snip--
```

<span class="caption">Listado 20-19: Creación de un alias de tipo `Job`
para una `Box` que contiene cada *closure* y luego envía el *trabajo* (*job*
al canal</span>

Después de crear una nueva instancia `Job` utilizando el *closure* que
obtenemos en `execute`, enviamos ese trabajo al final del canal emisor.
Estamos llamando `unwrap` on `send` para el caso de que el envío falle. Esto
podría suceder si, por ejemplo, dejamos de ejecutar todos nuestros
*threads*, lo que significa que el receptor ha dejado de recibir mensajes
nuevos. Por el momento, no podemos detener la ejecución de nuestros
*threads*: nuestros *threads* continúan ejecutándose mientras exista el
*pool*. La razón por la que usamos `unwrap` es que sabemos que el caso de
falla no ocurrirá, pero el compilador no lo sabe.

¡Pero aún no hemos terminado! En el *worker*, nuestro *closure* pasó a
`thread::spawn` todavía sólo *referencias* al extremo receptor del canal. En
su lugar, necesitamos que el *closure* se repita para siempre, pidiendo al
receptor del canal que realice un trabajo y ejecute el trabajo cuando lo
obtenga. Hagamos el cambio que se muestra en el Listado 20-20 a
`Worker::new`.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || {
            loop {
                let job = receiver.lock().unwrap().recv().unwrap();

                println!("Worker {} got a job; executing.", id);

                (*job)();
            }
        });

        Worker {
            id,
            thread,
        }
    }
}
```

<span class="caption">Listado 20-20: Recepción y ejecución de trabajos en el
*worker’s thread*</span>

Aquí, primero llamamos a `lock` en el `receiver` para adquirir el *mutex*, y
luego llamamos `unwrap` al *panic* sobre cualquier error. La adquisición de
un bloqueo puede fallar si el *mutex* está en un estado *envenenado*, lo que
puede ocurrir si otro *thread* entra en *panic* mientras se mantiene el
bloqueo en lugar de liberar el bloqueo. En esta situación, llamar a `unwrap`
para que este *thread* entre en *panic* es la acción correcta a tomar.
Siéntase libre de cambiar este `unwrap` a un `expect` con un mensaje de
error que sea significativo para usted.

Si obtenemos el bloqueo en el *mutex*, llamamos a `recv` para recibir un
`Job` del canal. Un `unwrap` final también pasa aquí por los errores, lo que
puede ocurrir si el hilo que contiene el lado emisor del canal se ha apagado
similar a como el método `send` devuelve `Err` si el lado receptor se apaga.

La llamada a los bloques `recv`, por lo que si aún no hay *trabajo* (*job*),
el hilo actual esperará hasta que un *job* esté disponible. El `Mutex<T>`
asegura que solo un hilo `Worker` a la vez está tratando de solicitar un
*job*.

Teóricamente, este código debería compilarse. Desafortunadamente, el
compilador de Rust no es perfecto todavía, y obtenemos este error:

```text
error[E0161]: cannot move a value of type std::ops::FnOnce() +
std::marker::Send: the size of std::ops::FnOnce() + std::marker::Send cannot be
statically determined
  --> src/lib.rs:63:17
   |
63 |                 (*job)();
   |                 ^^^^^^
```

Este error es bastante críptico porque el problema es bastante críptico.
Para llamar a un *closure* `FnOnce` que está almacenado en `Box<T>`(que es
lo que es nuestro alias de tipo `Job`), el *closure* tiene que moverse
*fuera* del `Box<T>` porque el *closure* asume la propiedad del
`self` cuando lo llamamos. En general, Rust no nos permite mover un valor de
un `Box<T>` porque Rust no sabe cuán grande será el valor dentro de
`Box<T>`: recordar en el Capítulo 15 que utilizamos `Box<T>` precisamente
porque teníamos un tamaño desconocido que queríamos almacenar en un
`Box<T>` para obtener un valor de un tamaño conocido.

Como vimos en el listado 17-15, podemos escribir métodos que usan la
sintaxis `self:Box<Self>`, que permite que el método tome posesión de un
valor `Self` almacenado en `Box<T>`. Eso es exactamente lo que queremos
hacer aquí, pero desafortunadamente Rust no nos lo permite: la parte de Rust
que implementa el comportamiento cuando se llama a un *closure* no se
implementa con `self:Box<Self>`. Así que Rust aún no comprende que podría
usar `self:Box<Self>` en esta situación para apropiarse del *closure* y
sacar el *closure* del `Box<T>`.

Rust sigue siendo un trabajo en progreso con lugares donde el compilador
podría mejorarse, pero en el futuro, el código en el listado 20-20 debería
funcionar bien. ¡Personas como tú están trabajando para solucionar este y
otros problemas!. Después de que hayas terminado este libro, nos encantaría
que te unas.

Pero, por ahora, vamos a solucionar este problema utilizando un truco útil.
Podemos decirle a Rust explícitamente que en este caso podemos apropiarnos
del valor dentro de `Box<T>` usando `self:Box<Self>`; luego, una vez que
tenemos la propiedad del *closure*, podemos llamarlo. Esto implica definir
un nuevo *trait* `FnBox` con el método `call_box` que usará
`self:Box<Self>` en su firma, definiendo `FnBox` para cualquier tipo que
implemente `FnOnce()`, cambiando nuestro alias de tipo a use el nuevo
*trait* y cambie `Worker` para usar el método `call_box`. Estos cambios se
muestran en el listado 20-21.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
trait FnBox {
    fn call_box(self: Box<Self>);
}

impl<F: FnOnce()> FnBox for F {
    fn call_box(self: Box<F>) {
        (*self)()
    }
}

type Job = Box<FnBox + Send + 'static>;

// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || {
            loop {
                let job = receiver.lock().unwrap().recv().unwrap();

                println!("Worker {} got a job; executing.", id);

                job.call_box();
            }
        });

        Worker {
            id,
            thread,
        }
    }
}
```

<span class="caption">Listado 20-21: Agregar un nuevo *trait* `FnBox` para
evitar las limitaciones actuales de `Box<FnOnce()>`</span>

Primero, creamos un nuevo *trait* llamado `FnBox`. Este *trait* tiene el
método `call_box`, que es similar a los métodos `call` en los otros *traits*
`Fn*` excepto que toma `self:Box<Self>` para tomar posesión de `self` y
mover el valor fuera de `Box<T>`.

A continuación, implementamos el *trait* `FnBox` para cualquier tipo `F` que
implemente el *trait* `FnOnce()`. Efectivamente, esto significa que
cualquier *closure* `FnOnce ()` puede usar nuestro método `call_box`. La
implementación de `call_box` usa `(*self)()` para mover el *closure* fuera
de `Box <T>` y llamar al *closure*.

Ahora necesitamos que nuestro alias tipo `Job` sea un `Box` de cualquier
cosa que implemente nuestro nuevo *trait* `FnBox`. Esto nos permitirá usar
`call_box` en `Worker` cuando obtengamos un valor `Job` en lugar de invocar
el *closure* directamente. Implementar el *trait* `FnBox` para cualquier
*closure* `FnOnce()` significa que no tenemos que cambiar nada sobre los
valores reales que estamos enviando al canal. Ahora Rust es capaz de
reconocer que lo que queremos hacer está bien.

Este truco es muy astuto y complicado. No te preocupes si no tiene perfecto
sentido; algún día, será completamente innecesario.

¡Con la implementación de este truco, nuestro *thread pool* está en buen
estado! Dale un `cargo run` y haz algunas solicitudes:

```text
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field is never used: `workers`
 --> src/lib.rs:7:5
  |
7 |     workers: Vec<Worker>,
  |     ^^^^^^^^^^^^^^^^^^^^
  |
  = note: #[warn(dead_code)] on by default

warning: field is never used: `id`
  --> src/lib.rs:61:5
   |
61 |     id: usize,
   |     ^^^^^^^^^
   |
   = note: #[warn(dead_code)] on by default

warning: field is never used: `thread`
  --> src/lib.rs:62:5
   |
62 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
   = note: #[warn(dead_code)] on by default

    Finished dev [unoptimized + debuginfo] target(s) in 0.99 secs
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

¡Éxito! Ahora tenemos un *thread pool* que ejecuta las conexiones de forma
asincrónica. Nunca se crean más de cuatro *threads*, por lo que nuestro
sistema no se sobrecargará si el servidor recibe muchas solicitudes. Si
hacemos una solicitud a */sleep*, el servidor podrá atender otras
solicitudes haciendo que otro hilo las ejecute.

Después de aprender sobre el bucle `while let` en el Capítulo 18, tal vez se
pregunte por qué no escribimos el código del hilo del *worker* como se
muestra en el Listado 20-22.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        let thread = thread::spawn(move || {
            while let Ok(job) = receiver.lock().unwrap().recv() {
                println!("Worker {} got a job; executing.", id);

                job.call_box();
            }
        });

        Worker {
            id,
            thread,
        }
    }
}
```

<span class="caption">Listado 20-22: una implementación alternativa de
`Worker::new` using `while let`</span>

Este código se compila y ejecuta, pero no da como resultado el
comportamiento de *threading* deseado: una solicitud lenta aún causará que
otras solicitudes esperen a ser procesadas. La razón es algo sutil: la
estructura `Mutex` no tiene un método público de `unlock` porque la
propiedad del *lock* se basa en el *lifetime* de `MutexGuard<T>` dentro de
`LockResult<MutexGuard<T>>`  que devuelve el método `lock`. En el momento de
la compilación, el comprobador de préstamos puede entonces hacer cumplir la
regla de que no se puede acceder a un recurso protegido por un `Mutex` a
menos que tengamos el *lock*. Pero esta implementación también puede
provocar que el *lock* se mantenga más tiempo de lo previsto si no pensamos
detenidamente sobre la duración del `MutexGuard<T>`. Debido a que los
valores en la expresión `while` permanecen en el alcance durante la duración
del bloque, el *lock* permanece retenido durante la duración de la llamada
a `job.call_box()`, lo que significa que otros *workers* no pueden recibir
*jobs*.

Al utilizar `loop` en su lugar y adquirir el *lock* y un *job* dentro del
bloque en lugar de fuera de él, el `MutexGuard` devuelto por el método
`lock` se elimina tan pronto como finaliza la instrucción `let job`. Esto
garantiza que el bloqueo se mantenga durante la llamada a `recv`, pero se
libera antes de la llamada a `job.call_box ()`, lo que permite atender
varias solicitudes al mismo tiempo.