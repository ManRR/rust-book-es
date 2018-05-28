## Procesamiento de una serie de elementos con iteradores

El patrón de iterador le permite realizar una tarea en una secuencia de
elementos a su vez. Un iterador es responsable de la lógica de iterar sobre
cada elemento y determinar cuándo terminó la secuencia. Cuando usa iteradores
no tiene que volver a implementar esa lógica usted mismo.

En Rust, los iteradores son *lazy*, lo que significa que no tienen ningún
efecto hasta que llamas a los métodos que consumen el iterador para usarlo.
Por ejemplo, el código en el listado 13-13 crea un iterador sobre los
elementos en el vector `v1` llamando al método `iter` definido en `Vec <T>`.
Este código en sí mismo no hace nada útil.

```rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();
```

<span class="caption">Listado 13-13: Creando un iterador</span>

Una vez que hemos creado un iterador, podemos usarlo de varias maneras. En el
Listado 3-5 del Capítulo 3, usamos iteradores con bucles `for` para ejecutar
código en cada elemento, aunque pasamos por alto lo que hacía la llamada a
`iter` hasta ahora.

El ejemplo del listado 13-14 separa la creación del iterador del uso del
iterador en el bucle `for`. El iterador se almacena en la variable `v1_iter`,
y no tiene lugar ninguna iteración en ese momento. Cuando se llama al bucle
`for` utilizando el iterador en `v1_iter`, cada elemento en el iterador se
usa en una iteración del bucle, que imprime cada valor.

```rust
let v1 = vec![1, 2, 3];

let v1_iter = v1.iter();

for val in v1_iter {
    println!("Got: {}", val);
}
```

<span class="caption">Listado 13-14: Uso de un iterador en un bucle
`for`</span>

En lenguajes que no tienen iteradores proporcionados por sus bibliotecas
estándar, es probable que escriba esta misma funcionalidad iniciando una
variable en el índice 0, usando esa variable para indexar en el vector para
obtener un valor e incrementando el valor de la variable en un bucle hasta
que alcanzó la cantidad total de elementos en el vector.

Los iteradores manejan toda esa lógica por usted, reduciendo el código
repetitivo que potencialmente podría arruinar. Los iteradores te dan más
flexibilidad para usar la misma lógica con muchos tipos diferentes de
secuencias, no solo estructuras de datos en las que puedes indexar, como
vectores. Examinemos cómo los iteradores hacen eso.

### El trait `Iterator` y el método `next`

