## Tratamiento de punteros inteligentes como referencias regulares con el *trait* `Deref`

La implementación del *trait* `Deref` le permite personalizar el
comportamiento del *operador de desreferencia*,`*`(en oposición al operador
de multiplicación u *glob operator*). Al implementar `Deref` de tal manera
que un puntero inteligente puede tratarse como una referencia regular, puede
escribir código que opera en referencias y usar ese código también con
punteros inteligentes.

Primero veamos cómo funciona el operador de desreferencia con referencias
regulares. Luego trataremos de definir un tipo personalizado que se comporte
como `Box <T>`, y veremos por qué el operador de desreferencia no funciona
como una referencia en nuestro tipo recién definido. Exploraremos cómo la
implementación del *trait* `Deref` hace posible que los punteros inteligentes
funcionen de manera similar a las referencias. A continuación, veremos la
función *coercción directa* (*deref coercion*) de Rust y cómo nos permite trabajar con
referencias o punteros inteligentes.

### Seguir el puntero hasta el valor con el operador de desreferencia

Una referencia regular es un tipo de puntero, y una forma de pensar en un
puntero es como una flecha hacia un valor almacenado en otro lugar. En el
Listado 15-6, creamos una referencia a un valor `i32` y luego usamos el
operador de referencia para seguir la referencia a los datos:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = 5;
    let y = &x;

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

<span class="caption">Listado 15-6: Usar el operador de desreferencia para
seguir una referencia a un valor `i32`</span>

La variable `x` tiene un valor `i32`, `5`. Establecemos `y` igual a una
referencia a `x`. Podemos afirmar que `x` es igual a `5`. Sin embargo, si
queremos hacer una afirmación sobre el valor en `y`, tenemos que usar `*y`
para seguir la referencia al valor al que apunta (de ahí *desreferencia*).
Una vez que desreferenciamos `y`, tenemos acceso al valor entero `y` apunta a
que podemos comparar con `5`.

Si intentáramos escribir `assert_eq!(5, y);` en su lugar, obtendríamos este
error de compilación:

```text
error[E0277]: the trait bound `{integer}: std::cmp::PartialEq<&{integer}>` is
not satisfied
 --> src/main.rs:6:5
  |
6 |     assert_eq!(5, y);
  |     ^^^^^^^^^^^^^^^^^ can't compare `{integer}` with `&{integer}`
  |
  = help: the trait `std::cmp::PartialEq<&{integer}>` is not implemented for
  `{integer}`
```

No se permite comparar un número y una referencia a un número porque son
tipos diferentes. Debemos usar el operador de desreferencia para seguir la
referencia al valor al que apunta.

### Usando `Box <T>` como una a referencia

Podemos reescribir el código en el Listado 15-6 para usar un `Box<T>` en
lugar de una referencia; el operador de desreferencia funcionará como se
muestra en el Listado 15-7:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = 5;
    let y = Box::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

<span class="caption">Listado 15-7: Usar el operador de desreferencia en un
`Box<i32>`</span>

La única diferencia entre el Listado 15-7 y el Listado 15-6 es que aquí
establecemos que `y` sea una instancia de un *box* que apunta al valor en `x`
en lugar de una referencia que apunta al valor de `x`. En la última
afirmación, podemos usar el operador de desreferencia para seguir el puntero
del *box* de la misma manera que lo hacíamos cuando `y` era una referencia. A
continuación, exploraremos qué tiene de especial el `Box <T>` que nos permite
usar el operador de referencia al definir nuestro propio tipo de *box*.

### Definiendo nuestro propio puntero inteligente

Construyamos un puntero inteligente similar al tipo `Box<T>` provisto por la
biblioteca estándar para experimentar cómo los punteros inteligentes se
comportan de manera diferente a las referencias por defecto. Luego veremos
cómo agregar la capacidad de usar el operador de desreferencia.

El tipo `Box<T>` se define en última instancia como una estructura tupla con
un elemento, por lo que el Listado 15-8 define un tipo `MyBox<T>` de la misma
manera. También definiremos una función `new` para que coincida con la
función `new` definida en `Box<T>`.

<span class="filename">Filename: src/main.rs</span>

```rust
struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}
```

<span class="caption">Listado 15-8: Definiendo un tipo `MyBox<T>`</span>

Definimos una estructura llamada `MyBox` y declaramos un parámetro genérico
`T`, porque queremos que nuestro tipo contenga valores de cualquier tipo. El
tipo `MyBox` es una estructura tuple con un elemento de tipo `T`. La función
`MyBox::new` toma un parámetro de tipo `T` y devuelve una instancia `MyBox`
que mantiene el valor pasado.

Intentemos agregar la función `main` en el Listado 15-7 al Listado 15-8 y
cambiarla para usar el tipo `MyBox<T>` que hemos definido en lugar de
`Box<T> `. El código del listado 15-9 no se compilará porque Rust no sabe
cómo desreferenciar `MyBox`.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

