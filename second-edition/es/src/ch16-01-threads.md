## Usar hilos *Threads* para ejecutar código simultáneamente

En la mayoría de los sistemas operativos actuales, el código de un programa
ejecutado se ejecuta en *proceso* y el sistema operativo administra múltiples
procesos a la vez. Dentro de su programa, también puede tener partes
independientes que se ejecutan simultáneamente. Las características que
ejecutan estas partes independientes se llaman *hilos* (*threads*).

La división de la computación en su programa en múltiples hilos puede mejorar
el rendimiento porque el programa realiza múltiples tareas al mismo tiempo,
pero también agrega complejidad. Debido a que los hilos se pueden ejecutar
simultáneamente, no hay garantía inherente sobre el orden en que se
ejecutarán partes de su código en diferentes hilos. Esto puede ocasionar
problemas, como:

* Condiciones de carrera, donde los hilos están accediendo a los datos o
 recursos en un orden inconsistente
* *Deadlocks*, donde dos subprocesos están esperando el uno al otro para
 terminar usando un recurso que tiene el otro subproceso, evitando que ambos
 subprocesos continúen
* Errores que ocurren solo en ciertas situaciones y son difíciles de
 reproducir y corregir confiablemente


Rust intenta mitigar los efectos negativos del uso de hilos, pero
la programación en un contexto multiproceso todavía requiere una reflexión
cuidadosa y requiere una estructura de código que es diferente de la de los
programas que se ejecutan en un solo hilo.

Los lenguajes de programación implementan subprocesos de diferentes formas.
Muchos de los sistemas operativos proporcionan una API para crear nuevos
hilos. Este modelo donde un lenguaje llama a las API del sistema operativo
para crear subprocesos a veces se llama *1:1*, es decir, un hilo del sistema operativo por hilo de un lengunaje.

Muchos lenguajes de programación proporcionan su propia implementación
especial de hilos. Los hilos de programación proporcionados por el lebguaje
se conocen como hilos *verdes*, y los lenguajes que usan estos hilos verdes
los ejecutarán en el contexto de un diferente número de hilos del sistema
operativo. Por esta razón, el modelo de subproceso verde se llama modelo
*M:N* : hay `M` subprocesos verdes por `N` hilos del sistema operativo,
donde `M` y `N` no son necesariamente los mismos número.

Cada modelo tiene sus propias ventajas y desventajas, y la compensación más
importante para Rust es el soporte en tiempo de ejecución. *Runtime* es un término confuso y puede tener diferentes significados en diferentes contextos.

En este contexto, por *runtime* nos referimos al código que está incluido en
el lenguaje en cada binario este código puede ser grande o pequeño según el
lenguaje, pero cada lenguaje no ensamblado tendrá una cierta cantidad de
código de tiempo de ejecución. Por esa razón, coloquialmente cuando las
personas dicen que un lenguaje no tiene “tiempo de ejecución”, a menudo
significa “tiempo de ejecución pequeño” (*“small runtime.”*). Los tiempos de
ejecución más pequeños tienen menos funciones pero tienen el
ventaja de dar como resultado binarios más pequeños, que hacen que sea más
fácil combinar el lenguaje con otros lenguajes en más contextos. Aunque
muchos lenguajes están bien para aumentar el tamaño del tiempo de ejecución a
cambio de más características, Rust no necesita casi tiempo de ejecución y no
puede comprometerse a poder llamar a C para mantener el rendimiento.

El modelo *green-threading* M: N requiere un tiempo de ejecución de lenguaje
más grande para administrar *threads*. Como tal, la biblioteca estándar de
Rust solo proporciona una implementación de 1:1 *threading*. Debido a que
Rust es un lenguaje de bajo nivel, hay *crates* que implemente el *threading*
M:N
if you would rather trade overhead for aspects such as more control over
which threads run when and lower costs of context switching, for example.

Ahora que hemos definido los hilos en Rust, exploremos cómo usar el
API relacionada con hilos proporcionada por la biblioteca estándar.

### Creando un nuevo hilo *Thread* con `spawn`

Para crear un nuevo hilo, llamamos a la función `thread::spawn` y le pasamos
un *closure* (hablamos de *closure* en el Capítulo 13) que contiene el código
que queremos ejecutar en el nuevo hilo. El ejemplo en el listado 16-1 imprime
un texto de un hilo principal y otro texto de un nuevo hilo:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;

