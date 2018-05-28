## Closures: Funciones anónimas que pueden capturar su entorno

Los *closures* de Rust son funciones anónimas que puede guardar en una
variable o pasar como argumentos a otras funciones. Puede crear el *closures*
en un lugar y luego llamar al *closures* para evaluarlo en un contexto
diferente. A diferencia de las funciones, los *closures* pueden capturar
valores del ámbito en el que se llaman. Demostraremos cómo estas
características de *closures* permiten la reutilización del código y la
personalización del comportamiento.

### Creando una abstracción de comportamiento con *Closures*

Vamos a trabajar en un ejemplo de una situación en la que es útil almacenar
un *closures* que se ejecutará más adelante. En el camino, hablaremos sobre
la sintaxis de los *closures*, la inferencia tipo y los *trait*.

Considere esta situación hipotética: trabajamos en una startup que está
creando una aplicación para generar planes de entrenamiento de ejercicios
personalizados. El backend está escrito en Rust y el algoritmo que genera el
plan de entrenamiento tiene en cuenta muchos factores, como la edad del
usuario de la aplicación, el índice de masa corporal, las preferencias de
ejercicio, entrenamientos recientes y un número de intensidad que
especifican. El algoritmo real utilizado no es importante en este ejemplo; lo
importante es que este cálculo demora unos segundos. Queremos llamar a este
algoritmo solo cuando lo necesitemos y solo llamarlo una vez para que el
usuario no espere más de lo necesario.

Simularemos llamar a este algoritmo hipotético con la función
`simulated_expensive_calculation` que se muestra en el listado 13-1, que
imprimirá `calculating slowly...`, esperará dos segundos y luego
devolverá el número que hayamos pasado.

<span class="filename">Filename: src/main.rs</span>

```rust
use std::thread;
use std::time::Duration;

fn simulated_expensive_calculation(intensity: u32) -> u32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    intensity
}
```

<span class="caption">Listado 13-1: una función para sustituir un cálculo
hipotético que tarda unos 2 segundos en ejecutarse</span>

Luego está la función `main`, que contiene las partes de la aplicación de
entrenamiento importantes para este ejemplo. Esta función representa el
código que la aplicación llamará cuando un usuario solicite un plan de
entrenamiento. Debido a que la interacción con la interfaz de la aplicación
no es relevante para el uso de *closures*, codificaremos los valores que
representan las entradas de nuestro programa e imprimiremos los resultados.

Las entradas requeridas son estas:

* Un número de intensidad del usuario, que se especifica cuando solicitan un
 entrenamiento para indicar si desean un entrenamiento de baja intensidad o
 un entrenamiento de alta intensidad
* Un número aleatorio que generará cierta variedad en los planes de
 entrenamiento

El resultado será el plan de entrenamiento recomendado. El listado 13-2
muestra la función `main` que usaremos.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let simulated_user_specified_value = 10;
    let simulated_random_number = 7;

    generate_workout(
        simulated_user_specified_value,
        simulated_random_number
    );
}
# fn generate_workout(intensity: u32, random_number: u32) {}
```

<span class="caption">Listado 13-2: Una función `main` con valores
codificados para simular la entrada del usuario y la generación de números
aleatorios</span>

Hemos codificado la variable `simulated_user_specified_value` como 10 y la
variable `simulated_random_number` como 7 por simplicidad; en un programa
real, obtendríamos el número de intensidad de la interfaz de la aplicación, y
usaríamos la caja `rand` para generar un número aleatorio, como lo hicimos en
el ejemplo del juego Adivinar en el Capítulo 2. La función `main` llama a la
función `generate_workout` con los valores de entrada simulados.

Ahora que tenemos el contexto, vamos al algoritmo. La función
`generate_workout` en el Listado 13-3 contiene la lógica comercial de la
aplicación que más nos interesa en este ejemplo. El resto de los cambios de
código en este ejemplo se realizarán en esta función.

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
# fn simulated_expensive_calculation(num: u32) -> u32 {
#     println!("calculating slowly...");
#     thread::sleep(Duration::from_secs(2));
#     num
# }
#
fn generate_workout(intensity: u32, random_number: u32) {
    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            simulated_expensive_calculation(intensity)
        );
        println!(
            "Next, do {} situps!",
            simulated_expensive_calculation(intensity)
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                simulated_expensive_calculation(intensity)
            );
        }
    }
}
```

