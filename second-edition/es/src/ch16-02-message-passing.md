## Uso *Message Passing* para transferir datos entre hilos

Un enfoque cada vez más popular para garantizar la concurrencia segura es
*message passing*, donde los hilos o actores se comunican enviándose mensajes
que contienen datos. Aquí está la idea en un lema de
[the Go language documentation](http://golang.org/doc/effective_go.html):  "Do not communicate by sharing memory; instead, share memory by communicating".

Una herramienta importante que Rust tiene para lograr la concurrencia deenvío de mensajes es el *channel*, un concepto de programación que la biblioteca
estándar de Rust proporciona implementación de. Puedes imaginar un canal en
la programación como algo así como un canal de agua, como un arroyo o un río.
Si pones algo así como pato de goma o barco en una corriente, viajará río
abajo hasta el final de la camino acuático.

Un canal en programación tiene dos mitades: un transmisor y un receptor. los
La mitad del transmisor es la ubicación aguas arriba donde se colocan patos
de goma en el río, y la mitad del receptor es donde el pato de goma termina
río abajo. Uno parte de su código llama métodos en el transmisor con los
datos que desea enviar, y otra parte verifica el destinatario para los
mensajes que llegan. Se dice que el canal está *cerrado* si la mitad del transmisor o del receptor está caído.

Aquí, trabajaremos en un programa que tiene un hilo para generar valores y
enviarlos por un canal, y otro hilo que recibirá los valores y
imprimirlos. Vamos a enviar valores simples entre hilos usando un canal
para ilustrar la característica. Una vez que esté familiarizado con la
técnica, podría usa canales para implementar un sistema de chat o un sistema
donde se realizan muchos subprocesos partes de un cálculo y enviar las partes
a un hilo que agrega el resultados.

Primero, en el listado 16-6, crearemos un canal pero no haremos nada con él.
Tenga en cuenta que esto aún no se compilará porque Rust no puede decir qué
tipo de valores quiero enviar por el canal.

<span class="filename">Filename: src/main.rs</span>

```rust
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();
#     tx.send(()).unwrap();
}
```

<span class="caption">Listado 16-6: Creando un canal y asignando las dos
mitades a `tx` y `rx`</span>

Creamos un nuevo canal usando la función `mpsc::channel`; `mpsc` significa
*productor múltiple, consumidor único*
(*multiple producer, single consumer*). En resumen, la forma en que la
biblioteca estándar de Rust implementa canales significa que un canal puede
tener múltiples *envíos* finales que producen valores pero solo un
*receptor* final que consume esos valores. Imagina que varias corrientes
fluyen juntas en un gran río: todo lo que se envíe a través de cualquiera de
las corrientes terminará en un río al final. Comenzaremos con un solo
productor por ahora, pero agregaremos varios productores cuando este ejemplo
funcione.

<!-- NEXT PARAGRAPH WRAPPED WEIRD INTENTIONALLY SEE #199 -->

La función `mpsc::channel` devuelve una tupla, cuyo primer elemento es el
extremo que envía y el segundo elemento es el extremo receptor. Las
abreviaturas `tx` y `rx` se usan tradicionalmente en muchos campos para
*transmisor* y *receptor* respectivamente, por lo que nombramos nuestras
variables como tales para indicar cada extremo. Estamos usando una
declaración `let` con un patrón que destruye las tuplas; discutiremos el uso
de patrones en sentencias `let` y la desestructuración en el Capítulo 18.
Usar una declaración `let` de esta manera es un enfoque conveniente para
extraer las piezas de la tupla devuelta por `mpsc::channel`.

Movamos el extremo transmisor a un hilo generado y hagamos que envíe una
cadena para que el hilo engendrado se comunique con el hilo principal, como
se muestra en el Listado 16-7. Esto es como poner un pato de goma en el río
río arriba o enviar un mensaje de chat de un hilo a otro.

<span class="filename">Filename: src/main.rs</span>

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });
}
```

<span class="caption">Listado 16-7: Mover `tx` a un hilo generado y enviar
“hi”</span>

De nuevo, estamos usando `thread::spawn` para crear un nuevo hilo y luego
usando `move` para mover `tx` al *closure* para que el hilo generado tenga
`tx`. El subproceso generado necesita poseer el extremo de transmisión del
canal para poder enviar mensajes a través del canal.

El extremo transmisor tiene un método de `send` que toma el valor que
queremos enviar. El método `send` devuelve un tipo `Result <T, E>`, por lo
que si el extremo receptor ya se ha eliminado y no hay ningún lugar para
enviar un valor, la operación de envío devolverá un error. En este ejemplo,
llamamos `unwrap` al pánico en caso de error. Pero en una aplicación real, lo
manejaríamos correctamente: regrese al Capítulo 9 para revisar estrategias
para el manejo correcto de errores.

En el listado 16-8, obtendremos el valor del extremo receptor del canal en el
hilo principal. Esto es como recuperar el pato de goma del agua al final del
río o como recibir un mensaje de chat.

<span class="filename">Filename: src/main.rs</span>

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

<span class="caption">Listado 16-8: Recibir el valor “hi” en el hilo
principal e imprimirlo</span>

El extremo receptor de un canal tiene dos métodos útiles: `recv` y
`try_recv`. Estamos usando `recv`, abreviatura de *receive*, que bloqueará la
ejecución del hilo principal y esperará hasta que se envíe un valor por el
canal. Una vez que se envía un valor, `recv` lo devolverá en un
`Result <T, E>`. Cuando el extremo emisor del canal se cierra, `recv`
devolverá un error para indicar que no vendrán más valores.

El método `try_recv` no bloquea, sino que devuelve inmediatamente un
`Result <T, E>`: un valor `Ok` que contiene un mensaje si hay uno disponible
y un valor `Err` si no hay ningún mensaje esta vez. El uso de `try_recv` es
útil si este hilo tiene otro trabajo pendiente mientras se esperan mensajes:
podríamos escribir un ciclo que llame `try_recv` de vez en cuando, maneje un
mensaje si hay uno disponible y haga otro trabajo por un tiempo hasta que
vuelva a verificar.

Hemos usado `recv` en este ejemplo por simplicidad; no tenemos otro trabajo
para el hilo principal que no sea esperar mensajes, por lo que es apropiado
bloquear el hilo principal.

Cuando ejecutamos el código en el listado 16-8, veremos el valor impreso
desde el hilo principal:

```text
Got: hi
```

¡Perfecto!

### Canales y transferencia de propiedad

Las reglas de *propiedad* (*ownership*) juegan un papel vital en el envío de
mensajes porque lo ayudan a escribir código seguro y simultáneo. La
prevención de errores en la programación concurrente es la ventaja de pensar
sobre la propiedad en todos sus programas de Rust. Hagamos un experimento
para mostrar cómo los canales y la propiedad trabajan juntos para evitar
problemas: trataremos de usar un valor `val` en el hilo generado *después* de
que lo hayamos enviado por el canal. Intente compilar el código en el listado
16-9 para ver por qué este código no está permitido:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
        println!("val is {}", val);
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

<span class="caption">Listado 16-9: Intentando usar `val` después de enviarlo
por el canal</span>

Aquí, tratamos de imprimir `val` después de enviarlo por el canal a través
de `tx.send`. Permitir esto sería una mala idea: una vez que el valor ha sido
enviado a otro hilo, ese hilo podría modificarlo o soltarlo antes de intentar
usar el valor nuevamente. Potencialmente, las modificaciones del otro
subproceso podrían causar errores o resultados inesperados debido a datos
inconsistentes o inexistentes. Sin embargo, Rust nos da un error si tratamos
de compilar el código en el Listado 16-9:

```text
error[E0382]: use of moved value: `val`
  --> src/main.rs:10:31
   |