fn main() {
    thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

<span class="caption">Listado 16-1: Creando un nuevo hilo para imprimir una
cosa mientras el hilo principal imprime algo más</span>

Tenga en cuenta que con esta función, el nuevo *thread* se detendrá cuando e
hilo principal finalice, haya terminado de ejecutarse o no. El resultado de
este programa puede ser un poco diferente cada vez, pero se verá de manera
similar a lo siguiente:

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

Las llamadas a `thread::sleep` obligan a un hilo a detener su ejecución
durante un breve período de tiempo, permitiendo la ejecución de un hilo
diferente. Los hilos probablemente se turnarán, pero eso no está garantizado:
depende de cómo el sistema operativo programa los hilos. En esta ejecución,
el hilo principal se imprimió primero, aunque la declaración de impresión del
hilo generado aparece primero en el código. Y a pesar de que le dijimos al
hilo generado que se imprimiera hasta que `i` sea 9, solo llegó a 5 antes de
que el hilo principal se apagara.

Si ejecuta este código y solo ve el resultado del hilo principal, o no ve
ninguna superposición, intente aumentar los números en los rangos para crear
más oportunidades para que el sistema operativo cambie entre los hilos.

### Esperando a que todos los hilos terminen usando `join` *Handles*

El código en el listado 16-1 no solo detiene el subproceso *engendrado*
(*spawned*) prematuramente la mayor parte del tiempo debido al final del
subproceso principal, sino que tampoco puede garantizar que el subproceso
generado se ejecute en absoluto. ¡La razón es que no hay garantía sobre el
orden en que se ejecutan los hilos!

Podemos solucionar el problema de que el subproceso generado no se ejecute, o
no se ejecute por completo, al guardar el valor de retorno de
`thread::spawn` en una variable. El tipo de devolución de `thread::spawn` es
`JoinHandle`. Un `JoinHandle` es un valor propio que, cuando llamemos al
método `join`, esperará a que su subproceso termine. El listado 16-2 muestra
cómo usar el `JoinHandle` del hilo que creamos en el listado 16-1 y llamar
`join` para asegurarse de que el hilo generado termine antes de que `main`
salga:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

<span class="caption">Listado 16-2: Guardar un `JoinHandle` de
`thread::spawn` para garantizar que el hilo se ejecuta hasta su
finalización</span>

Llamar a `join` en el *handle* bloquea el hilo que se está ejecutando
actualmente hasta que termina el hilo representado por el *handle*.
*Bloquear* un hilo significa que el hilo no puede funcionar o salir.
Debido a que hemos llamado `join` después del ciclo `for` del hilo principal,
la ejecución del Listado 16-2 debería producir un resultado similar al siguiente:

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

Los dos hilos continúan alternando, pero el hilo principal espera debido a la
llamada a `handle.join()` y no termina hasta que finaliza el hilo generado.

Pero veamos qué sucede cuando movemos `handle.join()` antes del bucle `for`
en `main`, así:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    handle.join().unwrap();

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
}
```

El hilo principal esperará a que termine el hilo generado (spawned) y luego ejecutará su ciclo `for`, por lo que la salida ya no se intercalará, como se
muestra aquí:

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Los pequeños detalles, como donde se llama a `join`, pueden afectar si sus
hilos se ejecutan al mismo tiempo o no.

### Usar `move` *Closures* con *Threads*

El `move` *closure* se usa a menudo junto con `thread::spawn` porque le
permite usar datos de un hilo en otro hilo.

En el Capítulo 13, mencionamos que podemos usar la palabra clave `move` antes
de la lista de parámetros de un *closure* para forzar al *closure* a tomar
posesión de los valores que utiliza en el entorno. Esta técnica es
especialmente útil cuando se crean nuevos hilos para transferir la propiedad
de los valores de un hilo a otro.

Observe en el listado 16-1 que el *closure* que pasamos a `thread::spawn` no
toma argumentos: no estamos usando ningún dato del hilo principal en el
código del hilo generado. Para usar datos del hilo principal en el hilo
generado, el *closure* del hilo engendrado debe capturar los valores que
necesita. El listado 16-3 muestra un intento de crear un vector en el hilo
principal y usarlo en el hilo generado. Sin embargo, esto aún no funcionará,
como verá en un momento.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

<span class="caption">Listado 16-3: Intentando usar un vector creado por el
hilo principal en otro hilo</span>

El *closure* utiliza `v`, por lo que capturará `v` y lo convertirá en parte
del entorno del *closure*. Debido a que `thread::spawn` ejecuta este
*closure* en un nuevo hilo, deberíamos poder acceder a `v` dentro del nuevo
hilo. Pero cuando compilamos este ejemplo, obtenemos el siguiente error:

```text
error[E0373]: closure may outlive the current function, but it borrows `v`,
which is owned by the current function
 --> src/main.rs:6:32
  |
