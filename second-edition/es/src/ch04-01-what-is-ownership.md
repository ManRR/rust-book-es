## ¿Qué es la propiedad (Ownership)?

La característica central de Rust es *ownership* (*propiedad*). Aunque la característica es directa
para explicar, tiene profundas implicaciones para el resto del lenguaje.

Todos los programas tienen que administrar la forma en que usan la memoria de una
computadora mientras se ejecuta. Algunos lenguajes tienen *recolección de basura*,
*recolector de basura*, que constantemente busca memoria que ya no se usa cuando el
programa se ejecuta; en otros lenguajes, el programador debe explícitamente asignar
y libera la memoria. Rust utiliza un tercer enfoque: la memoria se gestiona a través
de un sistema de propiedad con un conjunto de reglas que el compilador verifica en
tiempo de compilación. Ninguna de las características de propiedad ralentiza su programa
mientras está en funcionamiento.

Como la propiedad es un concepto nuevo para muchos programadores, lleva tiempo
acostumbrarse a ella. La buena noticia es que cuanto más experimentado te vuelves con Rust
y las reglas del sistema de propiedad (*ownership*), más podrá, naturalmente,
desarrollar código que sea seguro y eficiente. ¡Síguelo!

Cuando comprenda la propiedad, tendrá una base sólida para la comprensión
las características que hacen que Rust sea único. En este capítulo, aprenderá la propiedad
trabajando a través de algunos ejemplos que se centran en una estructura de datos muy común:
*strings*.

> ### La pila (*Stack*) y el montículo (*Heap*).

> En muchos lenguajes de programación, no tiene que pensar en el *Stack* y
> el *Heap* muy a menudo. Pero en un lenguaje de programación de sistemas como Rust, ya sea
> que un valor está en el *Stack* o el *Heap* tiene más de un efecto sobre cómo el lenguaje
> se comporta y por qué tiene que tomar ciertas decisiones. Partes de la propiedad
> se describirá en relación con el *Stack* y el *Heap* más adelante en este capítulo, por lo que
> aquí hay una breve explicación en preparada.
>
> Tanto la pila como el montículo son partes de la memoria que está disponible para usar en su código
> en tiempo de ejecución, pero están estructurados de diferentes maneras. El *stack* almacena los
> valores en el orden en que los obtiene y elimina los valores en el orden opuesto.
> Esto se conoce como *last in, first out*, *LIFO*, (último en entrar, primero en salir). Piensa en una pila de platos: cuando
> agregas más platos, los pones encima de la pila, y cuando necesitas un
> plato, tomas uno de la parte superior. Agregar o quitar platos del medio o
> ¡abajo no funcionaría tan bien! Agregar datos se llama *pushing onto the stack* (empujando hacia la pila),
> y eliminar datos se llama *popping off the stack* (saliendo de la pila).
>
> La pila es rápida debido a la forma en que accede a los datos: nunca tiene que
> buscar un lugar para poner nuevos datos o un lugar para obtener datos porque eso
> el lugar siempre es la parte superior. Otra propiedad que hace que la pila sea rápida es que
> todos los datos en la pila deben tomar un tamaño conocido y fijo.
>
> Los datos con un tamaño desconocido en el momento de la compilación o un tamaño que podría cambiar pueden ser
> almacenado en el *Heap* en su lugar. El *Heap* está menos organizado: cuando pones datos en
> el *Heap*, pides una cierta cantidad de espacio. El sistema operativo encuentra
> un lugar vacío en algún lugar del *Heap* que sea lo suficientemente grande, lo marca como que esta en uso,
> y devuelve un *puntero*, que es la dirección de esa ubicación. Este
> proceso se llama *allocating on the heap* (*asignando en el montículo*), a veces abreviado como solo
> “allocating”. No se considera asignar valores al *stack*.
> Debido a que el puntero es de un tamaño conocido y fijo, puede almacenar el puntero en el *stack*, pero cuando quiere los datos reales, debe seguir el puntero.
>
> Piense en sentarse en un restaurante. Cuando ingresas, indicas la cantidad de
> personas de tu grupo, y el personal encuentra una mesa vacía que se adapta a todos
> y te lleva allí. Si alguien en su grupo llega tarde, pueden preguntar dónde
> has estado sentado para encontrarte.
>
> Acceder a los datos en el *Heap* es más lento que acceder a los datos en el *stack* porque
> tienes que seguir un puntero para llegar allí. Los procesadores contemporáneos son más rápidos
> si saltan menos en la memoria. Continuando con la analogía, considera un camarero
> en un restaurante que recibe pedidos de muchas mesas. Es más eficiente obtener
> todos los pedidos en una mesa antes de pasar a la siguiente mesa. Tomando una orden de la mesa A, luego una
> orden de la mesa B, luego una de la mesa A otra vez, y
> entonces uno más de la mesa B sería un proceso mucho más lento. Por la misma razón, un procesador puede
> hacer su trabajo mejor si funciona con datos que están cerca de otros datos (como en el *stack*) en lugar de
> estar más lejos (como puede ser en el *heap*). Asignar una gran cantidad de espacio en el *heap* también
> puede llevar tiempo.
>
> Cuando su código llama a una función, los valores pasan a la función
> (incluidos, potencialmente, punteros a los datos en el montículo) y la función de
> las variables locales se insertan en la pila. Cuando la función termina, aquellos
> los valores se eliminan de la pila.
>
> Hacer un seguimiento de qué partes del código están usando qué datos en el montículo,
> minimizar la cantidad de datos duplicados en el montículo y limpiar sin usar
> los datos en el montículo para que no te quedes sin espacio son todos los problemas que aborda la
> *propiedad dirección* (*ownership addresses*). Una vez que comprenda la propiedad, no tendrá que pensar en la
> pila y el montículo con frecuencia, pero sabiendo que la gestión de los datos del montículo es por qué
> la propiedad existe puede ayudar a explicar por qué funciona de la manera en que funciona.