<span class="caption">Listado 13-3: La lógica de negocios que imprime los
planes de entrenamiento basados en las entradas y llamadas a la función
`simulated_expensive_calculation`</span>

El código en el Listado 13-3 tiene múltiples llamadas a la función de cálculo
lento. El primer bloque `if` invoca `simulated_expensive_calculation` dos
veces, el `if` dentro del `else` externo no lo llama en absoluto, y el código
dentro del segundo caso `else` lo llama una vez.

<!-- NEXT PARAGRAPH WRAPPED WEIRD INTENTIONALLY SEE #199 -->

El comportamiento deseado de la función `generate_workout` es verificar
primero si el usuario desea un entrenamiento de baja intensidad (indicado por
un número menor a 25) o un entrenamiento de alta intensidad (un número de 25
o más).

Los planes de entrenamiento de baja intensidad recomendarán una serie de
flexiones y sentadillas basadas en el complejo algoritmo que estamos
simulando.

Si el usuario desea un entrenamiento de alta intensidad, existe una lógica
adicional: si el valor del número aleatorio generado por la aplicación es 3,
la aplicación recomendará un descanso e hidratación. De lo contrario, el
usuario obtendrá una cantidad de minutos de ejecución basada en el complejo
algoritmo.

Este código funciona de la forma en que la empresa lo quiere ahora, pero
digamos que el equipo de ciencia de datos decide que tenemos que hacer
algunos cambios en la forma en que llamamos a la función
`simulated_expensive_calculation` en el futuro. Para simplificar la
actualización cuando ocurren esos cambios, queremos refactorizar este código
para que llame a la función `simulated_expensive_calculation` solo una vez.
También queremos reducir el lugar en el que estamos innecesariamente llamando
a la función dos veces sin agregar otras llamadas a esa función en el
proceso. Es decir, no queremos llamarlo si el resultado no es necesario y aún
así queremos llamarlo solo una vez.

#### Refactorización mediante funciones

Podríamos reestructurar el programa de entrenamiento de muchas maneras.
Primero, intentaremos extraer la llamada duplicada a la función
`simulated_expensive_calculation` en una variable, como se muestra en el
Listado 13-4.

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
# fn simulated_expensive_calculation(num: u32) -> u32 {
#     println!("calculating slowly...");
#     thread::sleep(Duration::from_secs(2));
#     num
# }
#
fn generate_workout(intensity: u32, random_number: u32) {
    let expensive_result =
        simulated_expensive_calculation(intensity);

    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            expensive_result
        );
        println!(
            "Next, do {} situps!",
            expensive_result
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_result
            );
        }
    }
}
```

<span class="caption">Listado 13-4: Extrayendo las llamadas a
`simulated_expensive_calculation` a un lugar y almacenando el resultado en la
variable `expensive_result`</span>

Este cambio unifica todas las llamadas a `simulation_expensive_calculation` y
resuelve el problema del primer bloque `if` invocando innecesariamente la
función dos veces. Desafortunadamente, ahora estamos llamando a esta función
y esperando el resultado en todos los casos, que incluye el bloque `if`
interno que no usa el valor del resultado en absoluto.

Queremos definir el código en un lugar en nuestro programa, pero solo
*ejecutar* ese código donde realmente necesitamos el resultado. ¡Este es un
caso de uso para *closures*!

#### Refactorización con *Closures* para almacenar código

En lugar de llamar siempre a la función `simulated_expensive_calculation`
antes de los bloques `if`, podemos definir un *closure* y almacenar *closure*
en una variable en lugar de almacenar el resultado de la llamada a la función
como se muestra en el Listado 13-5. Podemos mover todo el cuerpo de
`simulated_expensive_calculation` dentro del *closure* que estamos
presentando aquí.

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
let expensive_closure = |num| {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
# expensive_closure(5);
```

