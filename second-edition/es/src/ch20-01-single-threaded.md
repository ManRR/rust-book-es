## Building a Single-Threaded Web Server

Comenzaremos trabajando en un servidor web con un *single-threaded*. Antes de
comenzar, veamos una descripción general rápida de los protocolos
involucrados en la creación de servidores web. Los detalles de estos
protocolos están más allá del alcance de este libro, pero una breve
descripción general le brindará la información que necesita.

Los dos protocolos principales que intervienen en los servidores web son el
*Protocolo de transferencia de hipertexto*, (*Hypertext Transfer
Protocol*) *(HTTP)* y el
*Protocolo de control de transmisión*, (*Transmission Control Protocol*)
*(TCP)*. Ambos protocolos son *protocolos de solicitud-respuesta*, (*request-response*), lo que significa que un *cliente* inicia las
solicitudes y un *servidor* escucha las solicitudes y proporciona una
respuesta al cliente. El contenido de esas solicitudes y respuestas está
definido por los protocolos.

TCP es el protocolo de nivel inferior que describe los detalles de cómo se
obtiene la información de un servidor a otro, pero no especifica qué es esa
información. HTTP se construye sobre TCP definiendo el contenido de las
solicitudes y respuestas. Es técnicamente posible utilizar HTTP con otros
protocolos, pero en la gran mayoría de los casos, HTTP envía sus datos a
través de TCP. Trabajaremos con los bytes sin procesar de las solicitudes y
respuestas TCP y HTTP.

### Escuchando la conexión TCP

Nuestro servidor web necesita escuchar una conexión TCP, por lo que es la
primera parte en la que trabajaremos. La biblioteca estándar ofrece un módulo
`std::net` que nos permite hacer esto. Hagamos un nuevo proyecto de la manera
habitual:

```text
$ cargo new hello --bin
     Created binary (application) `hello` project
$ cd hello
```

