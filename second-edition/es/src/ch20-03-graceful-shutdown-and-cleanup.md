## Elegante *Shutdown* y *Cleanup*

El código en el listado 20-21 responde a las solicitudes de forma asincrónica
mediante el uso de un *thread pool*, como pretendíamos. Recibimos algunas
advertencias sobre los campos `workers`, `id`, y `thread` que no estamos
usando de manera directa que nos recuerda que no estamos limpiando nada.
Cuando utilizamos el método menos elegante
<span class="keystroke">ctrl-c</span> para detener el hilo principal, todos
los demás hilos también se detienen inmediatamente, incluso si están en medio
de atender una solicitud.

Ahora implementaremos el *trait* `Drop` para llamar `join` en cada uno de los
*threads* en el *pool* para que puedan finalizar las solicitudes en las que
están trabajando antes de cerrar. Luego implementaremos una forma de decirle
a los hilos que deben dejar de aceptar nuevas solicitudes y cerrar. Para ver
este código en acción, modificaremos nuestro servidor para que acepte solo
dos solicitudes antes de cerrar correctamente su *thread pool*.

### Implementando el *Trait* `Drop` en `ThreadPool`

Comencemos implementando `Drop` en nuestro *thread pool*. Cuando se abandona
el *pool*, nuestros hilos deben unirse para asegurarse de que terminen su
trabajo. El listado 20-23 muestra un primer intento de implementación de un
`Drop`; este código aún no funcionará.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            worker.thread.join().unwrap();
        }
    }
}
```

<span class="caption">Listado 20-23: Unirse a cada subproceso cuando el
*thread pool* se sale del alcance</span>

Primero, recorremos cada uno de los *thread pool* de `workers`. Usamos
`&mut` para esto porque `self` es una referencia mutable, y también
necesitamos poder mutar `worker`. Para cada *worker*, imprimimos un mensaje
que dice que este *worker* en particular se está cerrando, y luego llamamos
`join` en el hilo de ese *worker*. Si la llamada a `join` falla, usamos
`unwrap` para hacer que Rust se inunde y entrar en un apagado desvergonzado.

Aquí está el error que obtenemos cuando compilamos este código:

```text
error[E0507]: cannot move out of borrowed content
  --> src/lib.rs:65:13
   |
65 |             worker.thread.join().unwrap();
   |             ^^^^^^ cannot move out of borrowed content
```

El error nos dice que no podemos llamar `join` porque solo tenemos un
préstamo mutable de cada `worker` y `join` toma posesión de su argumento.
Para resolver este problema, debemos mover el hilo de la instancia `Worker`
que posee `thread` para que `join` pueda consumir el hilo. Hicimos esto en el
Listado 17-15: si `Worker` tiene una `Option<thread::JoinHandle<()>` en su
lugar, podemos llamar al método `take` en `Option` para mover el valor de la
variante `Some` y dejar un una variante `None` en su lugar. En otras palabras
un `Worker` que se ejecuta tendrá una variante `Some` en `thread`, y cuando
queremos limpiar un `Worker`, reemplazaremos `Some` por `None` de modo que el
`Worker` no tenga un hilo para ejecutar.

Entonces sabemos que queremos actualizar la definición de `Worker` de esta
manera:

<span class="filename">Filename: src/lib.rs</span>

```rust
# use std::thread;
struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}
```

Ahora apoyémonos en el compilador para encontrar los otros lugares que
necesitan cambiar. Al revisar este código, obtenemos dos errores:

```text
error[E0599]: no method named `join` found for type
`std::option::Option<std::thread::JoinHandle<()>>` in the current scope
  --> src/lib.rs:65:27
   |
65 |             worker.thread.join().unwrap();
   |                           ^^^^

error[E0308]: mismatched types
  --> src/lib.rs:89:13
   |
89 |             thread,
   |             ^^^^^^
   |             |
   |             expected enum `std::option::Option`, found struct
   `std::thread::JoinHandle`
   |             help: try using a variant of the expected type: `Some(thread)`
   |
   = note: expected type `std::option::Option<std::thread::JoinHandle<()>>`
              found type `std::thread::JoinHandle<_>`