<span class="caption">Listado 13-5: Definir un *closure* y almacenarlo en la
variable `expensive_closure`</span>

La definición de *closure* viene después de `=` para asignarla a la variable
`expensive_closure`. Para definir un *closure*, comenzamos con un par de
verticales pipes (`|`), dentro de los cuales especificamos los parámetros
para el *closure*; esta sintaxis fue elegido debido a su similitud con las
definiciones de *closure* en Smalltalk y Ruby. Este *closure* tiene un
parámetro llamado `num`: si tuviéramos más de uno
parámetro, los separaríamos con comas, como `| param1, param2 |`.

Después de los parámetros, colocamos llaves que sostienen el cuerpo del
*closure*: estos son opcionales si el cuerpo del *closure* es una sola
expresión. El fin del *closure*, después de las llaves, necesita un punto y
coma para completar el `let` declaración. El valor devuelto desde la última
línea en el cuerpo del *closure* (`num`) será el valor devuelto por el
*closure* cuando se llame, porque esa línea no termina en punto y coma; al
igual que en los cuerpos funcionales.

Tenga en cuenta que esta declaración `let` significa `expensive_closure`
contiene el *definición* de una función anónima, no el *valor resultante* de
llamar al función anónima. Recuerde que estamos usando un *closure* porque
queremos definir el código para llamar en un punto, almacenar ese código y
llamarlo en un momento posterior;
el código que queremos llamar ahora está almacenado en `expensive_closure`.

Con el *closure* definido, podemos cambiar el código en los bloques `if` para
llamar al *closure* para ejecutar el código y obtener el valor resultante.
Llamamos a un *closure* como hacemos una función: especificamos el nombre de
la variable que contiene el *closure* definición y seguirlo con paréntesis
que contienen los valores del argumento que desea usar, como se muestra en el
listado 13-6.

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
fn generate_workout(intensity: u32, random_number: u32) {
    let expensive_closure = |num| {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    };

    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            expensive_closure(intensity)
        );
        println!(
            "Next, do {} situps!",
            expensive_closure(intensity)
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_closure(intensity)
            );
        }
    }
}
```

<span class="caption">Listado 13-6: Llamar al `expensive_closure` que hemos
definido</span>

Ahora el costoso cálculo se realiza en un solo lugar, y solo estamos
ejecutando ese código donde necesitamos los resultados.

Sin embargo, hemos reintroducido uno de los problemas del Listado 13-3:
todavía estamos llamando al *closure* dos veces en el primer bloque `if`, que
llamará al código caro dos veces y hará que el usuario espere el doble del
tiempo que lo necesitan a. Podríamos solucionar este problema creando una
variable local para ese bloque `if` para contener el resultado de llamar al
*closure*, pero los *closures* nos proporcionan otra solución. Hablaremos de
esa solución en un momento. Pero primero hablemos sobre por qué no hay
anotaciones de tipo en la definición de *closure* y las características
involucradas con los *closures*.

### *Closure* tipo de inferencia y anotación

Los *closures* no requieren que anote los tipos de los parámetros o el valor
de retorno como lo hacen las funciones `fn`. Las anotaciones de tipo son
necesarias en las funciones porque son parte de una interfaz explícita
expuesta a sus usuarios. Definir esta interfaz de manera rígida es importante
para garantizar que todos estén de acuerdo con los tipos de valores que usa y
devuelve una función. Pero los *closures* no se usan en una interfaz expuesta
como esta: se almacenan en variables y se usan sin nombrarlas y exponerlas a
los usuarios de nuestra biblioteca.

Los *closures* generalmente son cortos y relevantes solo dentro de un
contexto estrecho en lugar de en cualquier escenario arbitrario. Dentro de
estos contextos limitados, el compilador puede inferir de manera confiable
los tipos de los parámetros y el tipo de retorno, de forma similar a cómo es
capaz de inferir los tipos de la mayoría de las variables.

Hacer que los programadores anoten los tipos en estas pequeñas funciones
anónimas sería molesto y en gran parte redundante con la información que el
compilador ya tiene disponible.

Al igual que con las variables, podemos agregar anotaciones de tipo si
queremos aumentar la claridad y la claridad a costa de ser más detallado de
lo estrictamente necesario. Anotar los tipos para el *closure* que definimos
en el listado 13-5 se parecería a la definición que se muestra en el listado
13-7.

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
let expensive_closure = |num: u32| -> u32 {
    println!("calculating slowly...");
    thread::sleep(Duration::from_secs(2));
    num
};
```