Todos los iteradores implementan un *trait* llamado `Iterator` que se define en la biblioteca estándar. La definición del *trait* se ve así:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // methods with default implementations elided
}
```

Observe que esta definición usa alguna sintaxis nueva: `type Item` y
`Self::Item`, que definen un *tipo asociado* con este *trait*. Hablaremos
sobre los tipos asociados en profundidad en el Capítulo 19. Por ahora, todo
lo que necesita saber es que este código dice que implementar el *trait*
`Iterator` requiere que también defina un tipo `Item`, y este tipo `Item`
es utilizado en el tipo de devolución del método `next`. En otras palabras,
el tipo `Item` será el tipo devuelto por el iterador.

El *trait* `Iterator` solo requiere que los implementadores definan un
método: el método `next`, que devuelve un elemento del iterador a la vez
envuelto en `Some` y, cuando la iteración termina, devuelve `None`.

Podemos llamar directamente al método `next` en los iteradores; El listado
13-15 demuestra qué valores se devuelven de las llamadas repetidas a `next`
en el iterador creado a partir del vector.

<span class="filename">Filename: src/lib.rs</span>

```rust
#[test]
fn iterator_demonstration() {
    let v1 = vec![1, 2, 3];

    let mut v1_iter = v1.iter();

    assert_eq!(v1_iter.next(), Some(&1));
    assert_eq!(v1_iter.next(), Some(&2));
    assert_eq!(v1_iter.next(), Some(&3));
    assert_eq!(v1_iter.next(), None);
}
```

<span class="caption">Listado 13-15: Llamar al método `next` en un
iterador</span>

Tenga en cuenta que necesitamos hacer `v1_iter` mutable: al llamar al método
`next` en un iterador cambia el estado interno que el iterador usa para hacer
un seguimiento de dónde está en la secuencia. En otras palabras, este código
*consume*, o agota, el iterador. Cada llamada a `next` come un elemento del
iterador. No necesitábamos hacer `v1_iter` mutable cuando usamos un bucle
`for` porque el bucle tomó posesión de `v1_iter` y lo hizo mutable detrás de
las escenas.

También tenga en cuenta que los valores que obtenemos de las llamadas a
`next` son referencias inmutables a los valores en el vector. El método
`iter` produce un iterador sobre referencias inmutables. Si queremos crear un
iterador que tome posesión de `v1` y devuelva valores propios, podemos llamar
a `into_iter` en lugar de `iter`. De forma similar, si queremos iterar sobre
referencias mutables, podemos llamar `iter_mut` en lugar de `iter`.

### Métodos que consumen el iterador

El *trait* `Iterator` tiene varios métodos diferentes con implementaciones
predeterminadas proporcionadas por la biblioteca estándar; Puede averiguar
acerca de estos métodos buscando en la documentación de la API de la
biblioteca estándar el *trait* `Iterator`. Algunos de estos métodos llaman al
método `next` en su definición, por lo que debes implementar el método `next`
al implementar el *trait* `Iterator`.

Los métodos que llaman `next` se llaman *adaptadores de consumo*, porque al
invocarlos se utiliza el iterador. Un ejemplo es el método `sum`, que toma
posesión del iterador y recorre los ítems repetidamente llamando `next`,
consumiendo así el iterador. A medida que avanza, agrega cada elemento a un
total acumulado y devuelve el total cuando se completa la iteración. El
listado 13-16 tiene una prueba que ilustra el uso del método `sum`:

<span class="filename">Filename: src/lib.rs</span>

```rust
#[test]
fn iterator_sum() {
    let v1 = vec![1, 2, 3];

    let v1_iter = v1.iter();

    let total: i32 = v1_iter.sum();

    assert_eq!(total, 6);
}
```

<span class="caption">Listado 13-16: Llamar al método `sum` para obtener el
total de todos los elementos en el iterador</span>

No se nos permite usar `v1_iter` después de la llamada a `sum` porque `sum`
toma posesión del iterador sobre el que lo llamamos.

### Métodos que producen otros iteradores

Otros métodos definidos en el *trait* `Iterator`, conocidos como
*iterator adaptors*, le permiten cambiar los iteradores en diferentes tipos
de iteradores. Puede encadenar múltiples llamadas a adaptadores de iterador
para realizar acciones complejas de forma legible. Pero como todos los
iteradores son flojos, debe llamar a uno de los métodos de adaptador de
consumo para obtener resultados de las llamadas a los adaptadores de iterador.

El listado 13-17 muestra un ejemplo de invocación del método del adaptador de
iterador `map`, que toma un cierre para llamar a cada elemento y producir un
nuevo iterador. El cierre aquí crea un nuevo iterador en el que cada elemento
del vector se ha incrementado en 1. Sin embargo, este código produce una
advertencia:

<span class="filename">Filename: src/main.rs</span>

```rust
let v1: Vec<i32> = vec![1, 2, 3];

v1.iter().map(|x| x + 1);
```

<span class="caption">Listado 13-17: Llamar al adaptador iterador `map` para
crear un nuevo iterador</span>

La advertencia que recibimos es esta:

```text
warning: unused `std::iter::Map` which must be used: iterator adaptors are lazy
and do nothing unless consumed
 --> src/main.rs:4:5
  |
4 |     v1.iter().map(|x| x + 1);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = note: #[warn(unused_must_use)] on by default
```

El código en el listado 13-17 no hace nada; el *closure* que hemos
especificado nunca se llama. La advertencia nos recuerda por qué: los
adaptadores de iterador son flojos, y necesitamos consumir el iterador aquí.

Para arreglar esto y consumir el iterador, usaremos el método `collect`, que
usamos en el Capítulo 12 con `env::args` en el Listado 12-1. Este método
consume el iterador y recopila los valores resultantes en un tipo de datos de
colección.

En el listado 13-18, recogemos los resultados de la iteración sobre el
iterador que se devuelve de la llamada a `map` en un vector. Este vector
terminará conteniendo cada elemento del vector original incrementado en 1.

<span class="filename">Filename: src/main.rs</span>

```rust
let v1: Vec<i32> = vec![1, 2, 3];

let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