Ahora ingrese el código en el Listado 20-1 en *src/main.rs* para comenzar.
Este código escuchará en la dirección `127.0.0.1: 7878` para las
transmisiones entrantes de TCP. Cuando recibe una transmisión entrante,
imprimirá `Connection established!`.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
use std::net::TcpListener;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        println!("Connection established!");
    }
}
```

<span class="caption">Listado 20-1: Escuchando las transmisiones entrantes e
imprimiendo un mensaje cuando recibimos una transmisión</span>

Usando `TcpListener`, podemos escuchar las conexiones TCP en la dirección
`127.0.0.1:7878`. En la dirección, la sección antes de los dos puntos es una
dirección IP que representa su computadora (esto es igual en cada computadora
y no representa específicamente la computadora de los autores), y `7878` es
el puerto. Hemos elegido este puerto por dos razones: HTTP normalmente se
acepta en este puerto, y 7878 es *rust* escrito en un teléfono.

La función `bind` en este escenario funciona como la función `new` en que
devolverá una nueva instancia `TcpListener`. La razón por la cual la función
se llama `bind` es que, en una red, conectarse a un puerto para escuchar se
conoce como “binding to a port”.

La función `bind` devuelve un `Result <T, E>`, que indica que el enlace puede
fallar. Por ejemplo, conectarse al puerto 80 requiere privilegios de
administrador (los no administradores pueden escuchar solo en puertos de más
de 1024), por lo que si intentamos conectarnos al puerto 80 sin ser un
administrador, el enlace no funcionaría. Como otro ejemplo, el enlace no
funcionaría si ejecutamos dos instancias de nuestro programa y, por lo tanto,
teníamos dos programas escuchando el mismo puerto. Debido a que estamos
escribiendo un servidor básico solo para fines de aprendizaje, no nos
preocuparemos por manejar este tipo de errores; en su lugar, usamos `unwrap`
para detener el programa si ocurren errores.

El método `incoming` en `TcpListener` devuelve un iterador que nos da un
secuencia de flujos (más específicamente, flujos de tipo `TcpStream`). Un
solo *stream* representa una conexión abierta entre el cliente y el servidor.
Una *connection* es el nombre del proceso completo de solicitud y respuesta
en el que el cliente se conecta al servidor, el servidor genera una respuesta
y el servidor cierra la conexión. Como tal, `TcpStream` leerá de sí mismo
para ver qué el cliente envió y luego nos permitió escribir nuestra respuesta
a la transmisión. En general, este bucle `for` procesará cada conexión por
turno y producirá una serie de secuencias para que podamos manejar.

Por ahora, nuestro manejo de la transmisión consiste en llamar a `unwrap`
para terminar nuestro programa si la transmisión tiene algún error; si no hay
ningún error, el programa imprime un mensaje. Añadiremos más funcionalidades
para el caso de éxito en el siguiente listado. La razón por la que podríamos
recibir errores del método `incoming` cuando un cliente se conecta al
servidor es que en realidad no estamos iterando conexiones. En cambio,
estamos iterando sobre *intentos de conexión*. los conexión puede no ser
exitosa por una serie de razones, muchas de ellas sistema operativo
específico. Por ejemplo, muchos sistemas operativos tienen un límite para la
cantidad de conexiones abiertas simultáneas que pueden admitir; nueva conexión
los intentos más allá de ese número producirán un error hasta que algunos de
los abiertos las conexiones estan cerradas

¡Tratemos de ejecutar este código! invocar `cargo run`  en la terminal y
luego cargar *127.0.0.1: 7878* en un navegador web. El navegador debe mostrar
un mensaje de error como “Connection reset”, porque el servidor no está
enviando ningún datos. Pero cuando mira su terminal, debería ver varios
mensajes que se imprimieron cuando el navegador se conectó al servidor!

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

A veces, verá múltiples mensajes impresos para una solicitud del navegador;
la razón podría ser que el navegador está haciendo una solicitud de la página
así como una solicitud de otros recursos, como el icono *favicon.ico* que
aparece en la pestaña del navegador.

También podría ser que el navegador esté intentando conectarse al servidor
varias veces porque el servidor no responde con ningún dato. Cuando `stream`
sale del alcance y se elimina al final del ciclo, la conexión se cierra como
parte de la implementación `drop`. Los navegadores a veces tratan las
conexiones cerradas reintentando, porque el problema puede ser temporal. ¡El
factor importante es que hemos logrado obtener un control para una conexión
TCP!

Recuerde detener el programa presionando
<span class="keystroke">ctrl-c</span> cuando termine de ejecutar una versión
particular del código. Luego, reinicie `cargo run` después de realizar cada
conjunto de cambios de código para asegurarse de que está ejecutando el
código más nuevo.

### Leyendo la Solicitud

¡Implementemos la funcionalidad para leer la solicitud desde el navegador!.
Para separar las preocupaciones de primero obtener una conexión y luego tomar
alguna medida con la conexión, comenzaremos una nueva función para procesar
las conexiones. En esta nueva función `handle_connection`, leeremos datos de
la transmisión TCP e imprimiremos para que podamos ver los datos que se
envían desde el navegador. Cambie el código para que se vea como el Listado
20-2.

<span class="filename">Filename: src/main.rs</span>

```rust,no_run
use std::io::prelude::*;
use std::net::TcpStream;
use std::net::TcpListener;

