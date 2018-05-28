## *Unsafe* (*inseguro*) Rust

Todo el código que hemos discutido hasta ahora ha tenido las garantías de
seguridad de la memoria de Rust aplicadas en tiempo de compilación. Sin
embargo, Rust tiene un segundo lenguaje oculto dentro que no impone estas
garantías de seguridad de la memoria: se llama *Rust inseguro*
(*unsafe Rust*) y funciona igual que el Rust normal, pero nos da superpoderes
adicionales.

Rust inseguro existe porque, por naturaleza, el análisis estático es
conservador. Cuando el compilador intenta determinar si el código mantiene o
no las garantías, es mejor que rechace algunos programas válidos en lugar de
aceptar algunos programas no válidos. Aunque el código podría estar bien, por
lo que Rust puede decir, ¡no lo es!. En estos casos, puede usar un código
inseguro para decirle al compilador: “Créame, sé lo que estoy haciendo”. Lo
malo es que lo usa bajo su propio riesgo: si utiliza el código inseguro
incorrectamente, pueden ocurrir problemas debido a la inseguridad de la
memoria, como la desreferenciación de punteros nulos.

Otra razón por la que Rust tiene un alter ego inseguro es que el hardware
subyacente de la computadora es inherentemente inseguro. Si Rust no le
permitió hacer operaciones inseguras, no podría hacer ciertas tareas. Rust
debe permitirle realizar una programación de sistemas de bajo nivel, como
interactuar directamente con el sistema operativo o incluso escribir su
propio sistema operativo. Trabajar con sistemas de programación de bajo nivel
es uno de los objetivos del lenguaje. Exploremos qué podemos hacer con Rust
inseguro y cómo hacerlo.

### Unsafe Superpowers

Para cambiar a Rust inseguro, use la palabra clave `unsafe` y luego comience
un nuevo bloque que contenga el código inseguro. Puedes tomar cuatro acciones
en Rust inseguro, llamadas *unsafe superpowers*, que no puedes usar en Rust seguro. Esos *Superpowers* incluyen la capacidad de:

* Desreferenciar un *raw pointer*
* Llamar a una función o método inseguro
* Accede o modifica una variable estática mutable
* Implementar un *trait* inseguro

Es importante comprender que `unsafe` no apaga el comprobador de préstamos ni
desactiva ninguna otra de las verificaciones de seguridad de Rust: si utiliza
una referencia en un código inseguro, seguirá siendo revisado. La palabra
clave `unsafe` solo le da acceso a estas cuatro características que el
compilador no verifica para la seguridad de la memoria. Todavía obtendrá un
cierto grado de seguridad dentro de un bloque inseguro.

Además, `unsafe` no significa que el código dentro del bloque sea
necesariamente peligroso o que definitivamente tenga problemas de seguridad
en la memoria: la intención es que, como programador, se asegure de que el
código dentro de un bloque `unsafe` acceda a la memoria de una manera válida.

Las personas son falibles y se producirán errores, pero al requerir que estas
cuatro operaciones inseguras estén dentro de bloques anotados con `unsafe`,
sabrá que cualquier error relacionado con la seguridad de la memoria debe
estar dentro de un bloque `unsafe`. Mantenga los bloques `unsafe` pequeños;
Estarás agradecido más tarde cuando investigue errores de memoria.

Para aislar el código inseguro tanto como sea posible, es mejor incluir el
código no seguro dentro de una abstracción segura y proporcionar una API
segura, que veremos más adelante en el capítulo cuando examinemos funciones y
métodos inseguros. Partes de la biblioteca estándar se implementan como
abstracciones seguras sobre el código inseguro que se ha auditado. Envolver
el código no seguro en una abstracción segura evita que los usos de
`unsafe` se filtren en todos los lugares donde usted o sus usuarios puedan
querer usar la funcionalidad implementada con el código `unsafe`, porque usar
una abstracción segura es seguro.

Miremos cada una de las cuatro *superpowers unsafe* a su vez. También veremos
algunas abstracciones que proporcionan una interfaz segura para el código
inseguro.

### Desreferenciar un *Raw Pointer*