9  |         tx.send(val).unwrap();
   |                 --- value moved here
10 |         println!("val is {}", val);
   |                               ^^^ value used here after move
   |
   = note: move occurs because `val` has type `std::string::String`, which does
not implement the `Copy` trait
```

Nuestro error de concurrencia ha causado un error de tiempo de compilación.
La función `send` toma posesión de su parámetro, y cuando se mueve el valor,
el receptor se apropia de él. Esto nos impide volver a usar accidentalmente
el valor después de enviarlo; el sistema de propiedad comprueba que todo está
bien.

### Enviar múltiples valores y ver el receptor esperando

El código en el listado 16-8 se compiló y ejecutó, pero no nos mostró
claramente que dos hilos separados estaban hablando entre sí a través del
canal. En el listado 16-10 hicimos algunas modificaciones que demostrarán que
el código en el listado 16-8 se está ejecutando simultáneamente: el hilo
generado ahora enviará múltiples mensajes y se detendrá por un segundo entre
cada mensaje.

<span class="filename">Filename: src/main.rs</span>

```rust
use std::thread;
use std::sync::mpsc;
use std::time::Duration;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

<span class="caption">Listado 16-10: Envío de mensajes múltiples y pausas
entre cada uno</span>

Esta vez, el hilo generado tiene un vector de *strings* que queremos enviar
al hilo principal. Realizamos iteraciones sobre ellos, enviando cada uno
individualmente, y hacemos una pausa entre cada uno llamando a la función
`thread::sleep` con un valor `Duration` de 1 segundo.