<span class="caption">Listado 15-9: Intentando usar `MyBox<T>` de la misma
manera que utilizamos las referencias y `Box<T>`</span>

Aquí está el error de compilación resultante:

```text
error[E0614]: type `MyBox<{integer}>` cannot be dereferenced
  --> src/main.rs:14:19
   |
14 |     assert_eq!(5, *y);
   |                   ^^
```

Nuestro tipo `MyBox<T>` no puede ser desreferenciado porque no hemos
implementado esa capacidad en nuestro tipo. Para habilitar la
desreferenciación con el operador `*`, implementamos el *trait* `Deref`.

### Tratar un tipo como una referencia mediante la implementación del *trait* `Deref`

Como se discutió en el Capítulo 10, para implementar un *trait*, necesitamos
proporcionar implementaciones para los métodos requeridos del *trait*. El
*trait* `Deref`, proporcionado por la biblioteca estándar, requiere que
implementemos un método llamado `deref` que toma `self` y devuelve una
referencia a los datos internos. El listado 15-10 contiene una implementación
de `Deref` para agregar a la definición de `MyBox`:

<span class="filename">Filename: src/main.rs</span>

```rust
use std::ops::Deref;

# struct MyBox<T>(T);
impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}
```

<span class="caption">Listado 15-10: Implementando `Deref` en
`MyBox<T>`</span>

La sintaxis `type Target = T;` define un tipo asociado para el *trait*
`Deref` a usar. Los tipos asociados son una manera ligeramente diferente de
declarar un parámetro genérico, pero no necesita preocuparse por ellos por el
momento; los cubriremos con más detalle en el Capítulo 19.

Completamos el cuerpo del método `deref` con `&self.0` para que `deref`
devuelva una referencia al valor al que queremos acceder con el operador
`*`. La función `main` en el Listado 15-9 que llama a `*` en el valor
`MyBox<T>`ahora se compila, ¡y las aserciones pasan!

Sin el *trait* `Deref`, el compilador solo puede desreferenciar `&`
referencias. El método `deref` le da al compilador la capacidad de tomar un
valor de cualquier tipo que implemente `Deref` y llame al método `deref` para
obtener una referencia `&` que sepa cómo desreferencia.

Cuando ingresamos `*y` en el Listado 15-9, entre bastidores, Rust
realmente ejecutó este código:

```rust,ignore
*(y.deref())
```

Rust sustituye el operador `*` con una llamada al método `deref` y luego una
desreferencia simple para que no tengamos que pensar si debemos o no llamar
al método `deref`. Esta función Rust nos permite escribir código que funciona
de manera idéntica si tenemos una referencia regular o un tipo que implementa
`Deref`.

La razón por la cual el método `deref` devuelve una referencia a un valor y
que la desreferencia simple fuera de los paréntesis en `*(y.deref ())` aún
es necesaria es el sistema de propiedad. Si el método `deref` devolviera el
valor directamente en lugar de una referencia al valor, el valor se movería
fuera de `self`. No queremos tomar posesión del valor interno dentro de
`MyBox<T>` en este caso o en la mayoría de los casos donde utilizamos el
operador de desreferencia.

Tenga en cuenta que el operador `*` se reemplaza con una llamada al método
`deref` y luego una llamada al operador `*` solo una vez, cada vez que usemos
un `*` en nuestro código. Debido a que la sustitución del operador `*` no se
repite infinitamente, terminamos con datos del tipo `i32`, que coincide con
`5` en `assert_eq!` En el Listado 15-9.

### Implicaciones *Deref* implícitas con funciones y métodos

*Deref coercion* es una conveniencia que Rust realiza en argumentos a
funciones y métodos. *Deref* coercion convierte una referencia a un tipo que
implementa `Deref` en una referencia a un tipo al que `Deref` puede convertir
el tipo original. La coerción de *Deref* ocurre automáticamente cuando
pasamos una referencia al valor de un tipo particular como un argumento para
una función o método que no coincide con el tipo de parámetro en la
definición de función o método. Una secuencia de llamadas al método `deref`
convierte el tipo que proporcionamos en el tipo que el parámetro necesita.

La *deref coercion* se agregó a Rust para que los programadores que escriben
funciones y llamadas a métodos no necesiten agregar tantas referencias
explícitas y referencias con `&` y `*`. La característica de *deref coercion*
también nos permite escribir más código que puede funcionar para referencias
o punteros inteligentes.

Para ver la *deref coercion* en acción, utilicemos el tipo `MyBox<T>` que
definimos en el Listado 15-8, así como la implementación de `Deref` que
agregamos en el Listado 15-10. El listado 15-11 muestra la definición de una
función que tiene un parámetro de *string slice*:

<span class="filename">Filename: src/main.rs</span>

```rust
fn hello(name: &str) {
    println!("Hello, {}!", name);
}
```