<span class="caption">Listado 13-7: Adición de anotaciones de tipo
opcionales del parámetro y tipos de valor de retorno en el *closure*</span>

Con las anotaciones de tipo agregadas, la sintaxis de los *closures* se
parece más a la sintaxis de las funciones. La siguiente es una comparación
vertical de la sintaxis para la definición de una función que agrega 1 a su
parámetro y un *closure* que tiene el mismo comportamiento. Hemos agregado
algunos espacios para alinear las partes relevantes. Esto ilustra cómo la
sintaxis de *closure* es similar a la sintaxis de la función, excepto por el
uso de *pipes* y la cantidad de sintaxis que es opcional:

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

La primera línea muestra una definición de función, y la segunda línea
muestra una definición de *closure* completamente anotada. La tercera línea
elimina las anotaciones de tipo de la definición de *closure*, y la cuarta
línea elimina las llaves, que son opcionales porque el cuerpo de *closure*
tiene una sola expresión. Estas son todas las definiciones válidas que
producirán el mismo comportamiento cuando se llamen.

Las definiciones de *closure* tendrán un tipo concreto inferido para cada
uno de sus parámetros y para su valor de retorno. Por ejemplo, el Listado
13-8 muestra la definición de un *closure* corto que simplemente devuelve el
valor que recibe como parámetro. Este *closure* no es muy útil excepto para
los propósitos de este ejemplo. Tenga en cuenta que no hemos agregado
ninguna anotación de tipo a la definición: si tratamos de llamar al
*closure* dos veces, utilizando un `String` como argumento la primera vez y
un `u32` la segunda vez, obtendremos un error.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
let example_closure = |x| x;

let s = example_closure(String::from("hello"));
let n = example_closure(5);
```

<span class="caption">Listado 13-8: Intentar llamar a un cierre cuyos tipos
se infieren con dos tipos diferentes</span>

El compilador nos da este error:

```text
error[E0308]: mismatched types
 --> src/main.rs
  |
  | let n = example_closure(5);
  |                         ^ expected struct `std::string::String`, found
  integral variable
  |
  = note: expected type `std::string::String`
             found type `{integer}`
