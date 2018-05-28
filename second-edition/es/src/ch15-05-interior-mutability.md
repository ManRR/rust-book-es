## `RefCell <T>` y el patrón de mutabilidad interior

<!-- NEXT PARAGRAPH WRAPPED WEIRD INTENTIONALLY SEE #199 -->

*Interior mutability* es un patrón de diseño en Rust que le permite mutar
datos incluso cuando hay referencias inmutables a esos datos; normalmente,
esta acción no está permitida por las reglas de endeudamiento. Para mutar los
datos, el patrón usa el código `unsafe` dentro de una estructura de datos
para "doblar" las reglas habituales de Rust que rigen la mutación y el
préstamo. Todavía no hemos cubierto el código inseguro; lo haremos en el
Capítulo 19. Podemos usar tipos que usan el patrón de mutabilidad interior
cuando podemos asegurarnos de que se seguirán las reglas de endeudamiento en
el tiempo de ejecución, aunque el compilador no pueda garantizarlo. El código
`unsafe` involucrado se envuelve en una API segura, y el tipo externo sigue
siendo inmutable.

Exploremos este concepto mirando el tipo `RefCell <T>` que sigue el patrón de
mutabilidad interior.

### Aplicación de las reglas de préstamo en tiempo de ejecución con `RefCell <T>`

A diferencia de `Rc<T>`, el tipo `RefCell<T>` representa propiedad única
sobre los datos que contiene. Entonces, ¿qué hace `RefCell<T>` diferente de
un tipo como `Box<T>`? Recuerde las reglas de préstamo que aprendió en el
Capítulo 4:

* En cualquier momento dado, puede tener *cualquiera* (pero no ambos) una
 referencia mutable o cualquier número de referencias inmutables.
* Las referencias siempre deben ser válidas.

Con referencias y `Box<T>`, las invariantes de las reglas de préstamo se
aplican en tiempo de compilación. Con `RefCell<T>`, estas invariantes se
aplican *en tiempo de ejecución*. Con referencias, si rompe estas reglas,
obtendrá un error de compilación. Con `RefCell <T>`, si rompe estas reglas,
su programa entrará en pánico y saldrá.

Las ventajas de verificar las reglas de endeudamiento en tiempo de
compilación son que los errores se detectarán antes en el proceso de
desarrollo, y no hay impacto en el rendimiento del tiempo de ejecución porque
todo el análisis se completa de antemano. Por esas razones, verificar las
reglas de endeudamiento en el momento de la compilación es la mejor opción en
la mayoría de los casos, por lo que este es el valor predeterminado de Rust.

La ventaja de verificar las reglas de endeudamiento en tiempo de ejecución es
que ciertos escenarios seguros para la memoria están permitidos, mientras que
las verificaciones en tiempo de compilación no los permiten. El análisis
estático, como el compilador Rust, es intrínsecamente conservador. Algunas
propiedades del código son imposibles de detectar al analizar el código: el
ejemplo más famoso es el Problema de Detención, que está más allá del alcance
de este libro, pero es un tema interesante de investigar.

Debido a que algunos análisis son imposibles, si el compilador Rust no puede
estar seguro de que el código cumple con las reglas de propiedad, podría
rechazar un programa correcto; de esta manera, es conservador. Si Rust
aceptara un programa incorrecto, los usuarios no podrían confiar en las
garantías que Rust ofrece. Sin embargo, si Rust rechaza un programa correcto,
el programador tendrá inconvenientes, pero nada catastrófico puede ocurrir.
El tipo `RefCell<T>` es útil cuando estás seguro de que tu código sigue las
reglas de préstamo, pero el compilador no puede comprenderlo ni garantizarlo.

Similar para `Rc<T>`, `RefCell<T>` solo se usa en escenarios de subproceso
único y le dará un error en tiempo de compilación si intenta usarlo en un
contexto multiproceso. Hablaremos sobre cómo obtener la funcionalidad de
`RefCell <T>` en un programa multiproceso en el Capítulo 16.