<span class="caption">Listado 15-11: Una función `hello` que tiene el
parámetro `name` de tipo `&str`</span>

Podemos llamar a la función `hello` con un *string slice* como argumento,
como `hello("Rust");`por ejemplo. La coerción de *Deref* hace posible llamar
a `hello` con una referencia a un valor de tipo `MyBox <String> `, como se
muestra en el Listado 15-12:

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::ops::Deref;
#
# struct MyBox<T>(T);
#
# impl<T> MyBox<T> {
#     fn new(x: T) -> MyBox<T> {
#         MyBox(x)
#     }
# }
#
# impl<T> Deref for MyBox<T> {
#     type Target = T;
#
#     fn deref(&self) -> &T {
#         &self.0
#     }
# }
#
# fn hello(name: &str) {
#     println!("Hello, {}!", name);
# }
#
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
}
```

<span class="caption">Listado 15-12: Llamar a `hello` con una referencia a un
valor `MyBox<String>`, que funciona debido a la *deref coercion*</span>

Aquí llamamos a la función `hello` con el argumento `&m`, que es una
referencia a un valor `MyBox<String>`. Debido a que implementamos el *trait* `Deref` en `MyBox<T>` en el Listado 15-10, Rust puede convertir
`&MyBox<String>` en `&String` llamando a `deref`. La biblioteca estándar
proporciona una implementación de `Deref` en `String` que devuelve un
*string slice*, y esto está en la documentación de API para `Deref`. Rust
llama `deref` nuevamente para convertir `&String` en `&str`, que coincide con
la definición de la función `hello`.

Si Rust no implementó la *deref coercion*, tendríamos que escribir el código
en el Listado 15-13 en lugar del código en el Listado 15-12 para llamar a
`hello` con un valor de tipo `& MyBox<String>`.

<span class="filename">Filename: src/main.rs</span>

```rust
# use std::ops::Deref;
#
# struct MyBox<T>(T);
#
# impl<T> MyBox<T> {
#     fn new(x: T) -> MyBox<T> {
#         MyBox(x)
#     }
# }
#
# impl<T> Deref for MyBox<T> {
#     type Target = T;
#
#     fn deref(&self) -> &T {
#         &self.0
#     }
# }
#
# fn hello(name: &str) {
#     println!("Hello, {}!", name);
# }
#
fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&(*m)[..]);
}
```

<span class="caption">Listado 15-13: El código que deberíamos escribir si
Rust no tuviera *deref coercion*</span>

El `(*m)` elimina la referencia del `MyBox<String>` en un `String`. Luego,
los `&` y `[..]` toman un *string slice* de `String` que es igual a toda el
*string* para que coincida con la firma de `hello`. El código sin
*deref coercion* es más difícil de leer, escribir y comprender con todos
estos símbolos involucrados. La *deref coercion* le permite a Rust manejar
estas conversiones automáticamente.

Cuando se define el *trait* `Deref` para los tipos involucrados, Rust
analizará los tipos y usará `Deref::deref` tantas veces como sea necesario
para obtener una referencia que coincida con el tipo del parámetro. ¡La
cantidad de veces que `Deref::deref` necesita insertarse se resuelve en
tiempo de compilación, por lo que no hay penalización de tiempo de ejecución
para aprovechar la *deref coercion*!

### Cómo *Deref Coercion* interactúa con la mutabilidad

De forma similar a cómo usa el *trait* `Deref` para anular el operador `*` en
referencias inmutables, puede usar el *trait* `DerefMut` para anular el
operador `*` en referencias mutables.

Rust hace *deref coercion* cuando encuentra tipos e implementaciones del
*trait* en tres casos:

* De `&T` a `&U` cuando `T: Deref<Target=U>`
* De `&mut T` a `&mut U` cuando `T: DerefMut<Target=U>`
* De `&mut T` a `&U` cuando `T: Deref<Target=U>`

Los primeros dos casos son iguales excepto para la mutabilidad. El primer
caso establece que si tiene un `&T`, y `T` implementa `Deref` para algún
tipo `U`, puede obtener un `&U` de forma transparente. El segundo caso indica
que la misma *deref coercion* ocurre para las referencias mutables.

El tercer caso es más complicado: Rust también forzará una referencia mutable
a una inmutable. Pero lo contrario *no* es posible: las referencias
inmutables nunca forzarán a las referencias mutables. Debido a las reglas de
préstamo, si tiene una referencia mutable, esa referencia mutable debe ser la
única referencia a esa información (de lo contrario, el programa no
compilaría). La conversión de una referencia mutable a una referencia
inmutable nunca romperá las reglas de endeudamiento. La conversión de una
referencia inmutable a una referencia mutable requeriría que haya solo una
referencia inmutable a esa información, y las reglas de endeudamiento no lo
garantizan. Por lo tanto, Rust no puede asumir que es posible convertir una
referencia inmutable en una referencia mutable.