En el Capítulo 4, en la sección “*Dangling* que cuelgan”, mencionamos que el
compilador asegura que las referencias siempre son válidas. *Unsafe* Rust
tiene dos nuevos tipos llamados *raw pointers* que son similares a las
referencias. Al igual que con las referencias, los punteros crudos pueden ser
inmutables o mutables y se escriben como `*const T` y `*mut T`,
respectivamente. El asterisco no es el operador de desreferencia; es parte
del nombre del tipo. En el contexto de los punteros sin formato,
*inmutables* significa que el puntero no puede asignarse directamente después
de haber sido desreferenciados.

A diferencia de las referencias y los punteros inteligentes, los punteros sin
formato:

* Se les permite ignorar las reglas de préstamo al tener punteros inmutables
 y mutables o múltiples punteros mutables en la misma ubicación
* No se garantiza que apunte a la memoria válida
* Pueden ser nulos
* No implemente ninguna limpieza automática

Al optar por evitar que Rust haga cumplir estas garantías, puede renunciar a
la seguridad garantizada a cambio de un mayor rendimiento o la capacidad de
interactuar con otro lenguaje o hardware donde las garantías de Rust no se
apliquen.

El listado 19-1 muestra cómo crear un *raw pointer* inmutable y mutable a
partir de las referencias.

```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;
```

<span class="caption">Listado 19-1: Creación de *raw pointers* a partir de
referencias</span>

Tenga en cuenta que no incluimos la palabra clave `unsafe` en este código.
Podemos crear *raw pointers* en código seguro; simplemente no podemos
desreferenciar *raw pointers* fuera de un bloque `unsafe`, como verán en un
momento.

Hemos creado *raw pointers* utilizando `as` para convertir una referencia
inmutable y mutable en sus correspondientes tipos de *raw pointers*. Debido a
que los creamos directamente a partir de referencias que se garantiza que son
válidas, sabemos que estos *raw pointers* particulares son válidos, pero no
podemos hacer esa suposición sobre cualquier *raw pointers*.

A continuación, crearemos un *raw pointers* cuya validez no podemos estar tan
seguros. El Listado 19-2 muestra cómo crear un *raw pointers* a una ubicación
arbitraria en la memoria. Intentar usar memoria arbitraria no está definido:
puede haber datos en esa dirección o puede que no, el compilador podría
optimizar el código para que no haya acceso a la memoria, o el programa
podría generar un error con un error de segmentación. Por lo general, no hay
una buena razón para escribir código como este, pero es posible.

```rust
let address = 0x012345usize;
let r = address as *const i32;
```

<span class="caption">Listado 19-2: Creación de un *raw pointer* a una
dirección de memoria arbitraria</span>

Recuerde que podemos crear *raw pointers* en código seguro, pero no podemos
*desreferenciar*, *raw pointers* y leer los datos que se apuntan. En el
listado 19-3, usamos el operador de desreferencia `*` en un *raw pointers*
que requiere un bloque `unsafe`.

```rust
let mut num = 5;

let r1 = &num as *const i32;
let r2 = &mut num as *mut i32;

unsafe {
    println!("r1 is: {}", *r1);
    println!("r2 is: {}", *r2);
}
```

<span class="caption">Listado 19-3: Cómo desreferenciar *raw pointers*
dentro de un bloque `unsafe`</span>

Crear un puntero no hace daño; solo cuando tratamos de acceder al valor que
señala, podríamos terminar lidiando con un valor no válido.

Tenga en cuenta también que en el listado 19-1 y 19-3, creamos *raw pointers* `*const i32` y `*mut i32` que apuntaban a la misma ubicación de memoria,
donde `num` está almacenado. Si en lugar de eso intentamos crear una
referencia inmutable y mutable a `num`, el código no se habría compilado
porque las reglas de propiedad de Rust no permiten una referencia mutable al
mismo tiempo que cualquier referencia inmutable. Con los *raw pointers*,
podemos crear un puntero mutable y un puntero inmutable en la misma ubicación
y cambiar los datos a través del puntero mutable, lo que podría crear una
carrera de datos. ¡Tenga cuidado!