Aquí hay un resumen de las razones para elegir `Box<T>`, `Rc<T>`, o
`RefCell<T>`:

* `Rc<T>` permite múltiples propietarios de los mismos datos; `Box<T>` y
 `RefCell <T>` tienen propietarios únicos.
* `Box <T>` permite tomar prestados inmutables o mutables en tiempo de
 compilación; `Rc<T>` solo permite tomar prestados inmutables en tiempo de
 compilación; `RefCell<T>` permite tomar prestados inmutables o mutables en
 el tiempo de ejecución.
* Debido a que `RefCell<T>` permite que los prestamos mutables se verifiquen
 en el tiempo de ejecución, puede mutar el valor dentro de `RefCell<T>` incluso cuando `RefCell <T>` sea inmutable.

La mutación del valor dentro de un valor inmutable es el patrón
*demutabilidad interior* (*interior mutability*). Veamos una situación en la
que la mutabilidad interior es útil y examinemos cómo es posible.

### Mutabilidad interior: Un préstamo mutable a un valor inmutable

Una consecuencia de las reglas de endeudamiento es que cuando tienes un valor
inmutable, no puedes tomarlo de forma mutable. Por ejemplo, este código no se
compilará:

```rust,ignore
fn main() {
    let x = 5;
    let y = &mut x;
}
```

Si intentaste compilar este código, obtendrías el siguiente error:

```text
error[E0596]: cannot borrow immutable local variable `x` as mutable
 --> src/main.rs:3:18
  |
2 |     let x = 5;
  |         - consider changing this to `mut x`
3 |     let y = &mut x;
  |                  ^ cannot borrow mutably
```

Sin embargo, hay situaciones en las que sería útil que un valor se mute en
sus métodos pero que parezca inmutable a otro código. El código fuera de los
métodos del valor no podría mutar el valor. El uso de `RefCell<T>` es una
forma de obtener la capacidad de tener mutabilidad interior. Pero
`RefCell<T>` no elude por completo las reglas de endeudamiento: el
verificador de préstamo en el compilador permite esta mutabilidad interior, y
las reglas de endeudamiento se verifican en tiempo de ejecución. Si infringe
las reglas, obtendrá un "¡pánico!" En lugar de un error de compilación.

Analicemos un ejemplo práctico donde podemos usar `RefCell<T>` para mutar un
valor inmutable y ver por qué es útil.

#### Un caso de uso para la mutación interior: *Mock Objects*

Un *test double* es el concepto de programación general para un tipo
utilizado en lugar de otro tipo durante la prueba. *Los objetos simulados*
(*Mock objects*) son tipos específicos para *test double* que registran lo
que sucede durante una prueba para que pueda afirmar que tuvieron lugar las
acciones correctas.

Rust no tiene objetos en el mismo sentido en que otros lenguajes tienen
objetos, y Rust no tiene una funcionalidad de objeto simulada incorporada en
la biblioteca estándar como lo hacen otros lenguajes. Sin embargo,
definitivamente puede crear una estructura que sirva los mismos propósitos
que un objeto simulado.

Este es el escenario que probaremos: crearemos una biblioteca que rastrea un
valor contra un valor máximo y envía mensajes en función de qué tan cerca
está el valor máximo del valor actual. Esta biblioteca podría usarse para
realizar un seguimiento de la cuota de un usuario para el número de llamadas
a la API que pueden hacer, por ejemplo.

Nuestra biblioteca solo brindará la funcionalidad de rastrear cuán cerca está
el máximo de un valor y cuáles deberían ser los mensajes en qué momento. Se
espera que las aplicaciones que usan nuestra biblioteca proporcionen el
mecanismo para enviar los mensajes: la aplicación podría poner un mensaje en
la aplicación, enviar un correo electrónico, enviar un mensaje de texto u
otra cosa. La biblioteca no necesita saber ese detalle. Todo lo que necesita
es algo que implemente un rasgo que proporcionaremos llamado `Messenger`. El
listado 15-20 muestra el código de la biblioteca:

<span class="filename">Filename: src/lib.rs</span>

```rust
pub trait Messenger {
    fn send(&self, msg: &str);
}

pub struct LimitTracker<'a, T: 'a + Messenger> {
    messenger: &'a T,
    value: usize,
    max: usize,
}

impl<'a, T> LimitTracker<'a, T>
    where T: Messenger {
    pub fn new(messenger: &T, max: usize) -> LimitTracker<T> {
        LimitTracker {
            messenger,
            value: 0,
            max,
        }
    }

    pub fn set_value(&mut self, value: usize) {
        self.value = value;

        let percentage_of_max = self.value as f64 / self.max as f64;

        if percentage_of_max >= 0.75 && percentage_of_max < 0.9 {
            self.messenger.send("Warning: You've used up over 75% of your quota!");
        } else if percentage_of_max >= 0.9 && percentage_of_max < 1.0 {
            self.messenger.send("Urgent warning: You've used up over 90% of your quota!");
        } else if percentage_of_max >= 1.0 {
            self.messenger.send("Error: You are over your quota!");
        }
    }
}
```

<span class="caption">Listado 15-20: Una biblioteca para hacer un seguimiento
de cuán cerca está un valor de un valor máximo y advertir cuando el valor
está en ciertos niveles</span>

Una parte importante de este código es que el *trait* `Messenger` tiene un
método llamado `send` que toma una referencia inmutable a `self` y al texto
del mensaje. Esta es la interfaz que nuestro *mock object* necesita tener.
La otra parte importante es que queremos probar el comportamiento del método
`set_value` en `LimitTracker`. Podemos cambiar lo que pasamos para el
parámetro `valor`, pero `set_value` no nos devuelve nada para hacer
afirmaciones. Queremos poder decir que si creamos un `LimitTracker` con algo
que implementa el *trait* `Messenger` y un valor particular para `max`,
cuando pasamos diferentes números para `value`, se le dice al mensajero que
envíe el mensaje apropiado .

Necesitamos un *mock object* que, en lugar de enviar un correo electrónico o
un mensaje de texto cuando llamemos a `send`, solo haga un seguimiento de los
mensajes que se le dice que envíe. Podemos crear una nueva instancia del
*mock object*, crear un `LimitTracker` que use el *mock object*, llamar al
método `set_value` en `LimitTracker`, y luego verificar que el *mock object*
tenga los mensajes que esperamos. El listado 15-21 muestra un intento de
implementar un *mock object* para hacer justamente eso, pero el *comprobador
de préstamos* (*borrow checker*) no lo permite:

<span class="filename">Filename: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    use super::*;

    struct MockMessenger {
        sent_messages: Vec<String>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger { sent_messages: vec![] }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        let mock_messenger = MockMessenger::new();
        let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);

        limit_tracker.set_value(80);

        assert_eq!(mock_messenger.sent_messages.len(), 1);
    }
}
```

<span class="caption">Listado 15-21: Un intento de implementar un
`MockMessenger` que no está permitido por el
*comprobador de préstamos* (*borrow checker*)</span>

Este código de prueba define una estructura `MockMessenger` que tiene un
campo `send_messages` con valores `Vec` de `String` para hacer un seguimiento
de los mensajes que se le dice que envíe. También definimos una función
asociada `new` para que sea conveniente crear nuevos valores `MockMessenger`
que comiencen con una lista vacía de mensajes. Luego implementamos el *trait*
`Messenger` para `MockMessenger` para que podamos dar un `MockMessenger` a
`LimitTracker`. En la definición del método `send`, tomamos el mensaje pasado
como un parámetro y lo almacenamos en la lista `MockMessenger` de
`sent_messages`.

En la prueba, estamos probando qué sucede cuando se le dice al `LimitTracker`
que establezca `value` en algo que sea más del 75% del valor `max`. Primero,
creamos un nuevo `MockMessenger`, que comenzará con una lista vacía de
mensajes. Luego creamos un nuevo `LimitTracker` y le damos una referencia al
nuevo `MockMessenger` y un valor `max` de 100. Llamamos al método `set_value`
en `LimitTracker` con un valor de 80, que es más que 75 por ciento de 100.
Luego afirmamos que la lista de mensajes que el `MockMessenger` sigue de
cerca debería tener ahora un mensaje.

Sin embargo, hay un problema con esta prueba, como se muestra aquí:

```text
error[E0596]: cannot borrow immutable field `self.sent_messages` as mutable
  --> src/lib.rs:52:13
   |