fn main() {
    let listener = TcpListener::bind("127.0.0.1:7878").unwrap();

    for stream in listener.incoming() {
        let stream = stream.unwrap();

        handle_connection(stream);
    }
}

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];

    stream.read(&mut buffer).unwrap();

    println!("Request: {}", String::from_utf8_lossy(&buffer[..]));
}
```

<span class="caption">Listado 20-2: Lectura del `TcpStream` e impresión de
los datos</span>

Traemos `std::io::prelude` en alcance para obtener acceso a ciertos
*traits* que nos permiten leer y escribir en la transmisión. En el bucle
`for` de la función `main`, en lugar de imprimir un mensaje que dice que
hicimos una conexión, ahora llamamos a la nueva función `handle_connection` y
le pasamos el `stream`.

En la función `handle_connection`, hemos hecho que el parámetro `stream` sea
mutable. La razón es que la instancia `TcpStream` hace un seguimiento de qué
datos nos devuelve internamente. Puede leer más datos de los que pedimos y
guardarlos para la próxima vez que solicitemos datos. Por lo tanto, debe ser
`mut` porque su estado interno puede cambiar; por lo general, pensamos que
“reading” no necesita mutación, pero en este caso necesitamos la palabra
clave `mut`.

Luego, necesitamos leer de la transmisión. Hacemos esto en dos pasos: primero
declaramos un `buffer` en la pila para contener los datos que se leen. Hemos
creado un tamaño de buffer de 512 bytes, que es lo suficientemente grande
como para contener los datos de una solicitud básica y suficiente para
nuestros propósitos en este capítulo. Si quisiéramos manejar solicitudes de
un tamaño arbitrario, la gestión del buffer debería ser más complicada; lo
mantendremos simple por ahora. Pasamos el búfer a `stream.read`, que leerá
bytes de `TcpStream` y los colocará en el búfer.

Segundo, convertimos los bytes en el buffer a una cadena e imprimimos esa
cadena. La función `String::from_utf8_lossy` toma `&[u8]` y produce `String`
a partir de ella. La parte “lossy” del nombre indica el comportamiento de
esta función cuando ve una secuencia UTF-8 no válida: reemplazará la
secuencia no válida con `�`, el `CARÁCTER DE REEMPLAZO U + FFFD`. Es posible
que vea caracteres de reemplazo para los caracteres en el búfer que no se
llenan con los datos de solicitud.

¡Probemos este código! Inicie el programa y vuelva a realizar una solicitud
en un navegador web. Tenga en cuenta que todavía obtendremos una página de
error en el navegador, pero la salida de nuestro programa en el terminal
ahora se verá similar a esto:

```text
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42 secs
     Running `target/debug/hello`