Con todos estos peligros, ¿por qué usaría *raw pointers*?. Un caso de uso
importante es cuando se interactúa con el código C, como verá en la siguiente
sección, “Llamar a una función o método inseguro”. Otro caso es cuando se
crean abstracciones seguras que el verificador de préstamos no entiende.
Introduciremos funciones inseguras y luego veremos un ejemplo de abstracción
segura que usa código inseguro.

### Llamar a una función o método inseguro

El segundo tipo de operación que requiere un bloque inseguro es llamadas a
funciones inseguras. Las funciones y métodos inseguros se ven exactamente
como las funciones y los métodos normales, pero tienen un `unsafe` adicional
antes del resto de la definición. La palabra clave `unsafe` en este contexto
indica que la función tiene requisitos que debemos mantener cuando llamamos a
esta función, porque Rust no puede garantizar que cumplamos con estos
requisitos. Al llamar a una función insegura dentro de un bloque `unsafe`,
estamos diciendo que hemos leído la documentación de esta función y asumimos
la responsabilidad de mantener los contratos de la función.

Aquí hay una función insegura llamada `dangerous` que no hace nada en su
cuerpo:

```rust
unsafe fn dangerous() {}

unsafe {
    dangerous();
}
```

Debemos llamar a la función `dangerous` dentro de un bloque `unsafe`
separado. Si tratamos de llamar `dangerous` sin el bloque `unsafe`,
obtendremos un error:

```text
error[E0133]: call to unsafe function requires unsafe function or block
 -->
  |
4 |     dangerous();
  |     ^^^^^^^^^^^ call to unsafe function
```

Al insertar el bloque `unsafe` en nuestra llamada a `dangerous`, le estamos
diciendo a Rust que hemos leído la documentación de la función, que sabemos
cómo usarla correctamente y que hemos verificado que estamos cumpliendo el
contrato. de la función.

Los cuerpos de funciones inseguras son efectivamente bloques `unsafe`, por lo
que para realizar otras operaciones inseguras dentro de una función insegura,
no es necesario agregar otro bloque `unsafe`.

#### Creando una abstracción segura sobre un código inseguro

El hecho de que una función contenga un código inseguro no significa que
tengamos que marcar toda la función como insegura. De hecho, envolver código
inseguro en una función segura es una abstracción común. Como ejemplo,
estudiemos una función de la biblioteca estándar, `split_at_mut`, que
requiere algún código inseguro y explore cómo podemos implementarlo. Este
método seguro se define en porciones mutables: toma una porción y la
convierte en dos dividiendo la porción en el índice dado como argumento. El
listado 19-4 muestra cómo usar `split_at_mut`.

```rust
let mut v = vec![1, 2, 3, 4, 5, 6];

let r = &mut v[..];

let (a, b) = r.split_at_mut(3);

assert_eq!(a, &mut [1, 2, 3]);
assert_eq!(b, &mut [4, 5, 6]);
```

<span class="caption">Listado 19-4: Uso de la función segura
`split_at_mut`</span>

No podemos implementar esta función usando solo Rust seguro. Un intento puede
parecerse al Listado 19-5, que no se compilará. Para simplificar,
implementaremos `split_at_mut` como una función en lugar de un método y solo
para segmentos de valores `i32` en lugar de para un tipo genérico `T`.

```rust,ignore
fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();

    assert!(mid <= len);

    (&mut slice[..mid],
     &mut slice[mid..])
}
```

<span class="caption">Listado 19-5: Un intento de implementación de
`split_at_mut` usando solo Rust seguro</span>

Esta función primero obtiene la longitud total del *slice*. Luego afirma que
el índice dado como parámetro está dentro del *slice* al verificar si es
menor o igual a la longitud. La afirmación significa que si pasamos un índice
que es mayor que el índice para dividir el *slice*, la función entrará en
pánico antes de intentar usar ese índice.

Luego devolvemos dos *slice* mutables en una tupla: una desde el inicio del
*slice* original al índice `mid` y otra desde `mid` hasta el final del
*slice*.

Cuando intentemos compilar el código en el listado 19-5, obtendremos un error.