```

Vamos a abordar el segundo error, que apunta al código al final de
`Worker::new`; necesitamos envolver el valor `thread` en `Some` cuando
creamos un nuevo `Worker`. Realice los siguientes cambios para corregir este
error:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Job>>>) -> Worker {
        // --snip--

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

El primer error está en nuestra implementación `Drop`. Anteriormente
mencionamos que teníamos la intención de llamar a `take` en el valor
`Option` para mover `thread` de `worker`. Los siguientes cambios lo harán:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
impl Drop for ThreadPool {
    fn drop(&mut self) {
        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

Como se discutió en el Capítulo 17, el método `take` en `Option` saca la
variante `Some` y deja `None` en su lugar. Estamos usando `if let` para
desestructurar el `Some` y obtener el hilo; entonces llamamos `join` en el
hilo. Si el hilo de un *worker* ya es `None`, sabemos que el *worker* ya ha
limpiado el hilo, por lo que no ocurre nada en ese caso.

### Señalización a los *Threads*  para dejar de escuchar *Jobs*

Con todos los cambios que hemos realizado, nuestro código se compila sin
advertencias. Pero la mala noticia es que este código aún no funciona como
queremos. La clave es la lógica en los *closures* ejecutados por los hilos de
las instancias `Worker`: en este momento, llamamos `join`, pero eso no
cerrará los hilos porque `loop` siempre buscan *jobs*. Si intentamos soltar
nuestro `ThreadPool` con nuestra implementación actual de `drop`, el hilo
principal se bloqueará para siempre esperando a que termine el primer hilo.

Para solucionar este problema, modificaremos los hilos para que escuchen la
ejecución de `Job` o una señal de que deben dejar de escuchar y salir del
ciclo infinito. En lugar de instancias `Job`, nuestro canal enviará una de
estas dos variantes *enum*.

<span class="filename">Filename: src/lib.rs</span>

```rust
# struct Job;
enum Message {
    NewJob(Job),
    Terminate,
}
```

Esta enumeración `Message` será una variante `NewJob` que contiene el `Job`
que debe ejecutar el hilo, o será una variante `Terminate` que hará que el
hilo salga de su ciclo y se detenga.

Necesitamos ajustar el canal para usar valores de tipo `Mensaje` en lugar de
escribir `Job`, como se muestra en el Listado 20-24.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Message>,
}

// --snip--

impl ThreadPool {
    // --snip--

    pub fn execute<F>(&self, f: F)
        where
            F: FnOnce() + Send + 'static
    {
        let job = Box::new(f);

        self.sender.send(Message::NewJob(job)).unwrap();
    }
}

// --snip--

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Message>>>) ->
        Worker {

        let thread = thread::spawn(move ||{
            loop {
                let message = receiver.lock().unwrap().recv().unwrap();

                match message {
                    Message::NewJob(job) => {
                        println!("Worker {} got a job; executing.", id);

                        job.call_box();
                    },
                    Message::Terminate => {
                        println!("Worker {} was told to terminate.", id);

                        break;
                    },
                }
            }
        });

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

<span class="caption">Listado 20-24: Enviar y recibir valores `Message` y
salir del bucle si un `Worker` recibe `Message::Terminate`</span>

Para incorporar el *enum* `Message`, necesitamos cambiar `Job` a `Message` en
dos lugares: la definición de `ThreadPool` y la firma de `Worker::new`. El
método `execute` de `ThreadPool` necesita enviar *jobs wrapped* envueltos en
la variante `Message::NewJob`. Luego, en `Worker::new` donde se recibe un
`Message` desde el canal, el trabajo será procesado si se recibe la variante
`NewJob`, y el hilo se saldrá del ciclo si la variante `Terminate` es
recibida.

Con estos cambios, el código se compilará y continuará funcionando de la
misma forma que lo hizo después del Listado 20-21. Pero recibiremos una
advertencia porque no estamos creando ningún mensaje de la variedad
`Terminate`. Arreglemos esta advertencia cambiando nuestra implementación
`Drop` para que se vea como el Listado 20-25.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
impl Drop for ThreadPool {
    fn drop(&mut self) {
        println!("Sending terminate message to all workers.");

        for _ in &mut self.workers {
            self.sender.send(Message::Terminate).unwrap();
        }

        println!("Shutting down all workers.");

        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}
```

