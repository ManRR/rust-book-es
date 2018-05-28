## Concurrencia de estado compartido

La transmisión de mensajes es una buena forma de manejar la concurrencia,
pero no es la única. Considere de nuevo esta parte del lema de la
documentación del lenguaje Go: "comunicarse compartiendo memoria".

¿Cómo se vería la comunicación al compartir la memoria? Además, ¿por qué los
entusiastas que pasan mensajes no lo usan y hacen todo lo contrario?

En cierto modo, los canales en cualquier lenguaje de programación son
similares a la propiedad única, porque una vez que transfiere un valor a un
canal, ya no debe usar ese valor. La concurrencia de memoria compartida es
como propiedad múltiple: varios subprocesos pueden acceder a la misma
ubicación de memoria al mismo tiempo. Como vimos en el Capítulo 15, donde los
punteros inteligentes hicieron posible la propiedad múltiple, la propiedad
múltiple puede agregar complejidad porque estos diferentes propietarios
necesitan administrarla. Las reglas de propiedad y el sistema de tipos de
Rust ayudan a que esta gestión sea correcta. Por ejemplo, veamos *mutexes*,
una de las primitivas de concurrencia más comunes para la memoria compartida.

### Usar mutexes para permitir el acceso a los datos de un hilo a la vez

*Mutex* es una abreviatura de *exclusión mutua*
(*mutual exclusion*), como en, un *mutex* permite solo
un hilo para acceder a algunos datos en un momento dado. Para acceder a los
datos en un *mutex*, un hilo debe indicar primero que quiere acceso pidiendo
adquirir el *lock* (*bloqueo*) del *mutex* el bloqueo es una estructura de
datos que es parte del *mutex* que realiza un seguimiento de quién tiene
acceso exclusivo a los datos en este momento. Por lo tanto, el *mutex* se
describe como *protector* de los datos que contiene a través del sistema de
bloqueo.

Los *mutex* tienen una reputación de ser difíciles de usar porque tienes que
recuerda dos reglas:

* Debe intentar adquirir el bloqueo antes de usar los datos.
* Cuando haya terminado con los datos que guardas mutex, debe desbloquear el
  datos para que otros hilos puedan adquirir el bloqueo.

Para una metáfora del mundo real para un *mutex*, imagine un panel de
discusión en una conferencia con solo un micrófono. Antes de que un panelista
pueda hablar, tienen que pregunta o indica que quieren usar el micrófono.
Cuando obtienen el micrófono, pueden hablar todo el tiempo que quieran y
luego entregar el micrófono al siguiente panelista que solicita hablar. Si un
panelista se olvida de quitar el micrófono cuando hayan terminado, nadie más
puede hablar. Si la administración del micrófono compartido falla, el panel
no funcionará ¡como se planeó!.

El manejo de *mutexes* puede ser increíblemente difícil de acertar, razón por
la cual muchas personas están entusiasmadas con los canales. Sin embargo,
gracias al sistema de tipos y las reglas de propiedad de Rust, no puedes
bloquear y desbloquear incorrectamente.

#### La API de `Mutex<T>`

Como ejemplo de cómo usar un mutex, comencemos utilizando un *mutex* en un
contexto de *single-threaded*, como se muestra en el listado 16-12:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::sync::Mutex;