```

La primera vez que llamamos `example_closure` con el valor `String`, el
compilador deduce el tipo de `x` y el tipo de retorno del *closure* para
ser `String`. Esos tipos luego se bloquean en el *closure* en
`example_closure`, y obtenemos un tipo de error si tratamos de usar un tipo
diferente con el mismo *closure*.

### Almacenamiento de *Closures* utilizando parámetros genéricos y los `Fn` *Traits*

Volvamos a nuestra aplicación de generación de entrenamiento. En el listado
13-6, nuestro código todavía estaba
llamando al costoso *closure* del cálculo más veces de las necesarias. Uno
opción para resolver este problema es guardar el resultado del *closure*
costoso en una variable para reutilizar y usar la variable en cada lugar
donde necesitamos el resultado, en lugar de llamar al *closure* nuevamente.
Sin embargo, este método podría resultar en una gran cantidad de código
repetido

Afortunadamente, otra solución está disponible para nosotros. Podemos crear
una estructura que mantendrá el *closure* y el valor resultante de llamar al
*closure*. los *struct* ejecutará el *closure* solo si necesitamos el valor
resultante, y almacenará en caché el valor resultante para que el resto de
nuestro código no tenga que ser responsable de guardar y reutilizar el
resultado. Puede conocer este patrón como *memoization* o *lazy evaluation*.

Para hacer una estructura que contenga un *closure*, necesitamos especificar
el tipo de *closure*, porque una definición de estructura necesita saber los
tipos de cada uno de sus campos. Cada instancia de *closure* tiene su propio
tipo anónimo único: es decir, incluso si dos *closure* tienen la misma firma
sus tipos aún se consideran diferente. Para definir estructuras,
enumeraciones o parámetros de funciones que usan *closure*,
usamos genéricos y *trait bounds*, como discutimos en el Capítulo 10.

Los *trait* `Fn` son proporcionados por la biblioteca estándar. Todos los
*closures* se implementan en
al menos uno de los *trait*: `Fn`,`FnMut`, o `FnOnce`. Discutiremos el
diferencia entre estos *trait* en “Capturando el entorno con *closures*” ;
en este ejemplo, podemos usar el *trait* `Fn`.

Agregamos tipos al *trait* `Fn` obligado a representar los tipos de los
parámetros y valores de retorno que los *closures* deben tener para
coincidir con este *trait* de *trait*. En este
caso, nuestro *closure* tiene un parámetro de tipo `u32` y devuelve un `u32`
por lo que
El *trait bounds* que especificamos es `Fn(u32) -> u32`.

El listado 13-9 muestra la definición de la estructura `Cacher` que contiene
un *closure* y un valor de resultado opcional.

<span class="filename">Filename: src/main.rs</span>

```rust
struct Cacher<T>
    where T: Fn(u32) -> u32
{
    calculation: T,
    value: Option<u32>,
}
```

<span class="caption">Listado 13-9: Definición de una estructura `Cacher`
que contiene un *closure* en `calculation` y un resultado opcional en
`value`</span>

La estructura `Cacher` tiene un campo de `calculation` del tipo genérico `T`.
Los *trait bounds* en `T` especifican que es un *closure* utilizando el
*trait* `Fn`. Cualquier *closure* que deseemos almacenar en el campo
`calculation` debe tener un parámetro `u32` (especificado entre paréntesis
después de `Fn` y debe devolver un `u32` (especificado después de `->`).

> Nota: Las funciones implementan los tres *trait* `Fn` también. Si lo que
> queremos hacer no requiere capturar un valor del entorno, podemos usar una
> función en lugar de un *closure* donde necesitamos algo que implemente un
> *trait* `Fn`.

El campo `value` es del tipo `Option <u32>`. Antes de ejecutar el *closure*
`value` será `None`. Cuando el código que utiliza un `Cacher` solicita el
*resultado* del *closure*, el `Cacher` ejecutará el *closure* en ese momento
y almacenará el resultado dentro de una variante `Some` en el campo `value`.
Luego, si el código solicita nuevamente el resultado del *closure*, en lugar
de ejecutar nuevamente el *closure*, el `Cacher` devolverá el resultado que
se encuentra en la variante `Some`.

La lógica alrededor del campo `valor` que acabamos de describir se define en
el listado 13-10.

<span class="filename">Filename: src/main.rs</span>

```rust
# struct Cacher<T>
#     where T: Fn(u32) -> u32
# {
#     calculation: T,
#     value: Option<u32>,
# }
#
impl<T> Cacher<T>
    where T: Fn(u32) -> u32
{
    fn new(calculation: T) -> Cacher<T> {
        Cacher {
            calculation,
            value: None,
        }
    }

    fn value(&mut self, arg: u32) -> u32 {
        match self.value {
            Some(v) => v,
            None => {
                let v = (self.calculation)(arg);
                self.value = Some(v);
                v
            },
        }
    }
}
```

<span class="caption">Listing 13-10: The caching logic of `Cacher`</span>

Queremos que `Cacher` administre los valores de los campos struct en lugar
de permitir que el código de llamada cambie potencialmente los valores en
estos campos directamente, por lo que estos campos son privados.

La función `Cacher::new` toma un parámetro genérico `T`, que hemos definido
como que tiene el mismo *trait bound* que la estructura `Cacher`. Luego
`Cacher::new` devuelve una instancia `Cacher` que contiene el *closure*
especificado en el campo `calculation` y un valor `None` en el campo `value`
porque aún no hemos ejecutado el *closure*.

Cuando el código de llamada necesita el resultado de evaluar el *closure*,
en lugar de llamar al *closure* directamente, llamará al método `value`.
Este método verifica si ya tenemos un valor resultante en `self.value` en
un `Some`; si lo hacemos, devuelve el valor dentro del `Some` sin
ejecutar el *closure* nuevamente.

Si `self.value` es `None`, el código llama al *closure* almacenado en
`self.calculation`, guarda el resultado en `self.value` para uso futuro y
también devuelve el valor.

El listado 13-11 muestra cómo podemos usar esta estructura `Cacher` en la
función `generate_workout` del Listado 13-6.

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::thread;
# use std::time::Duration;
#
# struct Cacher<T>
#     where T: Fn(u32) -> u32
# {
#     calculation: T,
#     value: Option<u32>,
# }
#
# impl<T> Cacher<T>
#     where T: Fn(u32) -> u32
# {
#     fn new(calculation: T) -> Cacher<T> {
#         Cacher {
#             calculation,
#             value: None,
#         }
#     }
#
#     fn value(&mut self, arg: u32) -> u32 {
#         match self.value {
#             Some(v) => v,
#             None => {
#                 let v = (self.calculation)(arg);
#                 self.value = Some(v);
#                 v
#             },
#         }
#     }
# }
#
fn generate_workout(intensity: u32, random_number: u32) {
    let mut expensive_result = Cacher::new(|num| {
        println!("calculating slowly...");
        thread::sleep(Duration::from_secs(2));
        num
    });

    if intensity < 25 {
        println!(
            "Today, do {} pushups!",
            expensive_result.value(intensity)
        );
        println!(
            "Next, do {} situps!",
            expensive_result.value(intensity)
        );
    } else {
        if random_number == 3 {
            println!("Take a break today! Remember to stay hydrated!");
        } else {
            println!(
                "Today, run for {} minutes!",
                expensive_result.value(intensity)
            );
        }
    }
}
```