<span class="caption">Listado 20-25: Enviar `Message::Terminate` a los
*workers* antes de llamar `join` en cada hilo de *worker*</span>

Ahora estamos iterando sobre los *workers* dos veces: una para enviar un
mansaje `Terminate` para cada *worker* y una vez para llamar `join` en el
hilo de cada *worker*. Si nosotros intentado enviar un mensaje y
`join` inmediatamente en el mismo ciclo, no pudimos
garantizar que el *worker* en la iteración actual sea el que obtenga el
mensaje del canal

Para comprender mejor por qué necesitamos dos bucles separados, imagine un
escenario con dos *workers*, si usamos un solo ciclo para iterar a través de
cada *worker*, en la primera iteración, un mensaje de finalización se enviará
por el canal y el `join` llamado en el hilo del primer *worker*. Si ese
primer *worker* estaba ocupado procesando una solicitud en ese momento, el
segundo *worker* tomaría el mensaje de terminación del canal y se apagaría.
Nos quedaríamos esperando que el primer *worker* cerrara, pero nunca lo haría
porque el segundo hilo recogió el mensaje de terminación. ¡Punto muerto!

Para evitar este escenario, primero colocamos todos nuestros mensajes
`Terminate` en el canal en un bucle; luego unimos todos los hilos en otro
ciclo. Cada *worker* dejará de recibir solicitudes en el canal una vez que
reciba un mensaje de terminación. Por lo tanto, podemos estar seguros de que
si enviamos la misma cantidad de mensajes de terminación porque hay *workers*
cada *worker* recibirá un mensaje de terminación antes de que se llame a
`join` en su hilo.

Para ver este código en acción, modifiquemos `main` para aceptar solo dos
solicitudes antes de cerrar con gracia el servidor, como se muestra en el
Listado 20-26.

<span class="filename">Filename: src/bin/main.rs</span>

```rust,ignore
fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming().take(2) {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }

    println!("Shutting down.");
}
```

<span class="caption">Listado 20-26: Apague el servidor después de atender
dos solicitudes saliendo del ciclo</span>

No querrías que un servidor web del mundo real se apagara después de servir
solo dos solicitudes. Este código solo demuestra que el cierre y la limpieza
elegantes funcionan correctamente.

El método `take` se define en el *trait* `Iterator` y limita la iteración a
los dos primeros elementos como máximo. El `ThreadPool` saldrá del alcance a
 final de `main`, y se ejecutará la implementación `drop`.

Inicie el servidor con `cargo run`, y realice tres solicitudes. La tercera
solicitud debería generar un error, y en su terminal debería ver un resultado
similar al siguiente:

```text
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 1.0 secs
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 3 got a job; executing.
Shutting down.
Sending terminate message to all workers.
Shutting down all workers.
Shutting down worker 0
Worker 1 was told to terminate.
Worker 2 was told to terminate.
Worker 0 was told to terminate.
Worker 3 was told to terminate.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

Es posible que vea un pedido diferente de *workers* y mensajes impresos.
Podemos ver cómo funciona este código a partir de los mensajes: los
*workers* 0 y 3 obtuvieron las dos primeras solicitudes y luego, en la
tercera solicitud, el servidor dejó de aceptar las conexiones. Cuando e
`ThreadPool` sale del ámbito de aplicación al final de `main`, su
implementación `Drop` entra en acción y el *pool* ordena a todos los
*workers* que finalicen. Cada uno de los *workers* imprime un mensaje cuando
ven el mensaje de terminación, y luego el *thread pool* llama a `join` para
cerrar cada *worker thread*.

Observe un aspecto interesante de esta ejecución en particular: el
`ThreadPool` envió los mensajes de terminación al canal, y antes de que
cualquier *worker* recibiera los mensajes, intentamos unirnos al *worker* 0.
El *worker* 0 aún no había recibido el mensaje de terminación, por lo que el
hilo principal bloqueado esperando que el *worker* 0 termine. Mientras tanto,
cada uno de los workers recibió los mensajes de terminación. Cuando el
*worker* 0 finalizó, el hilo principal esperó a que el resto de los workers
terminara. En ese momento, todos habían recibido el mensaje de terminación y
pudieron cerrar.

¡Felicidades! Ya hemos completado nuestro proyecto; tenemos un servidor web
básico que usa un *thread pool* para responder de forma asincrónica. Podemos
realizar un cierre elegante del servidor, que limpia todos los hilos del
*pool*.

Aquí está el código completo para referencia:

<span class="filename">Filename: src/bin/main.rs</span>

```rust,ignore
extern crate hello;
use hello::ThreadPool;