### Reglas de propiedad (*Ownership*)

Primero, echemos un vistazo a las reglas de propiedad. Tenga en cuenta estas reglas cuando trabaje a través de los ejemplos que los ilustran:

* Cada valor en Rust tiene una variable que se llama *owner*.
* Solo puede haber un propietario a la vez.
* Cuando el propietario sale del alcance, el valor se eliminará.

### Alcance variable (*Scope*)

Ya revisamos un ejemplo de un programa de Rust en el Capítulo 2. Ahora
que hemos pasado la sintaxis básica, no incluiremos todo el código `fn main () {` en ejemplos, así que si estás siguiendo, tendrás que poner los siguientes ejemplos dentro de una función `main` de forma manual. Como resultado, nuestros ejemplos serán una un poco más conciso, dejándonos enfocarnos en los detalles reales en lugar de código repetitivo.

Como primer ejemplo de propiedad, veremos el *scope* de algunas variables. Un *scope* es el rango dentro de un programa para el cual un artículo es válido. Digamos que tenemos una variable que se ve así:

```rust
let s = "hello";
```

La variable `s` se refiere a una *string* literal, donde el valor del *string* es
codificado en el texto de nuestro programa. La variable es válida desde el punto en el
que se declara hasta el final del *scope* actual. El listado 4-1 tiene comentarios
anotando donde la variable `s` es válida.

```rust
{                      // s no es válido aquí, aún no está declarada
    let s = "hello";   // s es válido a partir de este punto

    // hacer cosas con s
}                      // este alcance ha terminado, y s ya no es válido
```

<span class="caption">Listing 4-1: Una variable y el alcance en el que es válida</span>

En otras palabras, hay dos puntos importantes en el tiempo aquí:

* Cuando `s` entra en el *scope*, es válido.
* Sigue siendo válido hasta que se sale *scope*.

En este punto, la relación entre los ámbitos y cuándo las variables son válidas es similar a la de otros lenguajes de programación. Ahora construiremos sobre esta comprensión introduciendo el tipo `String`.

### El tipo `String`

Para ilustrar las reglas de propiedad (*ownership*), necesitamos un tipo de datos que sea más complejo que los que cubrimos en la sección "Tipos de datos" del Capítulo 3. Los tipos cubiertos anteriormente están todos almacenados en la pila (*stack*) y salieron de la pila (*stack*) cuando
su alcance ha terminado, pero queremos ver los datos que se almacenan en el montículo (*heap*) y explora cómo Rust sabe cuándo limpiar esos datos.

Usaremos `String` como ejemplo aquí y nos concentraremos en las partes de `String` que se relacionan con la propiedad. Estos aspectos también se aplican a otros tipos de datos complejos provisto por la biblioteca estándar (*std*) y que usted crea. Estudiaremos el tipo `String` con más profundidad en el Capítulo 8.