fn main() {
    let m = Mutex::new(5);

    {
        let mut num = m.lock().unwrap();
        *num = 6;
    }

    println!("m = {:?}", m);
}
```

<span class="caption">Listado 16-12: Exploración de la API de `Mutex <T>` en
un contexto de single-threaded único para simplificar</span>

Al igual que con muchos tipos, creamos un `Mutex<T>` utilizando la función
asociada `new`. Para acceder a los datos dentro del *mutex*, usamos el método
`lock` para adquirir el *lock*. Esta llamada bloqueará el hilo actual por lo
que no puede hacer ningún trabajo mientras sea nuestro turno de tener el
candado.

La llamada a `lock` fallaría si otro hilo que mantiene el *lock* entrara en
pánico. En ese caso, nadie podría conseguir el *lock*, así que hemos elegido
`unwrap` y pánico en este hilo si estamos en esa situación.

Después de que hayamos adquirido el *lock*, podemos tratar el valor de
retorno, llamado `num` en este caso, como una referencia mutable a los datos
dentro. El sistema de tipo asegura que adquirimos un *lock* antes de usar el
valor en `m`:`Mutex<i32>` no es un `i32`, entonces *debemos* adquirir el
*lock* para poder usar el valor `i32`. Nosotros no puedo olvidar; el sistema
de tipo no nos permitirá acceder al `i32` interno en caso contrario.

Como podría sospechar, `Mutex <T>` es un puntero inteligente. Más exactamente
la llamada `lock` *devuelve* un puntero inteligente llamado `MutexGuard`.
Este puntero inteligente implementa `Deref` para señalar nuestros datos
internos; el puntero inteligente también tiene una implementación `Drop` que
libera el *lock* automáticamente cuando un `MutexGuard`
sale del alcance, lo que ocurre al final del alcance interno en el Listado
16-12. Como resultado, no nos arriesgamos a olvidar liberar el *lock* y
*lock* el uso de mutex por otros *threads* porque la liberación del *lock*
ocurre automáticamente.

Después de soltar el *lock*, podemos imprimir el valor *mutex* y ver que
pudimos para cambiar el `i32` interno a 6.

#### Compartir un `Mutex <T>` entre varios hilos

Ahora, intentemos compartir un valor entre múltiples hilos utilizando
`Mutex<T>`. Desarrollaremos 10 subprocesos y cada uno de ellos incrementará
el valor de un contador en 1, por lo que el contador pasará de 0 a 10. Tenga
en cuenta que los siguientes ejemplos tendrán errores de compilación, y los
usaremos para obtener más información sobre cómo usarlos. `Mutex<T>` y cómo
Rust nos ayuda a usarlo correctamente. El listado 16-13 tiene nuestro ejemplo
inicial:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];

    for _ in 0..10 {
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

<span class="caption">Listado 16-13: Diez *threads* cada incremento un
contador protegido por un `Mutex<T>`</span>

Creamos una variable `counter` para mantener un `i32` dentro de un
`Mutex<T>`, como hicimos en el Listado 16-12. A continuación, creamos 10
hilos iterando sobre un *trait* de números. Usamos `thread::spawn` y le damos
a todos los hilos el mismo *closure*, uno que mueve el contador al hilo,
adquiere un bloqueo en `Mutex<T>` llamando al método `lock`, y luego agrega 1
a el valor en el *mutex* cuando un hilo termina de ejecutar su cierre, `num`
saldrá del alcance y liberará el bloqueo para que otro hilo pueda adquirirlo.

En el hilo principal, recogemos todos los *join handles*. Luego, como hicimos
en el listado 16-2, llamamos a `join` en cada *handle* para asegurarnos de
que todos los hilos terminen. En ese punto, el hilo principal adquirirá el
bloqueo e imprimirá el resultado de este programa.

Insinuamos que este ejemplo no se compilaría. ¡Ahora descubramos por qué!.

```text
error[E0382]: capture of moved value: `counter`
  --> src/main.rs:10:27
   |