use std::io::prelude::*;
use std::net::TcpListener;
use std::net::TcpStream;
use std::fs::File;
use std::thread;
use std::time::Duration;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();
    let pool = ThreadPool::new(4);

    for stream in listener.incoming().take(2) {
        let stream = stream.unwrap();

        pool.execute(|| {
            handle_connection(stream);
        });
    }

    println!("Shutting down.");
}

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];
    stream.read(&mut buffer).unwrap();

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

     let mut file = File::open(filename).unwrap();
     let mut contents = String::new();

     file.read_to_string(&mut contents).unwrap();

     let response = format!("{}{}", status_line, contents);

     stream.write(response.as_bytes()).unwrap();
     stream.flush().unwrap();
}
```

<span class="filename">Filename: src/lib.rs</span>

```rust
use std::thread;
use std::sync::mpsc;
use std::sync::Arc;
use std::sync::Mutex;

enum Message {
    NewJob(Job),
    Terminate,
}

pub struct ThreadPool {
    workers: Vec<Worker>,
    sender: mpsc::Sender<Message>,
}

trait FnBox {
    fn call_box(self: Box<Self>);
}

impl<F: FnOnce()> FnBox for F {
    fn call_box(self: Box<F>) {
        (*self)()
    }
}

type Job = Box<FnBox + Send + 'static>;

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

    pub fn execute<F>(&self, f: F)
        where
            F: FnOnce() + Send + 'static
    {
        let job = Box::new(f);

        self.sender.send(Message::NewJob(job)).unwrap();
    }
}

impl Drop for ThreadPool {
    fn drop(&mut self) {
        println!("Sending terminate message to all workers.");

        for _ in &mut self.workers {
            self.sender.send(Message::Terminate).unwrap();
        }

        println!("Shutting down all workers.");

        for worker in &mut self.workers {
            println!("Shutting down worker {}", worker.id);

            if let Some(thread) = worker.thread.take() {
                thread.join().unwrap();
            }
        }
    }
}

struct Worker {
    id: usize,
    thread: Option<thread::JoinHandle<()>>,
}

impl Worker {
    fn new(id: usize, receiver: Arc<Mutex<mpsc::Receiver<Message>>>) ->
        Worker {

        let thread = thread::spawn(move ||{
            loop {
                let message = receiver.lock().unwrap().recv().unwrap();

                match message {
                    Message::NewJob(job) => {
                        println!("Worker {} got a job; executing.", id);

                        job.call_box();
                    },
                    Message::Terminate => {
                        println!("Worker {} was told to terminate.", id);

                        break;
                    },
                }
            }
        });

        Worker {
            id,
            thread: Some(thread),
        }
    }
}
```

¡Podríamos hacer más aquí! Si desea continuar mejorando este proyecto, aquí
hay algunas ideas:

* Agregue más documentación a `ThreadPool` y sus métodos públicos.
* Agregar pruebas de la funcionalidad de la biblioteca.
* Cambie las llamadas a `unwrap` para un manejo de errores más robusto.
* Use `ThreadPool` para realizar alguna tarea que no sea servir solicitudes
 web.
* Encuentra un *crate* de *thread pool* en *https://crates.io/* e implemente
 un servidor web similar utilizando el *crate* en su lugar. Luego, compare su
 API y su robustez con el *thread pool* que implementamos.

## Resumen

¡Bien hecho! ¡Has llegado al final del libro! Queremos agradecerle por
unirse a nosotros en esta gira de Rust. Ahora está listo para implementar sus
propios proyectos de Rust y ayudar con los proyectos de otras personas. Tenga
en cuenta que hay una comunidad acogedora de otros *Rustaceans*
que les encantaría ayudarlo con cualquier desafío que encuentre en su viaje en
Rust.