Ya vimos *string* literales, donde un valor de *string* está codificado en nuestro programa. Los literales de *string* son convenientes, pero no son adecuados para toda situación en la cual podemos querer usar texto. Una razón es que son inmutable. Otra es que no todos los valores de *string* se pueden conocer cuando escribimos nuestro código: por ejemplo, ¿qué sucede si queremos tomar la entrada del usuario y almacenarla? por
estas situaciones, Rust tiene un segundo tipo de *string*, `String`. Este tipo es asignado en el montículo (*heap*) y como tal es capaz de almacenar una cantidad de texto que es desconocido para nosotros en tiempo de compilación. Puede crear un `String` a partir de un literal de *string*
usando la función `from`, así:

```rust
let s = String::from("hello");
```

Los dos puntos dobles (`::`) es un operador que nos permite el espacio de nombres
(*namespace*) esta función particular `from` bajo el tipo `String` en lugar de utilizar
algún tipo de nombre como `string_from`. Estudiaremos esta sintaxis más en la sección
“Método Sintaxis” del Capítulo 5 y cuando hablamos de espacios de nombres con módulos en
"Definiciones de módulos" en el Capítulo 7.

Este tipo de *string*, *pueden* mutar:

```rust
let mut s = String::from("hello");

s.push_str(", world!"); // push_str() agrega un literal a un String

println!("{}", s);      // Esto imprimirá `hello, world!`
```

Entonces, ¿cuál es la diferencia aquí? ¿Por qué `String` puede mutarse pero los literales no?
La diferencia es cómo estos dos tipos lidian con la memoria

### Memoria y Asignación

En el caso de un *string* literal, conocemos el contenido en tiempo de compilación, por lo que el texto está codificado directamente en el ejecutable final. Esta es la razón por la que los *string* literales son rápidos y eficientes. Pero estas propiedades solo provienen de la inmutabilidad del literal del *string*. Desafortunadamente, no podemos poner una burbuja de memoria en el binario por cada fragmento de texto cuyo tamaño se desconoce en el momento de la compilación y cuyo tamaño podría cambiar mientras se ejecuta el programa.

Con el tipo `String`, para admitir un fragmento de texto mutable y creíble,
necesitamos asignar una cantidad de memoria en el montículo, desconocida en tiempo de compilación, para contener los contenidos. Esto significa:

* La memoria debe solicitarse desde el sistema operativo en tiempo de ejecución.
* Necesitamos una forma de devolver esta memoria al sistema operativo cuando hayamos terminado con nuestro `String`.

Esa primera parte la hacemos nosotros: cuando llamamos a `String::from`, su implementación solicita la memoria que necesita. Esto es bastante universal en los lenguajes de programación.

Sin embargo, la segunda parte es diferente. En lenguajes con un *recolector de basura (GC)*, el GC realiza un seguimiento y limpia la memoria que ya no se usa, y no necesitamos pensar en ello. Sin un GC, es nuestra responsabilidad identificar cuándo ya no se usa la memoria y llamar al código para devolverla explícitamente, tal como lo hicimos para solicitarlo. Hacer esto correctamente ha sido históricamente un problema de programación difícil. Si lo olvidamos, perderemos la memoria. Si lo hacemos demasiado pronto, tendremos una variable inválida. Si lo hacemos dos veces, también es un error.
Necesitamos emparejar exactamente un `allocate` con exactamente un `free`.

Rust toma una ruta diferente: la memoria se devuelve automáticamente una vez que la variable que lo posee queda fuera del alcance. Aquí hay una versión de nuestro ejemplo de alcance del Listado 4-1 usando un `String` en lugar de un literal de *string*:

```rust
{
    let s = String::from("hello"); // s es válido a partir de este punto

    // hacer cosas con s
}                                  // este alcance ha terminado, y s ya no es válido
```

Aquí hay un punto natural en el que podemos devolver la memoria que nuestro `String` necesita para el sistema operativo: cuando `s` sale del alcance. Cuando una variable se sale del alcance, Rust llama a una función especial para nosotros. Esta función se llama `drop`, y es donde el autor de `String` puede poner el código para devolver la memoria. Rust llama `drop` automáticamente al cierre de la llave.