9  |         let handle = thread::spawn(move || {
   |                                    ------- value moved (into closure) here
10 |             let mut num = counter.lock().unwrap();
   |                           ^^^^^^^ value captured here after move
   |
   = note: move occurs because `counter` has type `std::sync::Mutex<i32>`,
   which does not implement the `Copy` trait

error[E0382]: use of moved value: `counter`
  --> src/main.rs:21:29
   |
9  |         let handle = thread::spawn(move || {
   |                                    ------- value moved (into closure) here
...
21 |     println!("Result: {}", *counter.lock().unwrap());
   |                             ^^^^^^^ value used here after move
   |
   = note: move occurs because `counter` has type `std::sync::Mutex<i32>`,
   which does not implement the `Copy` trait

error: aborting due to 2 previous errors
```

El mensaje de error indica que el valor `counter` se mueve al *closure* y
luego se captura cuando llamamos `lock`. Esa descripción suena como lo que
queríamos, ¡pero no está permitido!

Vamos a resolver esto simplificando el programa. En lugar de hacer 10 hilos
en un bucle 'for', hagamos dos hilos sin un bucle y veamos qué pasa.
Reemplace el primer bucle `for` en el Listado 16-13 con este código:

```rust,ignore
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Mutex::new(0);
    let mut handles = vec![];

    let handle = thread::spawn(move || {
        let mut num = counter.lock().unwrap();

        *num += 1;
    });
    handles.push(handle);

    let handle2 = thread::spawn(move || {
        let mut num2 = counter.lock().unwrap();

        *num2 += 1;
    });
    handles.push(handle2);

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

Hacemos dos hilos y cambiamos los nombres de variable usados con el segundo
hilo a `handle2` y `num2`. Cuando ejecutamos el código esta vez, compilar nos
da lo siguiente:

```text
error[E0382]: capture of moved value: `counter`
  --> src/main.rs:16:24
   |
8  |     let handle = thread::spawn(move || {
   |                                ------- value moved (into closure) here
...
16 |         let mut num2 = counter.lock().unwrap();
   |                        ^^^^^^^ value captured here after move
   |
   = note: move occurs because `counter` has type `std::sync::Mutex<i32>`,
   which does not implement the `Copy` trait

error[E0382]: use of moved value: `counter`
  --> src/main.rs:26:29
   |
8  |     let handle = thread::spawn(move || {
   |                                ------- value moved (into closure) here
...
26 |     println!("Result: {}", *counter.lock().unwrap());
   |                             ^^^^^^^ value used here after move
   |
   = note: move occurs because `counter` has type `std::sync::Mutex<i32>`,
   which does not implement the `Copy` trait

error: aborting due to 2 previous errors
```

Aha! El primer mensaje de error indica que `counter` se mueve al *closure*
para el hilo asociado con `handle`. Ese movimiento nos impide capturar
`contador` cuando intentamos llamar `lock` y almacenar el resultado en `num2`
en el segundo hilo. Entonces, Rust nos está diciendo que no podemos mover la
propiedad de `contador` en múltiples hilos. Esto era difícil de ver antes
porque nuestros hilos estaban en un bucle, y Rust no puede señalar diferentes
hilos en diferentes iteraciones del ciclo. Arreglemos el error del compilador
con un método de propiedad múltiple que discutimos en el Capítulo 15.

#### Propiedad múltiple con múltiples hilos

En el Capítulo 15, dimos un valor de varios propietarios usando el puntero
inteligente `Rc<T>` para crear un valor contado de referencia. Hagamos lo
mismo aquí y veamos qué pasa. Vamos a envolver el `Mutex<T>` en `Rc<T>` en el
Listado 16-14 y clonar el `Rc<T>` antes de mover la propiedad al hilo. Ahora
que hemos visto los errores, también volveremos a usar el ciclo `for`, y
mantendremos la palabra clave `move` con el closure.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use std::rc::Rc;
use std::sync::Mutex;
use std::thread;

fn main() {
    let counter = Rc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Rc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

<span class="caption">Listado 16-14: Intentando usar `Rc<T>` para permitir
que varios hilos sean propietarios del `Mutex<T>`</span>

Una vez más, compilamos y obtenemos ... ¡diferentes errores! el compilador
nos está enseñando mucho.

```text
error[E0277]: the trait bound `std::rc::Rc<std::sync::Mutex<i32>>:
std::marker::Send` is not satisfied in `[closure@src/main.rs:11:36:
15:10 counter:std::rc::Rc<std::sync::Mutex<i32>>]`
  --> src/main.rs:11:22
   |
11 |         let handle = thread::spawn(move || {
   |                      ^^^^^^^^^^^^^ `std::rc::Rc<std::sync::Mutex<i32>>`
cannot be sent between threads safely
   |
   = help: within `[closure@src/main.rs:11:36: 15:10
counter:std::rc::Rc<std::sync::Mutex<i32>>]`, the trait `std::marker::Send` is
not implemented for `std::rc::Rc<std::sync::Mutex<i32>>`
   = note: required because it appears within the type
`[closure@src/main.rs:11:36: 15:10 counter:std::rc::Rc<std::sync::Mutex<i32>>]`
   = note: required by `std::thread::spawn`
```

¡Guau, ese mensaje de error es muy prolijo! aquí hay algunas partes
importantes para enfocarse: el primer error en línea dice
```std::rc::Rc<std::sync::Mutex<i32>>` cannotbe sent between threads
safely``.
La razón para esto es en la siguiente parte importante para enfocarse, el mensaje de error. El mensaje de error destilado dice
``the trait bound `Send` is not satisfied``. Hablaremos de `Send` en la
próxima sección: es uno de los *trait* que asegura que los tipos que usamos
con los hilos están destinados a ser utilizados en situaciones concurrentes.

Desafortunadamente, `Rc<T>` no es seguro para compartir a través de
*threads*. Cuando `Rc<T>` administra el recuento de referencias, se suma al
conteo de cada llamada a `clon` y resta del recuento cuando se descarta cada
clon. Pero no usa ninguna primitiva de concurrencia para asegurarse de que
los cambios al conteo no puedan ser interrumpidos por otro hilo. Esto podría
llevar a recuentos incorrectos, errores sutiles que a su vez podrían provocar
pérdidas de memoria o un valor que se descarta antes de que terminemos con
él. Lo que necesitamos es un tipo exactamente como `Rc<T>` pero que haga
cambios en el recuento de referencias de una manera segura para hilos.

#### Recuento de referencia atómica con `Arc <T>`

Afortunadamente, `Arc<T>` *es* un tipo como `Rc<T>` que es seguro de usar en
situaciones concurrentes. La *a* significa *atomic*, lo que significa que es
un *tipo de referencia atómica*. Atomics es un tipo adicional de primitiva de
concurrencia que no cubriremos en detalle aquí: consulte la documentación
estándar de la biblioteca para `std::sync::atomic` para obtener más detalles.
En este punto, solo necesita saber que los atómicos funcionan como tipos
primitivos pero que es seguro compartirlos a través de *threads*.

A continuación, puede preguntarse por qué todos los tipos primitivos no son
atómicos y por qué los tipos de biblioteca estándar no están implementados
para usar `Arc<T>` de forma predeterminada. La razón es que la seguridad de
*threads* viene con una penalización de rendimiento que solo quieres pagar
cuando realmente lo necesitas. Si solo está realizando operaciones en valores
dentro de un solo hilo, su código puede ejecutarse más rápido si no tiene que
hacer cumplir las garantías que brindan los atómicos.

Volvamos a nuestro ejemplo: `Arc<T>` y `Rc<T>` tienen la misma API, por lo
que reparamos nuestro programa cambiando la línea `use`, la llamada a `new` y
la llamada a `clon`. El código en el listado 16-15 finalmente se compilará y ejecutará:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::sync::{Mutex, Arc};
use std::thread;

fn main() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    println!("Result: {}", *counter.lock().unwrap());
}
```

<span class="caption">Listado 16-15: Usar un `Arc<T>` para envolver el
`Mutex<T>` para poder compartir la propiedad entre múltiples hilos</span>

Este código imprimirá lo siguiente:

```text
Result: 10
```

¡Lo hicimos! Contamos de 0 a 10, lo que puede no parecer muy impresionante,
pero sí nos enseñó mucho sobre `Mutex<T>` y *thread safety*. También puede
usar la estructura de este programa para realizar operaciones más complicadas
que simplemente incrementar un contador. Usando esta estrategia, puedes
dividir un cálculo en partes independientes, dividir esas partes entre los
hilos, y luego usar un `Mutex<T>` para que cada hilo actualice el resultado
final con su parte.

### Similitudes entre `RefCell <T>` / `Rc <T>` y `Mutex <T>` / `Arc <T>`

Es posible que haya notado que `counter` es inmutable, pero podríamos obtener
una referencia mutable al valor que contiene; esto significa `Mutex<T>`
proporciona mutabilidad interior, como lo hace la familia `Cell`. De la misma
manera que utilizamos `RefCell<T>` en el Capítulo 15 para permitirnos mutar
contenidos dentro de un `Rc<T>`, usamos `Mutex<T>` para mutar contenidos
dentro de un `Arc<T>`.

Otro detalle a tener en cuenta es que Rust no puede protegerte de todo tipo
de errores lógicos cuando usas `Mutex<T>`. Recuerde en el Capítulo 15 que
usar `Rc<T>` conlleva el riesgo de crear ciclos de referencia, donde dos
valores `Rc<T>` se refieren entre sí, causando pérdidas de memoria. Del mismo
modo, `Mutex<T>` tiene el riesgo de crear *deadlocks*. Esto ocurre cuando una
operación necesita bloquear dos recursos y dos hilos han adquirido uno de los
bloqueos, lo que hace que esperen el uno al otro para siempre. Si le
interesan los bloqueos, intente crear un programa de Rust que tenga un
*deadlocks*; luego investigue las estrategias de mitigación de interbloqueo
para *mutexes* en cualquier lenguaje y aprenda a implementarlas en Rust. La
documentación API de la biblioteca estándar para `Mutex<T>` y `MutexGuard`
ofrece información útil.

Completaremos este capítulo hablando de los *trait* `Send` y `Sync` y cómo
podemos usarlos con tipos personalizados.