```text
error[E0499]: cannot borrow `*slice` as mutable more than once at a time
 -->
  |
6 |     (&mut slice[..mid],
  |           ----- first mutable borrow occurs here
7 |      &mut slice[mid..])
  |           ^^^^^ second mutable borrow occurs here
8 | }
  | - first borrow ends here
```

El comprobador de préstamos de Rust no puede entender que estamos tomando
prestadas partes diferentes del *slice*; solo sabe que tomamos prestado del misma *slice* dos veces. Pedir prestado diferentes partes de un *slice* es
fundamentalmente correcto porque los dos *slice* no se superponen, pero Rust
no es lo suficientemente inteligente como para saberlo. Cuando sabemos que el
código está bien, pero Rust no, es hora de buscar un código inseguro.

El Listado 19-6 muestra cómo utilizar un bloque `unsafe`, un *raw pointer* y
algunas llamadas a funciones inseguras para hacer que la implementación de
`split_at_mut` funcione.

```rust
use std::slice;

fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    let ptr = slice.as_mut_ptr();

    assert!(mid <= len);

    unsafe {
        (slice::from_raw_parts_mut(ptr, mid),
         slice::from_raw_parts_mut(ptr.offset(mid as isize), len - mid))
    }
}
```

<span class="caption">Listado 19-6: Usar código inseguro en la implementación
de la función `split_at_mut`</span>

Recuerde de la sección “Tipo de *Slice*” en el Capítulo 4 que los *slices*
son un puntero a algunos datos y la longitud del *slices*. Utilizamos el
método `len` para obtener la longitud de un *slices* y el método `as_mut_ptr`
para acceder al puntero sin formato de un *slices*. En este caso, como
tenemos un *slices* mutable a valores `i32`, `as_mut_ptr` devuelve un puntero
sin formato con el tipo `*mut i32`, que hemos almacenado en la variable `ptr`.

Mantenemos la afirmación de que el índice `mid` está dentro del *slices*.
Luego llegamos al código inseguro: la función `slice::from_raw_parts_mut`
toma un puntero sin formato y una longitud, y crea un *slices*. Usamos esta
función para crear un *slices* que comienza desde `ptr` y es `mid` largo.
Luego llamamos al método `offset` en `ptr` con `mid` como argumento para
obtener un puntero sin formato que comience en `mid`, y creamos un *slices*
usando ese puntero y el número restante de elementos después de `mid` como la
longitud.

La función `slice::from_raw_parts_mut` no es segura porque toma un puntero
sin formato y debe confiar en que este puntero es válido. El método `offset`
en punteros sin formato también es inseguro, porque debe confiar en que la
ubicación del desplazamiento también es un puntero válido. Por lo tanto,
tuvimos que poner un bloque `unsafe` alrededor de nuestras llamadas a
`slice::from_raw_parts_mut` y `offset` para que pudiéramos llamarlas. Al
observar el código y al agregar la afirmación de que `mid` debe ser menor o
igual que `len`, podemos decir que todos los *raw pointers* utilizados dentro
del bloque `unsafe` serán punteros válidos para los datos dentro del
*slice*. Este es un uso aceptable y apropiado de `unsafe`.

Tenga en cuenta que no necesitamos marcar la función `split_at_mut`
resultante como `unsafe`, y podemos llamar a esta función desde Rust seguro.
Hemos creado una abstracción segura del código inseguro con una
implementación de la función que usa el código `unsafe` de forma segura, ya
que solo crea punteros válidos a partir de los datos a los que tiene acceso
esta función.

Por el contrario, el uso de `slice::from_raw_parts_mut` en el Listado 19-7
probablemente se bloquee cuando se use el *slice*. Este código toma una
ubicación de memoria arbitraria y crea un *slice* de 10.000 elementos de
longitud.

```rust
use std::slice;

let address = 0x012345usize;
let r = address as *mut i32;

let slice = unsafe {
    slice::from_raw_parts_mut(r, 10000)
};
```

<span class="caption">Listado 19-7: Crear un *slice* desde una ubicación de
memoria arbitraria</span>

No poseemos la memoria en esta ubicación arbitraria, y no hay garantía de que
el *slice* que crea este código contenga valores `i32` válidos. Intentar
utilizar el `slice` como si fuera un *slice* válido da como resultado un
comportamiento indefinido.