En el hilo principal, no estamos llamando a la función `recv` explícitamente
más: en su lugar, estamos tratando `rx` como un iterador. Para cada valor
recibido, lo estamos imprimiendo. Cuando el canal está cerrado, la iteración
finalizará.

Al ejecutar el código en el listado 16-10, debería ver el siguiente resultado
con una pausa de 1 segundo entre cada línea:

```text
Got: hi
Got: from
Got: the
Got: thread
```

Debido a que no tenemos ningún código que haga una pausa o se retrase en el
ciclo `for` en el hilo principal, podemos decir que el hilo principal está
esperando recibir los valores del hilo generado.

### Creando múltiples productores clonando el transmisor

Anteriormente mencionamos que `mpsc` era un acrónimo de
*productor múltiple, consumidor único*
(*multiple producer, single consumer*). Pongamos `mpsc` para usar y expandir
el código en el Listado 16-10 para crear múltiples hilos que envíen valores
al mismo receptor. Podemos hacerlo clonando la mitad transmisora del canal,
como se muestra en el listado 16-11:

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::thread;
# use std::sync::mpsc;
# use std::time::Duration;
#
# fn main() {
// --snip--

let (tx, rx) = mpsc::channel();

let tx1 = mpsc::Sender::clone(&tx);
thread::spawn(move || {
    let vals = vec![
        String::from("hi"),
        String::from("from"),
        String::from("the"),
        String::from("thread"),
    ];

    for val in vals {
        tx1.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

thread::spawn(move || {
    let vals = vec![
        String::from("more"),
        String::from("messages"),
        String::from("for"),
        String::from("you"),
    ];

    for val in vals {
        tx.send(val).unwrap();
        thread::sleep(Duration::from_secs(1));
    }
});

for received in rx {
    println!("Got: {}", received);
}

// --snip--
# }
```

<span class="caption">Listado 16-11: Envío de múltiples mensajes de múltiples
productores</span>

Esta vez, antes de crear el primer hilo generado, llamamos `clone` en el
extremo de envío del canal. Esto nos dará un nuevo identificador de envío que
podemos pasar al primer hilo generado. Pasamos el final de envío original del
canal a un segundo hilo generado. Esto nos da dos hilos, cada uno enviando
mensajes diferentes al extremo receptor del canal.

Cuando ejecuta el código, su resultado debería verse más o menos así:

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

Puede ver los valores en otro orden; depende de tu sistema. Esto es lo que
hace que la concurrencia sea interesante y difícil. Si experimentas con
`thread::sleep`, dándole varios valores en los diferentes hilos, cada
ejecución será más no determinista y creará resultados diferentes cada vez.

Ahora que hemos analizado cómo funcionan los canales, veamos un método
diferente de concurrencia.