assert_eq!(v2, vec![2, 3, 4]);
```

<span class="caption">Listado 13-18: Llamar al método `map` para crear un
nuevo iterador y luego llamar al método `collect` para consumir el nuevo
iterador y crear un vector</span>

Debido a que `map` se *closure*, podemos especificar cualquier operación que
deseemos realizar en cada elemento. Este es un gran ejemplo de cómo los
*closures* le permiten personalizar un comportamiento mientras reutiliza el
comportamiento de iteración que proporciona el rasgo `Iterator`.

### Uso de *Closures* que capturan su entorno

Ahora que hemos introducido iteradores, podemos demostrar un uso común de
*closures* que capturan su entorno mediante el uso del adaptador de iterador
`filter`. El método `filter` en un iterador toma un *closure* que toma cada
elemento del iterador y devuelve un booleano. Si el *closure* devuelve
`true`, el valor se incluirá en el iterador producido por `filter`. Si
el *closure* devuelve `false`, el valor no se incluirá en el iterador
resultante.

En el listado 13-19, usamos `filter` con un cierre que captura la variable
`shoe_size` de su entorno para iterar sobre una colección de instancias de
estructura `Shoe`. Devolverá solo los zapatos que tengan el tamaño
especificado.

<span class="filename">Filename: src/lib.rs</span>

```rust
#[derive(PartialEq, Debug)]
struct Shoe {
    size: u32,
    style: String,
}

fn shoes_in_my_size(shoes: Vec<Shoe>, shoe_size: u32) -> Vec<Shoe> {
    shoes.into_iter()
        .filter(|s| s.size == shoe_size)
        .collect()
}

#[test]
fn filters_by_size() {
    let shoes = vec![
        Shoe { size: 10, style: String::from("sneaker") },
        Shoe { size: 13, style: String::from("sandal") },
        Shoe { size: 10, style: String::from("boot") },
    ];

    let in_my_size = shoes_in_my_size(shoes, 10);

    assert_eq!(
        in_my_size,
        vec![
            Shoe { size: 10, style: String::from("sneaker") },
            Shoe { size: 10, style: String::from("boot") },
        ]
    );
}
```

<span class="caption">Listado 13-19: Usando el método `filter` con un
*closure* que captura `shoe_size`</span>

La función `shoes_in_my_size` toma la propiedad de un vector de *shoes* y un
tamaño de *shoe* como parámetros. Devuelve un vector que contiene solo
*shoes* del tamaño especificado.

En el cuerpo de `shoes_in_my_size`, llamamos a `into_iter` para crear un
iterador que toma posesión del vector. Luego llamamos `filter` para adaptar
ese iterador a un nuevo iterador que solo contiene elementos para los cuales
el *closure* devuelve `true`.

El *closure* captura el parámetro `shoe_size` del entorno y compara el valor
con el tamaño de cada *shoe*, manteniendo solo los *shoes* del tamaño
especificado. Finalmente, al llamar `collect` se reúnen los valores devueltos
por el iterador adaptado en un vector devuelto por la función.

La prueba muestra que cuando llamamos `shoes_in_my_size`, recuperamos solo
*shoes* que tienen el mismo tamaño que el valor que especificamos.

### Creando nuestros propios iteradores con el *Trait* `Iterator`

Hemos demostrado que puede crear un iterador llamando `iter`, `into_iter`, o
`iter_mut` en un vector. Puede crear iteradores a partir de los otros tipos
de colecciones en la biblioteca estándar, como el *hash map*. También puede
crear iteradores que hagan lo que quiera implementando el *trait* `Iterator`
en sus propios tipos. Como se mencionó anteriormente, el único método para el
que debe proporcionar una definición es el método `next`. Una vez que haya
hecho eso, puede usar todos los demás métodos que tengan implementaciones
predeterminadas proporcionadas por el *trait* `Iterator`.

Para demostrar, creemos un iterador que solo contará del 1 al 5. Primero,
crearemos una estructura para contener algunos valores. Luego, haremos de
esta estructura un iterador implementando el *trait* `Iterator` y utilizando
los valores en esa implementación.

El listado 13-20 tiene la definición de la estructura `Counter` y una
función `new` asociada para crear instancias de `Counter`:

<span class="filename">Filename: src/lib.rs</span>

```rust
struct Counter {
    count: u32,
}

impl Counter {
    fn new() -> Counter {
        Counter { count: 0 }
    }
}
```

<span class="caption">Listado 13-20: Definición de la estructura `Counter` y
una función `new` que crea instancias de `Counter` con un valor inicial de 0
para `count`</span>

La estructura `Counter` tiene un campo llamado `count`. Este campo contiene
un valor `u32` que mantendrá un registro de dónde estamos en el proceso de
iteración de 1 a 5. El campo `count` es privado porque queremos que la
implementación de `Counter` administre su valor. La función `new` impone el
comportamiento de comenzar siempre nuevas instancias con un valor de 0 en el
campo `count`.

A continuación, implementaremos el *trait* `Iterator` para nuestro tipo
`Counter` definiendo el cuerpo del método `next` para especificar qué
queremos que suceda cuando se use este iterador, como se muestra en el
Listado 13-21:

<span class="filename">Filename: src/lib.rs</span>

```rust
# struct Counter {
#     count: u32,
# }
#
impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;

        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}