#### Uso de funciones `extern` para llamar al código externo

A veces, su código Rust podría necesitar interactuar con código escrito en
otro lenguaje. Para esto, Rust tiene una palabra clave, `extern`, que
facilita la creación y el uso de una *Foreign Function Interface (FFI)*. Un
FFI es una forma para que un lenguaje de programación defina funciones y
habilite un lenguaje de programación diferente (extranjero) para llamar a
esas funciones.

El listado 19-8 muestra cómo configurar una integración con la función `abs`
de la biblioteca estándar C. Las funciones declaradas dentro de bloques
`extern` siempre son inseguras para llamar desde el código Rust. La razón es
que otros lenguajes no hacen cumplir las reglas y garantías de Rust, y Rust
no puede verificarlos, por lo que la responsabilidad recae en el programador
para garantizar la seguridad.

<span class="filename">Filename: src/main.rs</span>

```rust
extern "C" {
    fn abs(input: i32) -> i32;
}

fn main() {
    unsafe {
        println!("Absolute value of -3 according to C: {}", abs(-3));
    }
}
```

<span class="caption">Listado 19-8: Declarar y llamar a una función `extern`
definida en otro lenguaje</span>

Dentro del bloque `extern" C "`, enumeramos los nombres y firmas de funciones
externas de otro lenguaje que queremos llamar. La parte `"C"` define qué
*application binary interface (ABI)* (*interfaz binaria de aplicación (ABI)*)
utiliza la función externa: el ABI define cómo llamar a la función en el
nivel de ensamblaje. El `"C"` ABI es el más común y sigue el ABI del lenguaje
de programación C.

> #### Llamar a funciones de Rust desde otros lenguajes
>
> También podemos usar `extern` para crear una interfaz que permita a otros
> lenguajes llamar a las funciones de Rust. En lugar de un bloque `extern`,
> agregamos la palabra clave `extern` y especificamos el ABI para usar justo
> antes de la palabra clave `fn`. También necesitamos agregar una anotación
>`#[no_mangle]` para decirle al compilador de Rust que no modifique el nombre
> de esta función. *Mangling* es cuando un compilador cambia el nombre al que
> hemos asignado una función por un nombre diferente que contiene más
> información para que otras partes del proceso de compilación lo consuman,
> pero es menos legible para el ser humano. Cada compilador de lenguaje de
> programación manipula los nombres de forma ligeramente diferente, por lo
> que para que una función Rust sea reconocible por otros lenguajes, debemos
> deshabilitar el nombre del compilador Rust.
>
> En el siguiente ejemplo, hacemos que la función `call_from_c` sea accesible
> desde el código C, después de compilarla en una biblioteca compartida y
> vincularla desde C:
>
> ```rust
> #[no_mangle]
> pub extern "C" fn call_from_c() {
>     println!("Just called a Rust function from C!");
> }
> ```
>
> Este uso de `extern` no requiere `unsafe`.

### Acceder o modificar una variable estática mutable

Hasta ahora, no hemos hablado sobre *variables globales*, que Rust sí admite,
pero puede ser problemático con las reglas de propiedad de Rust. Si dos hilos
están accediendo a la misma variable global mutable, puede causar una carrera
de datos.

En Rust, las variables globales se llaman *variables estáticas*. El listado
19-9 muestra una declaración de ejemplo y el uso de una variable estática con
un *string slice* como un valor.

<span class="filename">Filename: src/main.rs</span>

```rust
static HELLO_WORLD: &str = "Hello, world!";

fn main() {
    println!("name is: {}", HELLO_WORLD);
}
```

<span class="caption">Listado 19-9: Definición y uso de una variable estática
inmutable</span>

Las variables estáticas son similares a las constantes, que discutimos en la
sección “Diferencias entre variables y constantes” en el Capítulo 3. Los
nombres de las variables estáticas están en `SCREAMING_SNAKE_CASE` por
convención, y debemos *anotar* el tipo de la variable, que es `&'static str`
en este ejemplo. Las variables estáticas solo pueden almacenar referencias
con el *lifetime* `'static`, lo que significa que el compilador de Rust puede
calcular la duración; no necesitamos anotarlo explícitamente. El acceso a una
variable estática inmutable es seguro.