<span class="caption">Listing 13-11: Using `Cacher` in the
`generate_workout` function to abstract away the caching logic</span>

En lugar de guardar el *closure* en una variable directamente, guardamos una
nueva instancia de `Cacher` que contiene el *closure*. Luego, en cada lugar
que queremos el resultado, llamamos al método `value` en la instancia
`Cacher`. Podemos llamar al método `value` tantas veces como queramos, o no
llamarlo en absoluto, y el costoso cálculo se ejecutará como máximo una vez.

Intente ejecutar este programa con la función `main` del Listado 13-2.
Cambie los valores en las variables `simulated_user_specified_value` y
`simulated_random_number` para verificar que en todos los casos en los diversos bloques `if` y `else`, `calculating slowly...` aparece solo una
vez y solo cuando es necesario. El `Cacher` se encarga de la lógica
necesaria para garantizar que no estamos llamando el cálculo caro más de lo
que necesitamos para que `generate_workout` pueda enfocarse en la lógica del
negocio.

### Limitaciones de la implementación `Cacher`

Caching values is a generalmente son un comportamiento útil que podríamos
utilizar en otras partes de nuestro código con *closures* diferentes. Sin
embargo, hay dos problemas con la implementación actual de `Cacher` que
dificultaría su reutilización en diferentes contextos.