Request: GET / HTTP/1.1
Host: 127.0.0.1:7878
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101
Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Upgrade-Insecure-Requests: 1
������������������������������������
```

Dependiendo de su navegador, puede obtener resultados ligeramente diferentes.
Ahora que estamos imprimiendo los datos de solicitud, podemos ver por qué
recibimos múltiples conexiones de una solicitud de navegador mirando la ruta
después de `Request: GET`. Si todas las conexiones repetidas solicitan
*/*, sabemos que el navegador está intentando recuperar */* repetidamente
porque no recibe una respuesta de nuestro programa.

Vamos a desglosar los datos de esta solicitud para comprender qué le pide el
navegador a nuestro programa.

### Una mirada más cercana a una solicitud HTTP

HTTP es un protocolo basado en texto, y una solicitud toma este formato:

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

La primera línea es la *línea de solicitud* que contiene información sobre lo
que el cliente está solicitando. La primera parte de la línea de solicitud
indica el *método* se usa, como `GET` o `POST`, que describe cómo el cliente
está haciendo esta petición. Nuestro cliente usó una solicitud `GET`.

La siguiente parte de la línea de solicitud es */*, que indica el
*Recurso uniforme Identificador*, (*Uniform Resource Identifier*) *(URI)* que
el cliente está solicitando: un URI es casi, pero no del todo,
lo mismo que un *Localizador Uniforme de Recursos*, (*Uniform Resource
Locator*) *(URL)*. La diferencia entre los URI
y las URL no son importantes para nuestros propósitos en este capítulo, pero
la especificación HTTP utiliza el término URI, por lo que podemos sustituir
mentalmente URL por URI aquí.

La última parte es la versión HTTP que usa el cliente y luego la línea de
solicitud termina en una *secuencia CRLF*. (CRLF significa *carriage return*
y *line feed*, ¡que son términos de los días de la máquina de escribir!). La
secuencia CRLF también puede ser escrito como `\r\n`, donde `\r` es un
retorno de carro y `\n` es un salto de línea. La secuencia CRLF separa la
línea de solicitud del resto de los datos de solicitud.
Tenga en cuenta que cuando se imprime el CRLF, vemos un nuevo inicio de línea
en lugar de `\r\n`.

Mirando los datos de línea de solicitud que recibimos al ejecutar nuestro
programa hasta ahora, vemos que `GET` es el método, */* es el URI de
solicitud, y `HTTP/1.1` es la versión.

Después de la línea de solicitud, las líneas restantes que comienzan desde
`Host:` en adelante son encabezados, las peticiones `GET` no tienen cuerpo.

Intente hacer una solicitud desde un navegador diferente o solicite una
dirección, como *127.0.0.1: 7878/test*, para ver cómo cambian los datos de
solicitud.

Ahora que sabemos lo que el navegador está pidiendo, ¡enviemos algunos datos!

### Escribir una respuesta

Ahora implementaremos el envío de datos en respuesta a una solicitud del
cliente. Las respuestas tienen el siguiente formato:

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

La primera línea es *una línea de estado* (*status line*) que contiene la
versión HTTP utilizada en la respuesta, un código de estado numérico que
resume el resultado de la solicitud y una frase de motivo que proporciona una
descripción de texto del código de estado. Después de la secuencia CRLF hay
encabezados, otra secuencia CRLF y el cuerpo de la respuesta.

Aquí hay una respuesta de ejemplo que usa HTTP versión 1.1, tiene un código
de estado de 200, una frase de razón OK, sin encabezados y sin cuerpo:

```text
HTTP/1.1 200 OK\r\n\r\n
```

El código de estado 200 es la respuesta de éxito estándar. El texto es una
pequeña respuesta HTTP exitosa. ¡Escribamos esto en la transmisión como
respuesta a una solicitud exitosa!. Desde la función `handle_connection`,
elimine `println!` Que estaba imprimiendo los datos de solicitud y
reemplácelo con el código en el Listado 20-3.

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::io::prelude::*;
# use std::net::TcpStream;
fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];

    stream.read(&mut buffer).unwrap();

    let response = "HTTP/1.1 200 OK\r\n\r\n";

    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

<span class="caption">Listado 20-3: Escribir una pequeña respuesta HTTP
exitosa a la transmisión</span>

La primera nueva línea define la variable `response` que contiene los datos
del mensaje de éxito. Luego llamamos `as_bytes` a nuestra `response` para
convertir los datos de cadena en bytes. El método `write` en `stream` toma
 `&[u8]` y envía esos bytes directamente a través de la conexión.

Debido a que la operación `write` puede fallar, usamos `unwrap` en cualquier
resultado de error como antes. De nuevo, en una aplicación real agregaría el
manejo de errores aquí. Finalmente, `flush` esperará e impedirá que el
programa continúe hasta que todos los bytes se escriban en la conexión;
`TcpStream` contiene un búfer interno para minimizar las llamadas al sistema
operativo subyacente.

Con estos cambios, ejecutemos nuestro código y hagamos una solicitud. Ya no
imprimimos datos en la terminal, por lo que no veremos otra salida que no sea
la salida de Cargo. Cuando carga *127.0.0.1:7878* en un navegador web, debe
obtener una página en blanco en lugar de un error. ¡Has codificado
manualmente una solicitud y respuesta HTTP!

### Devolución de HTML real

Implementemos la funcionalidad para devolver más que una página en blanco.
Cree un nuevo archivo, *hello.html*, en la raíz del directorio de su proyecto
no en el directorio *src*. Puede ingresar cualquier HTML que desee; El
listado 20-4 muestra una posibilidad.

<span class="filename">Filename: hello.html</span>

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Hello!</h1>
    <p>Hi from Rust</p>
  </body>
</html>
```

<span class="caption">Listado 20-4: Un archivo HTML de muestra para devolver
en una respuesta</span>

Este es un documento HTML5 mínimo con un encabezado y texto. Para devolver
esto desde el servidor cuando se recibe una solicitud, modificaremos
`handle_connection` como se muestra en el Listado 20-5 para leer el archivo
HTML, agregarlo a la respuesta como un cuerpo y enviarlo.

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::io::prelude::*;
# use std::net::TcpStream;
use std::fs::File;
// --snip--

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];
    stream.read(&mut buffer).unwrap();

    let mut file = File::open("hello.html").unwrap();

    let mut contents = String::new();
    file.read_to_string(&mut contents).unwrap();

    let response = format!("HTTP/1.1 200 OK\r\n\r\n{}", contents);

    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