```

<span class="caption">Listado 13-21: Implementando el *trait* `Iterator` en
nuestra estructura `Counter`</span>

Configuramos el tipo `Item` asociado para nuestro iterador en `u32`, lo que
significa que el iterador devolverá los valores `u32`. Nuevamente, no se
preocupe por los tipos asociados todavía, los cubriremos en el Capítulo 19.

Queremos que nuestro iterador agregue 1 al estado actual, por lo que
inicializamos `count` a 0 para que devuelva 1 primero. Si el valor de `count`
es menor que 6, `next` devolverá el valor actual envuelto en `Some`, pero si
`count` es 6 o superior, nuestro iterador devolverá `None`.

#### Usando nuestro *Iterator’s* método `Counter` `next`

Una vez que implementamos el *trait* `Iterator`, ¡tenemos un iterador! El
listado 13-22 muestra una prueba que demuestra que podemos usar la
funcionalidad de iterador de nuestra estructura `Counter` llamando
directamente al método `next`, tal como lo hicimos con el iterador creado a
partir de un vector en el listado 13-15.

<span class="filename">Filename: src/lib.rs</span>

```rust
# struct Counter {
#     count: u32,
# }
#
# impl Iterator for Counter {
#     type Item = u32;
#
#     fn next(&mut self) -> Option<Self::Item> {
#         self.count += 1;
#
#         if self.count < 6 {
#             Some(self.count)
#         } else {
#             None
#         }
#     }
# }
#
#[test]
fn calling_next_directly() {
    let mut counter = Counter::new();

    assert_eq!(counter.next(), Some(1));
    assert_eq!(counter.next(), Some(2));
    assert_eq!(counter.next(), Some(3));
    assert_eq!(counter.next(), Some(4));
    assert_eq!(counter.next(), Some(5));
    assert_eq!(counter.next(), None);
}
```

<span class="caption">Listado 13-22: Prueba de la funcionalidad de la
implementación del método `next`</span>

Esta prueba crea una nueva instancia `Counter` en la variable `counter` y
luego llama `next` repetidamente, verificando que hemos implementado el
comportamiento que queremos que tenga este iterador: devolviendo los valores
de 1 a 5.

#### Uso de otros métodos de *trait* `iterador`

Implementamos el *trait* `Iterator` definiendo el método `next`, por lo que
ahora podemos usar cualquier implementación predeterminada del método
`Iterator` según se define en la biblioteca estándar, ya que todos usan la
funcionalidad del método `next`.

Por ejemplo, si por algún motivo quisiéramos tomar los valores producidos por
una instancia de `Counter`, emparejarlos con los valores producidos por otra
instancia `Counter` después de omitir el primer valor, multiplicar cada par,
mantener solo los resultados que son divisible por 3, y agregue todos los
valores resultantes, podríamos hacerlo, como se muestra en la prueba en el
listado 13-23:

<span class="filename">Filename: src/lib.rs</span>

```rust
# struct Counter {
#     count: u32,
# }
#
# impl Counter {
#     fn new() -> Counter {
#         Counter { count: 0 }
#     }
# }
#
# impl Iterator for Counter {
#     // Our iterator will produce u32s
#     type Item = u32;
#
#     fn next(&mut self) -> Option<Self::Item> {
#         // increment our count. This is why we started at zero.
#         self.count += 1;
#
#         // check to see if we've finished counting or not.
#         if self.count < 6 {
#             Some(self.count)
#         } else {
#             None
#         }
#     }
# }
#
#[test]
fn using_other_iterator_trait_methods() {
    let sum: u32 = Counter::new().zip(Counter::new().skip(1))
                                 .map(|(a, b)| a * b)
                                 .filter(|x| x % 3 == 0)
                                 .sum();
    assert_eq!(18, sum);
}
```

<span class="caption">Listado 13-23: Usando una variedad de métodos del
*trait*`Iterator` en nuestro iterador `Counter`</span>

Tenga en cuenta que `zip` produce solo cuatro pares; el quinto par teórico
`(5, None)` nunca se produce porque `zip` devuelve `None` cuando cualquiera
de sus iteradores de entrada devuelve `None`.

Todas estas llamadas a métodos son posibles porque especificamos cómo
funciona el método `next` y la biblioteca estándar proporciona
implementaciones predeterminadas para otros métodos que llaman `next`.