> Nota: En C ++, este patrón de desasignación de recursos al final de la vida útil de un elemento se denomina a veces *Resource Acquisition Is Initialization (RAII)* (*Inicialización de adquisición de recursos (RAII)*).
> La función `drop` en Rust le resultará familiar si ha utilizado patrones RAII.

Este patrón tiene un profundo impacto en la forma en que se escribe el código de Rust. Puede parecer simple en este momento, pero el comportamiento del código puede ser
inesperado en situaciones más complicadas cuando queremos que múltiples variables usen
los datos que hemos asignado en el montículo (*heap*). Exploremos algunas de esas
situaciones ahora.

#### Ways Variables and Data Interact: Move

Múltiples variables pueden interactuar con los mismos datos de diferentes maneras en Rust.
Veamos un ejemplo usando un número entero en el Listado 4-2.

```rust
let x = 5;
let y = x;
```

<span class="caption">Listing 4-2: Asignando el valor entero de la variable `x` a `y`</span>

Probablemente podamos adivinar qué está haciendo esto: “vincula el valor `5` a `x`; luego haga una copia del valor en `x` y agréguela a `y`.” Ahora tenemos dos variables,`x` e `y`, y ambas son `5`. De hecho, esto es lo que está sucediendo, porque los enteros son valores simples con un tamaño conocido y fijo, y estos dos valores `5` se insertan en la pila (*stack*).

Ahora veamos la versión `String`:

```rust
let s1 = String::from("hello");
let s2 = s1;
```

Esto se ve muy similar al código anterior, por lo que podemos suponer que la forma en que funciona sería la misma: es decir, la segunda línea haría una copia del valor en `s1` y lo vincularía a `s2`. Pero esto no es exactamente lo que sucede.

Eche un vistazo a la Figura 4-1 para ver lo que está sucediendo con `String` debajo de las sábanas. Una `String` se compone de tres partes, que se muestran a la izquierda: un puntero a la memoria que contiene el contenido del *string*, una longitud y una capacidad. Este grupo de datos se almacena en la pila (*stack*). A la derecha está la memoria en el montículo *(heap*) que contiene los contenidos.

<img alt="String in memory" src="img/trpl04-01.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-1: Representación en memoria de un `String`
sosteniendo el valor `"hello"` bound to `s1`</span>

La longitud es la cantidad de memoria, en bytes, que está usando el contenido de `String`.
La capacidad es la cantidad total de memoria, en bytes, que el `String` recibió del sistema
operativo. La diferencia entre la duración y la capacidad importa, pero no en este contexto,
por lo que, por ahora, está bien ignorar la capacidad.

Cuando asignamos `s1` a `s2`, los datos de `String` se copian, lo que significa que copiamos
el puntero, la longitud y la capacidad que están en la pila (*stack*). No copiamos los datos
en el montículo (*heap*) al que se refiere el puntero. En otras palabras, la representación de
datos en la memoria se parece a la Figura 4-2.

<img alt="s1 and s2 pointing to the same value" src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-2: Representación en memoria de la variable `s2`
que tiene una copia del puntero, longitud y capacidad de `s1`</span>

La representación * no * se parece a la Figura 4-3, que es como se vería la memoria
si Rust copiara también los datos del montículo. Si Rust hiciera esto, la operación `s2 = s1`
podría ser muy costosa en términos de rendimiento en tiempo de ejecución si los datos
en el montículo fueran grandes.

<img alt="s1 and s2 to two places" src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-3: Otra posibilidad para qué `s2 = s1`
podría hacer si Rust copiara los datos del montículo (*heap*) también</span>

Anteriormente, dijimos que cuando una variable queda fuera del alcance, Rust
llama automáticamente a la función `drop` y limpia la memoria del montículo para esa variable.
Pero la Figura 4-2 muestra ambos punteros de datos apuntando a la misma ubicación.
Esto es un problema: cuando `s2` y `s1` salen del alcance, ambos intentarán liberar la misma memoria.
Esto se conoce como un *double free* error y es uno de los errores de seguridad de memoria
que mencionamos anteriormente. Liberar la memoria dos veces puede provocar daños en la memoria,
lo que puede generar vulnerabilidades de seguridad.

Para garantizar la seguridad de la memoria, hay un detalle más de lo que sucede en esta
situación en Rust. En lugar de intentar copiar la memoria asignada, Rust considera que
`s1` ya no es válido y, por lo tanto, Rust no necesita liberar nada cuando `s1` sale del alcance.
Echa un vistazo a lo que sucede cuando tratas de usar `s1` después de crear `s2`; no funcionará:

```rust,ignore
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1);
```

Recibirá un error como este porque Rust le impide usar la referencia invalidada:

```text
error[E0382]: use of moved value: `s1`
 --> src/main.rs:5:28
  |
3 |     let s2 = s1;
  |         -- value moved here
4 |
5 |     println!("{}, world!", s1);
  |                            ^^ value used here after move
  |
  = note: move occurs because `s1` has type `std::string::String`, which does
  not implement the `Copy` trait
```

Si ha escuchado los términos *shallow copy* (*copia superficial*) y *deep copy* (*copia profunda*)
mientras trabaja con otros lenguajes, el concepto de copiar el puntero, la longitud y la capacidad
sin copiar los datos probablemente suene como una copia superficial. Pero debido a que Rust
también invalida la primera variable, en lugar de llamarse una copia superficial, se la
conoce como *mover*. En este ejemplo, diríamos que `s1` se *movió* a `s2`. Entonces,
lo que realmente sucede se muestra en la Figura 4-4.

<img alt="s1 moved to s2" src="img/trpl04-04.svg" class="center" style="width: 50%;" />

<span class="caption">Figure 4-4: La representación en memoria después de `s1` ha sido invalidada</span>

¡Eso resuelve nuestro problema! Con solo `s2` válido, cuando salga del alcance, solo
liberará la memoria y terminaremos.

Además, hay una opción de diseño implícita: Rust nunca creará automáticamente copias “profundas”
de sus datos. Por lo tanto, se puede suponer que cualquier copiado *automático* es económico en
términos de rendimiento en tiempo de ejecución.

#### Ways Variables and Data Interact: Clone

Si *hacemos* queremos copiar profundamente los datos del montículo (*heap*) de `String`,
no solo los datos de la pila (*stack*), podemos usar un método común llamado `clone`.
Estudiaremos la sintaxis del método en el Capítulo 5, pero dado que los métodos son
una característica común en muchos lenguajes de programación, probablemente ya los haya visto antes.

Aquí hay un ejemplo del método `clone` en acción:

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

Esto funciona bien y produce explícitamente el comportamiento que se muestra en
la Figura 4-3, donde los datos del montículo *no* se copian.

Cuando ves una llamada a `clone`, sabes que se está ejecutando algún código
arbitrario y que el código puede ser costoso. Es un indicador visual de que
algo diferente está sucediendo.

#### Stack-Only Data: Copy

Hay otra arruga de la que aún no hemos hablado. Este código que usa números enteros,
parte del cual se mostró en el Listado 4-2, funciona y es válido:

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

Pero este código parece contradecir lo que acabamos de aprender: no tenemos una llamada a
`clon`, pero` x` sigue siendo válida y no se movió a `y`.

La razón es que los tipos como los enteros que tienen un tamaño conocido en el momento
de la compilación se almacenan completamente en la pila, por lo que las copias de los
valores reales son rápidos de realizar. Eso significa que no hay ninguna razón por la que
quisiéramos evitar que `x` sea válida después de que creamos la variable` y`. En otras
palabras, aquí no hay diferencia entre la copia profunda y la poco profunda, por lo que
llamar `clon` no haría nada diferente de la copia superficial normal y podemos omitirlo.

Rust tiene una anotación especial llamada el *trait* `Copy` que podemos colocar en
tipos como enteros que están almacenados en la pila (hablaremos más sobre los rasgos en el Capítulo 10).
Si un tipo tiene el *trait* `Copy`, una variable más vieja todavía se puede usar después de la asignación.
Rust no nos permitirá anotar un tipo con el *trait* `Copy` si el tipo, o cualquiera de sus partes,
ha implementado el *trait* `Drop`. Si el tipo necesita que ocurra algo especial cuando el
valor se sale del alcance y agregamos la anotación `Copy` a ese tipo, obtendremos un error
en tiempo de compilación. Para obtener información sobre cómo agregar la anotación `Copy`
a su tipo, consulte “Rasgos derivables” en el Apéndice C.

Entonces, ¿qué tipos son `Copy`? Puede verificar la documentación del tipo dado para asegurarse,
pero como regla general, cualquier grupo de valores escalares simples puede ser `Copy`, y
nada que requiera asignación o es alguna forma de recurso es `Copy`. Estos son algunos de los
tipos que son `Copiar`:

* Todos los tipos de enteros, como `u32`.
* El tipo booleano, `bool`, con los valores `true` y `false`.
* Todos los tipos de punto flotante, como `f64`.
* El tipo de carácter, `char`.
* Tuplas, pero solo si contienen tipos que también son `Copy`. Por ejemplo,
   `(i32, i32)` que son `Copy`, pero `(i32, String)` no lo son.

### Propiedad y funciones

La semántica para pasar un valor a una función es similar a la de establecer
un valor para una variable. Pasar una variable a una función se moverá o copiará,
tal como lo hace la asignación. El listado 4-3 tiene un ejemplo con algunas anotaciones
que muestran dónde las variables entran y salen del alcance (*scope*).

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let s = String::from("hello");  // s entra en el alcance

    takes_ownership(s);             // s's el valor se mueve a la función ...
                                    // ... y entonces ya no es válido aquí

    let x = 5;                      // x entra en el alcance

    makes_copy(x);                  // x se movería a la función,
                                    // pero i32 es Copy, así que está bien todavía
                                    // use x después

} // Aquí, x sale del alcance, luego s. Pero como el valor de s se movió, nada
  // sucede de manera especial.

fn takes_ownership(some_string: String) { // some_string entra en el alcance
    println!("{}", some_string);
} // Aquí, some_string sale del alcance y se llama `drop`. La memoria de
  // respaldo se libera.

fn makes_copy(some_integer: i32) { // some_integer entra en el alcance
    println!("{}", some_integer);
} // Aquí, some_integer sale del alcance. Nada especial sucede.
```

<span class="caption">Listing 4-3: Funciones con propiedad y alcance anotado</span>

Si intentamos usar `s` después de la llamada a` takes_ownership`, Rust arrojaría
un error en tiempo de compilación. Estas comprobaciones estáticas nos protegen de
los errores. Intenta agregar código a `main` que use` s` y `x` para ver dónde puedes
usarlos y dónde las reglas de propiedad te impiden hacerlo.

### Valores de retorno y alcance

La devolución de valores también puede transferir la propiedad (*ownership*).
El listado 4-4 es un ejemplo con anotaciones similares a las del Listado 4-3.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership mueve su valor de 
                                        // retorno a s1

    let s2 = String::from("hello");     // s2 entra en el alcance

    let s3 = takes_and_gives_back(s2);  // s2 se mueve a takes_and_gives_back, 
                                        // que también mueve su valor 
                                        // de retorno a s3
} // Aquí, s3 sale del alcance y se descarta. s2 sale del alcance pero se movió,
  // por lo que no sucede nada. s1 sale del alcance y se descarta.

fn gives_ownership() -> String {             // gives_ownership moverá su 
                                             // valor de retorno a la función 
                                             // que lo llama.

    let some_string = String::from("hello"); // some_string entra en el alcance

    some_string                              // some_string se devuelve y 
                                             // se mueve a la función de llamada.
}

// takes_and_gives_back tomará un *string* y devolverá uno
fn takes_and_gives_back(a_string: String) -> String { // a_string entra en el alcance

    a_string  // a_string se devuelve y se mueve a la función de llamada
}
```

<span class="caption">Listing 4-4: Transferir la propiedad (*ownership*) de los valores devueltos</span>

La propiedad (*ownership*) de una variable sigue el mismo patrón cada vez: la asignación de un valor
a otra variable lo mueve. Cuando una variable que incluye datos en el montículo queda
fuera del alcance, el valor se limpiará mediante `drop` a menos que los datos hayan
sido movidos para ser propiedad *owned* de otra variable.

Tomar posesión y luego devolver la propiedad (*ownership*) con cada función es un poco tedioso.
¿Qué sucede si queremos permitir que una función use un valor pero no tome posesión?
Es bastante molesto que cualquier cosa que pasemos también deba transmitirse si queremos
volver a usarlo, además de cualquier dato que provenga del cuerpo de la función que también
deseemos devolver.

Es posible devolver múltiples valores usando una tupla, como se muestra en el Listado 4-5.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String

    (s, length)
}
```

<span class="caption">Listing 4-5: Devolución de la propiedad (*ownership*) de los parámetros</span>

Pero esto es demasiada ceremonia y mucho trabajo para un concepto que debería ser común.
Afortunadamente para nosotros, Rust tiene una característica para este concepto, llamada
*referencias* (*references*).