```

<span class="caption">Listado 20-5: Envío de los contenidos de *hello.html*
como el cuerpo de la respuesta</span>

Hemos agregado una línea en la parte superior para poner el `File` de la
biblioteca estándar en el alcance. El código para abrir un archivo y leer los
contenidos debería ser familiar; lo usamos en el Capítulo 12 cuando leemos el
contenido de un archivo para nuestro proyecto de E/S en el Listado 12-4.

A continuación, usamos `format!` Para agregar el contenido del archivo como
el cuerpo de la respuesta de éxito.

Ejecute este código con `cargo run` y cargue *127.0.0.1:7878*
en su navegador; ¡deberías ver tu HTML renderizado!

Actualmente, ignoramos los datos de solicitud en `buffer` y simplemente
enviamos el contenido del archivo HTML incondicionalmente. Eso significa que
si intentas solicitar *127.0.0.1:7878/something-else* en tu navegador, igual
obtendrás la misma respuesta HTML. Nuestro servidor es muy limitado y no es
lo que hacen la mayoría de los servidores web. Queremos personalizar nuestras
respuestas según la solicitud y solo enviar el archivo HTML para una
solicitud bien formada a */*.

### Validar la solicitud y responder selectivamente

En este momento, nuestro servidor web devolverá el código HTML en el archivo
sin importar lo que el cliente solicite. Agreguemos funcionalidad para
verificar que el navegador esté solicitando */* antes de devolver el archivo
HTML y devuelva un error si el navegador solicita algo más. Para esto,
necesitamos modificar `handle_connection`, como se muestra en el Listado
20-6. Este nuevo código verifica el contenido de la solicitud recibida contra
lo que sabemos que parece una solicitud para */* y agrega bloques `if` y
`else` para tratar las solicitudes de manera diferente.

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::io::prelude::*;
# use std::net::TcpStream;
# use std::fs::File;
// --snip--

fn handle_connection(mut stream: TcpStream) {
    let mut buffer = [0; 512];
    stream.read(&mut buffer).unwrap();

    let get = b"GET / HTTP/1.1\r\n";

    if buffer.starts_with(get) {
        let mut file = File::open("hello.html").unwrap();

        let mut contents = String::new();
        file.read_to_string(&mut contents).unwrap();

        let response = format!("HTTP/1.1 200 OK\r\n\r\n{}", contents);

        stream.write(response.as_bytes()).unwrap();
        stream.flush().unwrap();
    } else {
        // some other request
    }
}
```

<span class="caption">Listado 20-6: Igualar las solicitudes y gestionar las
solicitudes a */* que otras solicitudes</span>

Primero, codificamos los datos correspondientes a la solicitud */* en la
variable `get`. Debido a que estamos leyendo bytes sin formato en el búfer,
transformamos `get` en una cadena de bytes agregando la sintaxis  `b""` *byte
string* al comienzo de los datos de contenido. Luego comprobamos si `buffer`
comienza con los bytes en `get`. Si lo hace, significa que hemos recibido una
solicitud bien formada para */*, que es el caso de éxito que manejaremos en
el bloque `if` que devuelve los contenidos de nuestro archivo HTML.

Si `buffer` *no* comienza*con los *bytes* en `get`, significa que hemos
recibido alguna otra solicitud. Añadiremos código al bloque `else` en un
momento para responder a todas las demás solicitudes.

Ejecute este código ahora y solicite *127.0.0.1:7878*; deberías obtener el
HTML en *hello.html*. Si realiza cualquier otra solicitud, como
*127.0.0.1:7878/something-else*, obtendrá un error de conexión como los que
vio al ejecutar el código en el Listado 20-1 y el Listado 20-2.

Ahora agreguemos el código del listado 20-7 al bloque `else` para devolver
una respuesta con el código de estado 404, que indica que no se encontró el
contenido de la solicitud. También le devolveremos algo de HTML para que una
página se represente en el navegador e indique la respuesta al usuario final.

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::io::prelude::*;
# use std::net::TcpStream;
# use std::fs::File;
# fn handle_connection(mut stream: TcpStream) {
# if true {
// --snip--

} else {
    let status_line = "HTTP/1.1 404 NOT FOUND\r\n\r\n";
    let mut file = File::open("404.html").unwrap();
    let mut contents = String::new();

    file.read_to_string(&mut contents).unwrap();

    let response = format!("{}{}", status_line, contents);

    stream.write(response.as_bytes()).unwrap();
    stream.flush().unwrap();
}
# }
```

<span class="caption">Listado 20-7: Respondiendo con el código de estado 404
y una página de error si se solicitó algo más que */*</span>

