## Funciones avanzadas y *Closures*

Finalmente, exploraremos algunas funciones avanzadas relacionadas con
funciones y *closures*, que incluyen punteros de funciones y *closures* de retorno.

### Punteros de función

Hemos hablado sobre cómo pasar *closures* a las funciones; ¡también puede
pasar funciones regulares a funciones! Esta técnica es útil cuando desea
pasar una función que ya ha definido en lugar de definir un nuevo *closure*.
Hacer esto con punteros de función le permitirá usar funciones como
argumentos para otras funciones. Las funciones fuerzan al tipo `fn` (con una
f minúscula), que no debe confundirse con el *trait* de *closure* `Fn`. El
tipo `fn` se llama *puntero de función*
(*function pointer*). La sintaxis para especificar que un parámetro es un
puntero a la función es similar a la de los *closures*, como se muestra en el
Listado 19-35.

<span class="filename">Filename: src/main.rs</span>

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}

fn do_twice(f: fn(i32) -> i32, arg: i32) -> i32 {
    f(arg) + f(arg)
}

fn main() {
    let answer = do_twice(add_one, 5);

    println!("The answer is: {}", answer);
}
```

<span class="caption">Listado 19-35: Usar el tipo `fn` para aceptar un
puntero a la función como argumento</span>

Este código imprime `The answer is: 12`. Especificamos que el parámetro `f`
en `do_twice` es un `fn` que toma un parámetro de tipo `i32` y devuelve un
`i32`. Entonces podemos llamar a `f` en el cuerpo de `do_twice`. En `main`,
podemos pasar el nombre de función `add_one` como primer argumento a
`do_twice`.

A diferencia de los *closures*, `fn` es un tipo en lugar de un *trait*, por
lo que especificamos `fn` como el tipo de parámetro directamente en lugar de
declarar un parámetro de tipo genérico con uno de los *traits* `Fn` como un
*trait bound*.

Los punteros de función implementan los tres *traits* de *closure*
(`Fn`, `FnMut` y `FnOnce`), por lo que siempre puede pasar un puntero a la
función como argumento para una función que espera un *closure*. Lo mejor es
escribir funciones usando un tipo genérico y uno de los *traits* de *closure*
para que sus funciones puedan aceptar funciones o *closures*.

Un ejemplo de dónde solo desearía aceptar `fn` y no *closures* es cuando
interactúa con un código externo que no tiene *closures*: las funciones C
pueden aceptar funciones como argumentos, pero C no tiene *closures*.

Como ejemplo de dónde podría usar un *closure* definido en línea o una
función nombrada, veamos el uso de `map`. Para usar la función `map` para
convertir un vector de números en un vector de *string*, podríamos usar un
*closure*, como este:

```rust
let list_of_numbers = vec![1, 2, 3];
let list_of_strings: Vec<String> = list_of_numbers
    .iter()
    .map(|i| i.to_string())
    .collect();
```

O podríamos nombrar una función como el argumento para `map` en lugar del
*closure*, así:

```rust
let list_of_numbers = vec![1, 2, 3];
let list_of_strings: Vec<String> = list_of_numbers
    .iter()
    .map(ToString::to_string)
    .collect();
```

Tenga en cuenta que debemos usar la sintaxis totalmente calificada de la que
hablamos anteriormente en la sección “*Traits* avanzados” porque hay varias
funciones disponibles llamadas `to_string`. Aquí, estamos usando la función
`to_string` definida en el *trait* `ToString`, que la biblioteca estándar ha
implementado para cualquier tipo que implemente `Display`.

Algunas personas prefieren este estilo, y algunas personas prefieren usar
*closures*. Terminan compilando el mismo código, por lo que debe usar el
estilo que le resulte más claro.

### Returning Closures

Los *closures* están representados por *traits*, lo que significa que no
puede devolver los *closures* directamente. En la mayoría de los casos en los
que es posible que desee devolver un *trait*, en su lugar puede usar el tipo
concreto que implementa el *trait* como el valor de retorno de la función.
Pero no se puede hacer eso con *closures* porque no tienen un tipo concreto
que sea retornable; no está permitido usar el puntero de función `fn` como un
tipo de retorno, por ejemplo.

El siguiente código intenta devolver un *closure* directamente, pero no se
compilará:

```rust,ignore
fn returns_closure() -> Fn(i32) -> i32 {
    |x| x + 1
}
```

El error del compilador es el siguiente:

```text
error[E0277]: the trait bound `std::ops::Fn(i32) -> i32 + 'static:
std::marker::Sized` is not satisfied
 -->
  |
1 | fn returns_closure() -> Fn(i32) -> i32 {
  |                         ^^^^^^^^^^^^^^ `std::ops::Fn(i32) -> i32 + 'static`
  does not have a constant size known at compile-time
  |
  = help: the trait `std::marker::Sized` is not implemented for
  `std::ops::Fn(i32) -> i32 + 'static`
  = note: the return type of a function must have a statically known size
```

¡El error hace referencia al *trait* `Sized` de nuevo! Rust no sabe cuánto
espacio necesitará para almacenar el *closure*. Vimos una solución a este
problema antes. Podemos usar un *trait object*:

```rust
fn returns_closure() -> Box<Fn(i32) -> i32> {
    Box::new(|x| x + 1)
}
```

Este código compilará muy bien. Para obtener más información sobre los
*trait objects*, consulte la sección “Uso de *trait objects* que permiten
valores de diferentes tipos” en el Capítulo 17.

## Resumen

¡Uf! Ahora tiene algunas características de Rust en su caja de herramientas
que no usará con frecuencia, pero sabrá que están disponibles en
circunstancias muy particulares. Hemos introducido varios temas complejos
para que cuando los encuentres en sugerencias de mensajes de error o en el
código de otras personas, puedas reconocer estos conceptos y la sintaxis. Use
este capítulo como referencia para guiarlo a las soluciones.

A continuación, pondremos en práctica todo lo que hemos discutido a lo largo
del libro y haremos un proyecto más.