El primer problema es que una instancia `Cacher` supone que siempre obtendrá
el mismo valor para el parámetro `arg` que el método `value`. Es decir, esta
prueba de `Cacher` fallará:

```rust,ignore
#[test]
fn call_with_different_values() {
    let mut c = Cacher::new(|a| a);

    let v1 = c.value(1);
    let v2 = c.value(2);

    assert_eq!(v2, 2);
}
```

Esta prueba crea una nueva instancia `Cacher` con un *closure* que devuelve
el valor pasado en él. Llamamos al método `value` en esta instancia `Cacher`
con un valor `arg` de 1 y luego un valor `arg` de 2, y esperamos que la
llamada a `value` con el valor `arg` de 2 para regresar 2.

Ejecute esta prueba con la implementación `Cacher` en el listado 13-9 y el
listado 13-10, y la prueba fallará en `assert_eq!` con este mensaje:

```text
thread 'call_with_different_values' panicked at 'assertion failed: `(left == right)`
  left: `1`,
 right: `2`', src/main.rs
```

El problema es que la primera vez que llamamos a `c.value` con 1, la
instancia `Cacher` guarda `Some(1)` en `self.value`. A partir de entonces,
no importa lo que pasemos al método de `value`, siempre devolverá 1.

Intenta modificar `Cacher` para mantener un mapa hash en lugar de un solo
valor. Las claves del mapa hash serán los valores `arg` que se pasan, y los
valores del mapa hash serán el resultado de llamar al *closure* de esa
clave. En lugar de ver si `self.value` directamente tiene un valor `Some` o
`None`, la función `value` buscará el `arg` en el mapa hash y devolverá el
valor si está presente. Si no está presente, el `Cacher` llamará al
*closure* y guardará el valor resultante en el mapa hash asociado con su
valor `arg`.

El segundo problema con la implementación actual de `Cacher` es que solo
acepta *closure* que toman un parámetro de tipo `u32` y devuelven un `u32`.
Es posible que deseemos cache los resultados de los *closure* que toman un
*string slice* y devolver valores `usize`, por ejemplo. Para solucionar este
problema, intente introducir más parámetros genéricos para aumentar la
flexibilidad de la funcionalidad `Cacher`.

### Capturando el entorno con *Closures*

En el ejemplo del generador de ejercicios, solo utilizamos *closures* como
funciones anónimas en línea. Sin embargo, los *closures* tienen una
capacidad adicional que las funciones no tienen: pueden capturar su entorno
y acceder a las variables desde el ámbito en el que están definidas.

El listado 13-12 tiene un ejemplo de un *closure* almacenado en la variable
`equal_to_x` que usa la variable `x` del entorno circundante del *closure*.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = 4;

    let equal_to_x = |z| z == x;

    let y = 4;

    assert!(equal_to_x(y));
}
```

<span class="caption">Listado 13-12: Ejemplo de *closure* que se refiere a
una variable en su ámbito de aplicación</span>

Aquí, aunque `x` no es uno de los parámetros de `equal_to_x`, el *closure*
`equal_to_x` puede usar la variable `x` que está definida en el mismo ámbito
en el que se define `equal_to_x`.

No podemos hacer lo mismo con las funciones; si intentamos con el siguiente
ejemplo, nuestro código no se compilará:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let x = 4;

    fn equal_to_x(z: i32) -> bool { z == x }

    let y = 4;

    assert!(equal_to_x(y));
}
```

Recibimos un error:

```text
error[E0434]: can't capture dynamic environment in a fn item; use the || { ...
} closure form instead
 --> src/main.rs
  |
4 |     fn equal_to_x(z: i32) -> bool { z == x }
  |                                          ^
```

¡El compilador incluso nos recuerda que esto solo funciona con *closures*!