Aquí, nuestra respuesta tiene una línea de estado con el código de estado 404
y la frase de razón `NOT FOUND`. Todavía no estamos devolviendo encabezados,
y el cuerpo de la respuesta será el HTML en el archivo *404.html*. Deberá
crear un archivo *404.html* junto a *hello.html* para la página de error; de
nuevo, siéntase libre de usar cualquier HTML que desee o use el HTML de
ejemplo en el Listado 20-8.

<span class="filename">Filename: 404.html</span>

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Oops!</h1>
    <p>Sorry, I don't know what you're asking for.</p>
  </body>
</html>
```

<span class="caption">Listado 20-8: muestra el contenido de la página para
enviar de vuelta con cualquier respuesta 404</span>

Con estos cambios, ejecute su servidor de nuevo. La solicitud de
*127.0.0.1:7878* debe devolver el contenido de *hello.html*, y cualquier otra
solicitud, como *127.0.0.1:7878/foo*, debe devolver el HTML de error de
*404.html*.

### Un toque de refactorización

Por el momento, los bloques `if` y `else` tienen mucha repetición: ambos leen
archivos y escriben el contenido de los archivos en la transmisión. Las
únicas diferencias son la línea de estado y el nombre del archivo. Hagamos
que el código sea más conciso sacando esas diferencias en líneas separadas
`if` y `else` que asignarán los valores de la línea de estado y el nombre de
archivo a las variables; entonces podemos usar esas variables
incondicionalmente en el código para leer el archivo y escribir la respuesta.
El listado 20-9 muestra el código resultante después de reemplazar los
bloques grandes `if` y `else`.

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::io::prelude::*;
# use std::net::TcpStream;
# use std::fs::File;
// --snip--

fn handle_connection(mut stream: TcpStream) {
#     let mut buffer = [0; 512];
#     stream.read(&mut buffer).unwrap();
#
#     let get = b"GET / HTTP/1.1\r\n";
    // --snip--

    let (status_line, filename) = if buffer.starts_with(get) {
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

<span class="caption">Listado 20-9: Refactorizando los bloques `if` y `else`
para contener solo el código que difiere entre los dos casos</span>

Ahora los bloques `if` y `else` solo devuelven los valores apropiados para la
línea de estado y el nombre de archivo en una tupla; luego usamos la
desestructuración para asignar estos dos valores a `status_line` y `filename`
usando un patrón en la declaración `let`, como se discutió en el Capítulo 18.

El código previamente duplicado está ahora fuera de los bloques `if` y `else`
y usa las variables `status_line` y `filename`. Esto hace que sea más fácil
ver la diferencia entre los dos casos, y significa que tenemos un solo lugar
para actualizar el código si queremos cambiar el funcionamiento de la lectura
de archivos y la escritura de respuestas. El comportamiento del código en el
listado 20-9 será el mismo que en el listado 20-8.

¡Increíble! Ahora tenemos un servidor web simple en aproximadamente 40
líneas de código Rust que responde a una solicitud con una página de
contenido y responde a todas las demás solicitudes con una respuesta 404.

Actualmente, nuestro servidor se ejecuta en un único hilo, lo que significa
que solo puede servir una solicitud a la vez. Examinemos cómo puede ser un
problema simulando algunas solicitudes lentas. Luego lo arreglaremos para que
nuestro servidor pueda manejar múltiples solicitudes a la vez.