51 |         fn send(&self, message: &str) {
   |                 ----- use `&mut self` here to make mutable
52 |             self.sent_messages.push(String::from(message));
   |             ^^^^^^^^^^^^^^^^^^ cannot mutably borrow immutable field
```

No podemos modificar el `MockMessenger` para realizar un seguimiento de los
mensajes, porque el método `send` toma una referencia inmutable a `self`.
Tampoco podemos tomar la sugerencia del texto de error para usar `&mut self`
en su lugar, porque entonces la firma de `send` no coincidiría con la firma
en la definición de rasgo `Messenger` (no dude en probar y ver qué error
mensaje que obtienes).

¡Esta es una situación en la que la mutabilidad interior puede ayudar!
Almacenaremos `sent_messages` dentro de `RefCell<T>`, y luego el mensaje
`send` podrá modificar `send_messages` para almacenar los mensajes que hemos
visto. El listado 15-22 muestra cómo se ve:

<span class="filename">Filename: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use std::cell::RefCell;

    struct MockMessenger {
        sent_messages: RefCell<Vec<String>>,
    }

    impl MockMessenger {
        fn new() -> MockMessenger {
            MockMessenger { sent_messages: RefCell::new(vec![]) }
        }
    }

    impl Messenger for MockMessenger {
        fn send(&self, message: &str) {
            self.sent_messages.borrow_mut().push(String::from(message));
        }
    }

    #[test]
    fn it_sends_an_over_75_percent_warning_message() {
        // --snip--
#         let mock_messenger = MockMessenger::new();
#         let mut limit_tracker = LimitTracker::new(&mock_messenger, 100);
#         limit_tracker.set_value(75);

        assert_eq!(mock_messenger.sent_messages.borrow().len(), 1);
    }
}
```

<span class="caption">Listado 15-22: Usar `RefCell<T>` para mutar un valor
interno mientras que el valor externo se considera inmutable</span>

El campo `sent_messages` ahora es de tipo` RefCell<Vec<String>>`en lugar de
`Vec<String>`. En la función `new`, creamos una nueva instancia 
`RefCell<Vec<String>>` alrededor del vector vacío.

Para la implementación del método `send`, el primer parámetro sigue siendo un
préstamo inmutable de `self`, que coincide con la definición del *trait*.
Llamamos `borrow_mut` en `RefCell<Vec<String>>` en `self.sent_messages` para
obtener una referencia mutable al valor dentro de `RefCell<Vec<String>>`,
que es el vector. Luego podemos llamar `push` en la referencia mutable al
vector para hacer un seguimiento de los mensajes enviados durante la prueba.

El último cambio que tenemos que hacer es la afirmación: para ver cuántos
elementos hay en el vector interno, llamamos `borrow` en
`RefCell<Vec<String>>`para obtener una referencia inmutable del vector.

Ahora que has visto cómo usar `RefCell<T>`, ¡profundizaremos en cómo funciona!

#### Mantener el seguimiento de los préstamos en tiempo de ejecución con `RefCell <T>`

Al crear referencias inmutables y mutables, usamos la sintaxis `&` y `&mut`,
respectivamente. Con `RefCell<T>`, usamos los métodos `borrow` y `borrow_mut`
que son parte de la API segura que pertenece a `RefCell<T>`. El método
`borrow` devuelve el tipo de puntero inteligente `Ref<T>`, y `borrow_mut`
devuelve el tipo de puntero inteligente `RefMut<T>`. Ambos tipos implementan
`Deref`, por lo que podemos tratarlos como referencias regulares.

El `RefCell<T>` realiza un seguimiento de cuántos punteros inteligentes
`Ref<T>` y `RefMut<T>` están actualmente activos. Cada vez que llamamos
`borrow`, `RefCell<T>` aumenta su cuenta de cuántos préstamos impagos están
activos. Cuando un valor `Ref<T>` sale del alcance, el recuento de préstamos
inmutables disminuye en uno. Al igual que las reglas de endeudamiento en
tiempo de compilación, `RefCell<T>` nos permite tener muchos préstamos
inmutables o un préstamo mutable en cualquier momento.

Si tratamos de violar estas reglas, en lugar de obtener un error de
compilación como lo haríamos con las referencias, la implementación de
`RefCell<T>` entrará en pánico en el tiempo de ejecución. El listado 15-23
muestra una modificación de la implementación de `send` en el listado 15-22.
Tratamos deliberadamente de crear dos préstamos mutables activos para el
mismo alcance para ilustrar que `RefCell<T>` nos impide hacer esto en tiempo
de ejecución.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
impl Messenger for MockMessenger {
    fn send(&self, message: &str) {
        let mut one_borrow = self.sent_messages.borrow_mut();
        let mut two_borrow = self.sent_messages.borrow_mut();

        one_borrow.push(String::from(message));
        two_borrow.push(String::from(message));
    }
}
```

<span class="caption">Listado 15-23: Creando dos referencias mutables en el
mismo ámbito para ver que `RefCell<T>` entrará en pánico</span>

Creamos una variable `one_borrow` para el puntero inteligente
`RefMut<T>` devuelto por `borrow_mut`. Luego creamos otro préstamo mutable de
la misma manera en la variable `two_borrow`. Esto hace que dos referencias
mutables en el mismo ámbito, que no está permitido. Cuando ejecutamos las
pruebas para nuestra biblioteca, el código en el listado 15-23 compilará sin
ningún error, pero la prueba fallará:

```text
---- tests::it_sends_an_over_75_percent_warning_message stdout ----
	thread 'tests::it_sends_an_over_75_percent_warning_message' panicked at
'already borrowed: BorrowMutError', src/libcore/result.rs:906:4
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

Tenga en cuenta que el código entró en pánico con el mensaje
`already borrowed: BorrowMutError`. Así es como `RefCell<T>` maneja las
violaciones de las reglas de endeudamiento en tiempo de ejecución.

Capturar errores de préstamo en tiempo de ejecución en lugar de compilar
significa que encontrará un error en su código más adelante en el proceso de
desarrollo y posiblemente no hasta que su código se implemente en la
producción. Además, su código incurrirá en una pequeña penalización de
rendimiento en el tiempo de ejecución como resultado de realizar un
seguimiento de los préstamos en tiempo de ejecución en lugar de tiempo de
compilación. Sin embargo, el uso de `RefCell<T>` hace posible escribir un
objeto simulado que puede modificarse para llevar un registro de los mensajes
que ha visto mientras lo está usando en un contexto donde solo se permiten
valores inmutables. Puede usar `RefCell<T>` a pesar de sus compensaciones
para obtener más funcionalidad de la que proporcionan las referencias
regulares.

### Tener múltiples propietarios de datos mutables mediante la combinación de `Rc<T>` y `RefCell<T>`

Una forma común de usar `RefCell<T>` es en combinación con `Rc<T>`. Recuerde
que `Rc<T>` te permite tener varios propietarios de algunos datos, pero solo
da acceso inmutable a esos datos. Si tiene un `Rc<T>` que contiene un
`RefCell <T>`, ¡puede obtener un valor que puede tener varios propietarios
*y* que puede mutar!

Por ejemplo, recuerde el ejemplo de la lista de contras en el listado 15-18
donde usamos `Rc<T>` para permitir que varias listas compartan la propiedad
de otra lista. Debido a que `Rc<T>` contiene solo valores inmutables, no
podemos cambiar ninguno de los valores en la lista una vez que los hemos
creado. Agreguemos `RefCell<T>` para obtener la capacidad de cambiar los
valores en las listas. El listado 15-24 muestra que al usar un `RefCell<T>`
en la definición `Cons`, podemos modificar el valor almacenado en todas las
listas:

<span class="filename">Filename: src/main.rs</span>

```rust
#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use List::{Cons, Nil};
use std::rc::Rc;
use std::cell::RefCell;

fn main() {
    let value = Rc::new(RefCell::new(5));

    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

    let b = Cons(Rc::new(RefCell::new(6)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(10)), Rc::clone(&a));

    *value.borrow_mut() += 10;

    println!("a after = {:?}", a);
    println!("b after = {:?}", b);
    println!("c after = {:?}", c);
}
```

<span class="caption">Listado 15-24: Usando `Rc<RefCell<i32>>` para crear
una `List` que podemos mutar</span>

Creamos un valor que es una instancia de `Rc<RefCell<i32>>` y lo almacenamos
en una variable llamada `value` para que podamos acceder a él directamente
más tarde. Luego creamos una `List` en `a` con una variante `Cons` que
contiene `value`. Necesitamos clonar `value` para que tanto `a` como `value`
tengan la propiedad del valor interno `5` en lugar de transferir la propiedad
de `value` a `a` o pedir prestado `a` borrow de `value`.

Concluimos la lista `a` en `Rc<T>` así que cuando creamos las listas `b`
y `c`, ambos pueden referirse a `a`, que es lo que hicimos en el Listado
15-18.

Después de que hemos creado las listas en `a`, `b`, y `c`, agregamos 10 al
valor en `value`. Hacemos esto llamando `borrow_mut` en `value`, que usa la
función de eliminación de referencias automática que discutimos en el
Capítulo 5 (consulte la sección "¿Dónde está el operador `->`?") Para
eliminar la referencia de `Rc<T>` al valor interno `RefCell<T>`. El método
`borrow_mut` devuelve un puntero inteligente `RefMut<T>`, y usamos el
operador de referencia en él y cambiamos el valor interno.

Cuando imprimimos `a`, `b`, y `c`, podemos ver que todos tienen el valor
modificado de 15 en lugar de 5:

```text
a after = Cons(RefCell { value: 15 }, Nil)
b after = Cons(RefCell { value: 6 }, Cons(RefCell { value: 15 }, Nil))
c after = Cons(RefCell { value: 10 }, Cons(RefCell { value: 15 }, Nil))
```

¡Esta técnica es bastante ordenada! al usar `RefCell<T>`, tenemos un valor
`List` inmutable externamente. Pero podemos usar los métodos en `RefCell<T>`
que proporcionan acceso a su mutabilidad interior para que podamos modificar
nuestros datos cuando sea necesario. Los controles de tiempo de ejecución de
las reglas de préstamo nos protegen de las *carreras de datos* (*data races*)
y a veces vale la pena cambiar un poco la velocidad por esta flexibilidad en
nuestras estructuras de datos.

La biblioteca estándar tiene otros tipos que proporcionan mutabilidad
interior, como `Cell<T>`, que es similar excepto que en lugar de dar
referencias al valor interno, el valor se copia dentro y fuera de `Cell<T>`.
También hay `Mutex<T>`, que ofrece mutabilidad interior que es seguro para
usar en varios *hilos* (*threads*); discutiremos su uso en el Capítulo 16.
Consulte los documentos de la biblioteca estándar para obtener más detalles
sobre las diferencias entre estos tipos.