Cuando un *closure* captura un valor de su entorno, usa memoria para
almacenar los valores para usar en el cuerpo del *closure*. Este uso de la
memoria es una sobrecarga que no deseamos pagar en los casos más comunes en
los que queremos ejecutar código que no captura su entorno. Debido a que
nunca se permite que las funciones capturen su entorno, la definición y el
uso de funciones nunca incurrirán en esta sobrecarga.

Los *closures* pueden capturar los valores de su entorno de tres maneras,
que se relacionan directamente con las tres formas en que una función puede
tomar un parámetro: tomar posesión, pedir prestado de forma mutable y pedir
prestado de manera inmutable. Estos están codificados en los tres *trait*
`Fn` de la siguiente manera:

* `FnOnce` consume las variables que captura de su ámbito adjunto, conocido
 como *environment* del *closure*. Para consumir las variables capturadas,
 el *closure* debe tomar posesión de estas variables y moverlas al *closure*
 cuando se define. La parte `Once` del nombre representa el hecho de que el
 *closure* no puede tomar posesión de las mismas variables más de una vez,
 por lo que solo se puede llamar una vez.
* `FnMut` puede cambiar el entorno porque toma prestados valores de forma
 mutable.
* `Fn` toma prestados valores del entorno inmutables.

Cuando crea un *closure*, Rust infiere qué *trait* usar en función de cómo
utiliza el *closure* los valores del entorno. Todos los *closures*
implementan `FnOnce` porque todos se pueden llamar al menos una vez. Los
*closures* que no mueven las variables capturadas también implementan
`FnMut`, y los *closures* que no necesitan acceso mutable a las variables
capturadas también implementan `Fn`. En el listado 13-12, el *closure*
`equal_to_x` toma `x` inmutablemente (entonces `equal_to_x` tiene el *trait*
`Fn`) porque el cuerpo del *closure* solo necesita leer el valor en `x`.

Si desea obligar al *closure* a apropiarse de los valores que utiliza en el
entorno, puede utilizar la palabra clave `move` antes de la lista de
parámetros. Esta técnica es principalmente útil al pasar un *closure* a un
nuevo hilo para mover los datos de modo que sea propiedad del nuevo hilo.

Tendremos más ejemplos de *closures* `move` en el Capítulo 16 cuando hablamos de concurrencia. Por ahora, aquí está el código del listado 13-12 con la palabra clave `move` agregada a la definición de clausura y usando
vectores en lugar de enteros, porque los enteros pueden copiarse en lugar de
moverse; tenga en cuenta que este código aún no se compilará.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let x = vec![1, 2, 3];

    let equal_to_x = move |z| z == x;

    println!("can't use x here: {:?}", x);

    let y = vec![1, 2, 3];

    assert!(equal_to_x(y));
}
```

Recibimos el siguiente error:

```text
error[E0382]: use of moved value: `x`
 --> src/main.rs:6:40
  |
4 |     let equal_to_x = move |z| z == x;
  |                      -------- value moved (into closure) here
5 |
6 |     println!("can't use x here: {:?}", x);
  |                                        ^ value used here after move
  |
  = note: move occurs because `x` has type `std::vec::Vec<i32>`, which does not
  implement the `Copy` trait
```

El valor 'x' se mueve hacia el *closure* cuando se define el *closure*,
porque añadimos la palabra clave `mover`. El *closure* tiene la propiedad de
`x`, y `main` no puede usar `x` en la instrucción `println!`. Al eliminar
`println!` Se solucionará este ejemplo.

La mayoría de las veces al especificar uno de los *trait bounds* `Fn`, puede
comenzar con `Fn` y el compilador le dirá si necesita `FnMut` o `FnOnce`
según lo que ocurra en el cuerpo del *closure*.

Para ilustrar situaciones en las que los *closures* pueden capturar su
entorno son útiles como parámetros de función, pasemos a nuestro siguiente
tema: iteradores.