Las constantes y las variables estáticas inmutables pueden parecer similares,
pero una diferencia sutil es que los valores en una variable estática tienen
una dirección fija en la memoria. El uso del valor siempre tendrá acceso a
los mismos datos. Las constantes, por otro lado, pueden duplicar sus datos
siempre que se utilicen.

Otra diferencia entre las constantes y las variables estáticas es que las
variables estáticas pueden ser mutables. El acceso y la modificación de
variables estáticas mutables es *inseguro*. El listado 19-10 muestra cómo
declarar, acceder y modificar una variable estática mutable llamada `COUNTER`.

<span class="filename">Filename: src/main.rs</span>

```rust
static mut COUNTER: u32 = 0;

fn add_to_count(inc: u32) {
    unsafe {
        COUNTER += inc;
    }
}

fn main() {
    add_to_count(3);

    unsafe {
        println!("COUNTER: {}", COUNTER);
    }
}
```

<span class="caption">Listado 19-10: leer o escribir en una variable estática
mutable es inseguro</span>

Al igual que con las variables regulares, especificamos la mutabilidad usando
la palabra clave `mut`. Cualquier código que lea o escriba de `COUNTER` debe
estar dentro de un bloque `unsafe`. Este código compila e imprime
`COUNTER: 3` como se esperaría porque tiene un solo hilo. Tener múltiples
hilos de acceso `COUNTER` probablemente resultaría en carreras de datos.

Con los datos mutables que son accesibles a nivel mundial, es difícil
garantizar que no haya carreras de datos, razón por la cual Rust considera
que las variables estáticas mutables no son seguras. Siempre que sea posible,
es preferible utilizar las técnicas de concurrencia y *thread-safe smart
pointers* que discutimos en el Capítulo 16, de modo que el compilador
compruebe que los datos a los que se accede desde diferentes subprocesos se
realizan de forma segura.

### Implementando un *Trait* Inseguro

La acción final que solo funciona con `unsafe` está implementando un *trait*
inseguro. Un *trait* no es seguro cuando al menos uno de sus métodos tiene
alguna invariante que el compilador no puede verificar. Podemos declarar que
un *trait* es `unsafe` añadiendo la palabra clave `unsafe` antes de `trait` y
marcando la implementación del `trait` como `unsafe` también, como se muestra
en el Listado 19-11.

```rust
unsafe trait Foo {
    // methods go here
}

unsafe impl Foo for i32 {
    // method implementations go here
}
```

<span class="caption">Listado 19-11: Definición e implementación de un *trait*
inseguro</span>

Al utilizar `unsafe impl`, prometemos que mantendremos las invariantes que el
compilador no puede verificar.

Como ejemplo, recuerde los *trait* del marcador `Sync` y `Send` que
discutimos en la sección “Concurrencia extensible con los *Traits*`Sync` y
`Send`” en el Capítulo 16: el compilador implementa estos *trait*
automáticamente si nuestros tipos están compuestos completamente de los tipos
`Send` y `Sync`. Si implementamos un tipo que contiene un tipo que no es
`Send` o `Sync`, como *raw pointers*, y queremos marcar ese tipo como `Send`
o `Sync`, debemos usar `unsafe`. Rust no puede verificar que nuestro tipo
mantenga las garantías de que puede enviarse de manera segura a través de
hilos o acceder desde múltiples hilos; por lo tanto, necesitamos hacer esos
controles manualmente e indicar como tal con `unsafe`.

### Cuándo usar el código no seguro

Usar `unsafe` para tomar una de las cuatro acciones (*superpoderes*)
(*superpowers*) que acabamos de discutir no es incorrecto o incluso
desaprobado. Pero es más complicado obtener el código `unsafe` correcto
porque el compilador no puede ayudar a mantener la seguridad de la memoria.
Cuando tiene una razón para usar el código `unsafe`, puede hacerlo, y tener
la anotación `unsafe` explícita hace que sea más fácil rastrear el origen de
los problemas si ocurren.