6 |     let handle = thread::spawn(|| {
  |                                ^^ may outlive borrowed value `v`
7 |         println!("Here's a vector: {:?}", v);
  |                                           - `v` is borrowed here
  |
help: to force the closure to take ownership of `v` (and any other referenced
variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ^^^^^^^
```

Rust *infiere* cómo capturar `v`, y porque `println!` solo necesita una
referencia a `v`, el *closure* intenta tomar prestado `v`. Sin embargo, hay
un problema: Rust no puede decir cuánto tiempo se ejecutará el subproceso
generado, por lo que no sabe si la referencia a `v` siempre será válida.

El listado 16-4 proporciona un escenario que es más probable que tenga una
referencia a `v` que no será válida:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(|| {
        println!("Here's a vector: {:?}", v);
    });

    drop(v); // oh no!

    handle.join().unwrap();
}
```

<span class="caption">Listado 16-4: Un hilo con un *closure* que intenta
capturar una referencia a `v` de un hilo principal que suelta `v`</span>

Si se nos permitiera ejecutar este código, existe la posibilidad de que el
hilo generado se ponga inmediatamente en segundo plano sin ejecutar en
absoluto. El hilo generado tiene una referencia a `v` dentro, pero el hilo
principal cae inmediatamente `v`, usando la función `drop` que discutimos en
el Capítulo 15. Luego, cuando el hilo engendrado comienza a ejecutarse, `v`
ya no está válido, por lo que una referencia al mismo tampoco es válida. ¡Oh
no!

Para corregir el error del compilador en el listado 16-3, podemos usar los
consejos del mensaje de error:

```text
help: to force the closure to take ownership of `v` (and any other referenced
variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ^^^^^^^
```

Al agregar la palabra clave `mover` antes del *closure*, forzamos al
*closure* a tomar posesión de los valores que está usando en lugar de
permitir que Rust infiera que debe tomar prestados los valores. La
modificación del listado 16-3 que se muestra en el listado 16-5 se compilará
y ejecutará según lo que pretendemos:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

<span class="caption">Listado 16-5: Usar la palabra clave `move` para forzar
a un *closure* a tomar posesión de los valores que usa</span>

¿Qué pasaría con el código en el listado 16-4 donde el hilo principal llamado
`drop` si usamos un *closure* `move`?, ¿`move` arreglaría ese caso?.
Lamentablemente no; obtendríamos un error diferente porque lo que el listado
16-4 intenta hacer no está permitido por un motivo diferente. Si agregamos
`move` al *closure*, moveríamos `v` al entorno del *closure*, y ya no
podríamos llamar `drop` en el hilo principal. Obtendríamos este error de
compilación en su lugar:

```text
error[E0382]: use of moved value: `v`
  --> src/main.rs:10:10
   |
6  |     let handle = thread::spawn(move || {
   |                                ------- value moved (into closure) here
...
10 |     drop(v); // oh no!
   |          ^ value used here after move
   |
   = note: move occurs because `v` has type `std::vec::Vec<i32>`, which does
   not implement the `Copy` trait
```

¡Las reglas de propiedad de Rust nos han salvado de nuevo! Obtuvimos un error
del código en el listado 16-3 porque Rust estaba siendo conservador y solo
pedía prestado `v` para el hilo, lo que significaba que el hilo principal
podría invalidar teóricamente la referencia del hilo generado. Al decirle a
Rust que mueva la propiedad de `v` al hilo generado, le garantizamos a Rust
que el hilo principal ya no usará `v`. Si cambiamos el Listado 16-4 de la
misma manera, entonces estamos violando las reglas de propiedad cuando
tratamos de usar `v` en el hilo principal. La palabra clave `move` anula el
valor predeterminado de préstamo de Rust; no nos permite violar las reglas de
propiedad.

Con una comprensión básica de los hilos y la API del hilo, veamos qué podemos
*hacer* con los hilos.
