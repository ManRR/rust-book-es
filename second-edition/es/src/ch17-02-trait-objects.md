## Usar *Trait Objects* que permiten valores de diferentes tipos

En el Capítulo 8, mencionamos que una limitación de los vectores es que pueden
almacenar elementos de un solo tipo. Creamos una solución en el Listado 8-10
donde definimos una enumeración `SpreadsheetCell` que tenía variantes para
contener enteros, flotantes, y texto Esto significaba que podíamos almacenar
diferentes tipos de datos en cada celda y todavía tiene un vector que
representa una fila de celdas. Esto es perfectamente buena solución cuando
nuestros artículos intercambiables son un conjunto fijo de tipos que conocemos
cuando nuestro código es compilado

Sin embargo, a veces queremos que nuestro usuario de la biblioteca pueda
ampliar el conjunto de tipos que son válidos en una situación particular.
Para mostrar cómo podemos lograr esto, crearemos una herramienta de ejemplo
de interfaz gráfica de usuario (GUI) que itera a través de una lista de
artículos, llamando a un método de 'dibujar' en cada uno para dibujarlo en el
pantalla: una técnica común para herramientas GUI. Crearemos un *library crate* llamada `gui` que contiene la estructura de una biblioteca GUI. Este
*crate* puede incluir algunos tipos que las personas pueden usar, como `Button` o `TextField`. En adición, los usuarios de `gui` querrán crear sus
propios tipos que puedan dibujarse: para ejemplo, un programador podría
agregar una `Imagen` y otro podría agregar un `SelectBox`.

No implementaremos una biblioteca de GUI completamente desarrollada para este
ejemplo, pero mostraremos cómo las piezas encajarían juntas en el momento de
escribir la biblioteca, no podemos conocer y definir todos los tipos que
otros programadores pueden querer crear. Pero sí sabemos que `gui` necesita
hacer un seguimiento de muchos valores de diferentes tipos, y necesita llamar
a un método `draw` en cada uno de estos valores de tipos diferentes. No es
necesario saber exactamente qué sucederá cuando llamemos al método `draw`,
solo que el valor tendrá ese método disponible para llamar.

Para hacer esto en un lenguaje con herencia, podríamos definir una clase
llamada `Component` que tiene un método llamado `draw` en él. Las otras
clases, como `Button`, `Image`, y `SelectBox`, heredarían de `Component` y
por lo tanto heredar el método `draw`. Cada uno podría anular el método
`draw` para definir su comportamiento personalizado, pero el framework podría
tratar todos los tipos como si fueran instancias `Component` y llamar `draw`
sobre ellos. Pero como Rust no tiene herencia, necesitamos otra forma de
estructurar la biblioteca `gui` para permitir a los usuarios ampliarla con
nuevos tipos.

### Definición de un *trait* para el comportamiento común

Para implementar el comportamiento que queremos que tenga `gui`, definiremos
un *trait* llamado `Draw` que tendrá un método llamado `draw`. Entonces
podemos definir un vector que toma un *trait object*. Un *trait object*
apunta a una instancia de un tipo que implementa el *trait* que especificamos.
Creamos un *trait object* especificando algunos tipo de puntero, como una
referencia `&` o un puntero inteligente `Box<T>`, y luego
especificando el *trait* relevante. (Hablaremos sobre la razón por la cual
los objetos de *trait* deben usar un puntero en el Capítulo 19 en la sección
“Dynamically Sized Types & Sized”).
Podemos usar *trait object* en lugar de un tipo genérico o concreto. Donde
sea que nosotros usar un *trait object*, el sistema de tipos de Rust
asegurará en tiempo de compilación que cualquier el valor utilizado en ese
contexto implementará el *trait* del *trait object*.
En consecuencia, no necesitamos saber todos los tipos posibles en tiempo de
compilación.

Hemos mencionado que en Rust, nos abstenemos de llamar a estructuras y
enumeraciones “objects” para distinguirlos de los objetos de otros lenguajes.
En una estructura o enum, los datos en los campos struct y el comportamiento
en bloques `impl` son separados, mientras que en otros lenguajes, los datos y
el comportamiento combinados en un concepto a menudo se etiquetan como un
objeto. Sin embargo, los *trait object* *son* más como objetos en otros
lenguajes en el sentido de que combinan datos y comportamiento.
Pero los *trait object* difieren de los objetos tradicionales en que no
podemos agregar datos a un *trait object*. Los *trait object* no son tan
útiles en general como los objetos en otros lenguajes: su propósito
específico es permitir la abstracción a través del comportamiento común.

El listado 17-3 muestra cómo definir un *trait* llamado `Draw` con un método
llamado `draw`:

<span class="filename">Filename: src/lib.rs</span>

```rust
pub trait Draw {
    fn draw(&self);
}
```

<span class="caption">Listado 17-3: Definición del *trait* `Draw`</span>

Esta sintaxis debería ser familiar a partir de nuestras discusiones sobre
cómo definir *trait* en el Capítulo 10. Luego viene una nueva sintaxis: el
Listado 17-4 define una estructura llamada `Screen` que contiene un vector
llamado `components`. Este vector es del tipo `Box<Draw>`, que es un
*trait object*; es un sustituto para cualquier tipo dentro de un `Box` que implementa el *trait* `Draw`.

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub trait Draw {
#     fn draw(&self);
# }
#
pub struct Screen {
    pub components: Vec<Box<Draw>>,
}
```

<span class="caption">Listado 17-4: Definición de la estructura `Screen` con
un campo `components` que contiene un vector de *trait objects* que
implementan el *trait* `Draw`</span>

En la estructura `Screen`, definiremos un método llamado `run` que llamará al
método `draw` en cada uno de sus `components`, como se muestra en el Listado
17-5:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub trait Draw {
#     fn draw(&self);
# }
#
# pub struct Screen {
#     pub components: Vec<Box<Draw>>,
# }
#
impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

<span class="caption">Listado 17-5: Un método `run` en `Screen` que llama al
método `draw` en cada componente</span>

Esto funciona de manera diferente a la definición de una estructura que usa
un parámetro de tipo genérico con *trait bounds*. Un parámetro de tipo
genérico solo puede ser sustituido por un tipo concreto a la vez, mientras
que los *trait objects* permiten que múltiples tipos concretos rellenen el
*trait objects* en el tiempo de ejecución. Por ejemplo, podríamos haber
definido la estructura `Screen` usando un tipo genérico y un *trait bounds*
como en el Listado 17-6:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub trait Draw {
#     fn draw(&self);
# }
#
pub struct Screen<T: Draw> {
    pub components: Vec<T>,
}

impl<T> Screen<T>
    where T: Draw {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}
```

<span class="caption">Listado 17-6: Una implementación alternativa de la
estructura `Screen` y su método `run` usando genéricos y
*trait bounds*</span>

Esto nos restringe a una instancia de `Screen` que tiene una lista de
componentes, todos de tipo `Button` o todos de tipo `TextField`. Si solo
tiene colecciones homogéneas, es preferible usar genéricos y *trait bounds*
porque las definiciones se monomorfizarán en el momento de la compilación
para usar los tipos concretos.

Por otro lado, con el método que usa *trait objects*, una instancia de
`Screen` puede contener un `Vec<T>` que contiene un `Box<Botón>` así como un
`Box<TextField>`. Veamos cómo funciona esto, y luego hablaremos sobre las
implicaciones de rendimiento en el tiempo de ejecución.

### Implementando el *Trait*

Ahora agregaremos algunos tipos que implementan el *trait* `Draw`.
Proporcionaremos el tipo `Button`. Nuevamente, la implementación de una
biblioteca GUI está más allá del alcance de este libro, por lo que el método
`draw` no tendrá ninguna implementación útil en su cuerpo. Para imaginar cómo
se vería la implementación, una estructura `Button` podría tener campos para
`width`, `height` y `label`, como se muestra en el Listado 17-7:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub trait Draw {
#     fn draw(&self);
# }
#
pub struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // code to actually draw a button
    }
}
```

<span class="caption">Listado 17-7: Una estructura `Button` que implementa el
*trait* `Draw`</span>

Los campos `width`, `height`, y `label` en `Button` diferirán de los campos
en otros componentes, como el tipo `TextField`, que podría tener esos campos
más un campo `placeholder` en su lugar. Cada uno de los tipos que queremos
dibujar en la pantalla implementará el *trait* `Draw` pero usará un código
diferente en el método `draw` para definir cómo dibujar ese tipo particular,
como `Button` tiene aquí (sin el código GUI real) , que está más allá del
alcance de este capítulo). El tipo `Button`, por ejemplo, podría tener un
bloque `impl` adicional que contenga métodos relacionados con lo que sucede
cuando un usuario hace clic en el botón. Este tipo de métodos no se aplicará
a tipos como `TextField`.

Si alguien que utiliza nuestra biblioteca decide implementar una estructura
`SelectBox` que tenga los campos `width`, `height`, y `options`, implementará
también el *trait* `Draw` en el tipo `SelectBox`, como se muestra en el
Listado 17 -8:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate gui;
use gui::Draw;

struct SelectBox {
    width: u32,
    height: u32,
    options: Vec<String>,
}

impl Draw for SelectBox {
    fn draw(&self) {
        // code to actually draw a select box
    }
}
```

<span class="caption">Listado 17-8: Otro *crate* usando `gui` e implementando
el *trait* `Draw` en una estructura `SelectBox`</span>

El usuario de nuestra biblioteca ahora puede escribir su función `main` para
crear una instancia `Screen`. Para la instancia de `Screen`, pueden agregar
un `SelectBox` y un `Button` poniendo cada uno en `Box<T>` para convertirse
en un *trait object*. Luego pueden llamar al método `run` en la instancia
`Screen`, que llamará `draw` en cada uno de los componentes. El listado 17-9 muestra esta implementación:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
use gui::{Screen, Button};

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(SelectBox {
                width: 75,
                height: 10,
                options: vec![
                    String::from("Yes"),
                    String::from("Maybe"),
                    String::from("No")
                ],
            }),
            Box::new(Button {
                width: 50,
                height: 10,
                label: String::from("OK"),
            }),
        ],
    };

    screen.run();
}
```

<span class="caption">Listado 17-9: Usar *trait objects* para almacenar
valores de diferentes tipos que implementan el mismo *trait*</span>

Cuando escribimos la biblioteca, no sabíamos que alguien podría agregar la
`SelectBox`, pero nuestra implementación `Screen` fue capaz de operar en
nuevo tipo y dibujarlo porque `SelectBox` implementa el tipo `Draw`, que
significa que implementa el método de `draw`.

Este concepto-de preocuparse solo por los mensajes a los que responde un valor
en lugar del tipo concreto del valor, es similar al concepto *duck typing*
(*tipaje de pato*) en lenguajes tipados dinámicamente: si camina como un pato
y grazna como un pato, ¡entonces debe ser un pato! En la implementación de
`run` en `Screen` en Listado 17-5, `run` no necesita saber cuál es el tipo
concreto de cada componente. No verifica si un componente es una instancia de
un `Button` o una `SelectBox`, simplemente llama al método `draw` en el 
componente. Al especificar
`Box<Draw>` como el tipo de los valores en el vector `components`, hemos
definido `Screen` necesita valores a los que podamos llamar el método `draw`.

La ventaja de usar *trait objects* y el sistema de tipos de Rust para
escribir código similar al código que usa el tipado de pato es que nunca
tenemos que verificar si valor implementa un método particular en tiempo de
ejecución o se preocupa por obtener errores si un valor no implementa un
método pero lo llamamos de todos modos. Rust no compilará nuestro código si
los valores no implementan los *trait* que los *trait objects* necesitan.

Por ejemplo, el Listado 17-10 muestra lo que sucede si tratamos de crear una
`Screen` con un `String` como componente:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate gui;
use gui::Screen;

fn main() {
    let screen = Screen {
        components: vec![
            Box::new(String::from("Hi")),
        ],
    };

    screen.run();
}
```

<span class="caption">Listado 17-10: Intentar usar un tipo que no implementa
el *trait* del *trait objects*</span>

Obtendremos este error porque `String` no implementa el *trait* `Draw`:

```text
error[E0277]: the trait bound `std::string::String: gui::Draw` is not satisfied
  --> src/main.rs:7:13
   |
 7 |             Box::new(String::from("Hi")),
   |             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the trait gui::Draw is not
   implemented for `std::string::String`
   |
   = note: required for the cast to the object type `gui::Draw`
```

Este error nos permite saber que estamos pasando algo a `Screen` que no
queríamos pasar y que deberíamos pasar un tipo diferente o debemos
implementar `Draw` en `String` para que `Screen` pueda invocar `draw` en él.

### Trait Objects Perform Dynamic Dispatch

Recordemos en la sección “Rendimiento del código usando genéricos” en el
Capítulo 10 nuestra discusión sobre el proceso de monomorfización realizado
por el compilador cuando utilizamos trait bounds en genéricos: el compilador
genera implementaciones no genéricas de funciones y métodos para cada tipo
concreto que usamos en el lugar de un parámetro de tipo genérico. El código que resulta de la monomorfización está haciendo *dispatch estático*, que es
cuando el compilador sabe a qué método está llamando en tiempo de
compilación. Esto se opone a *dispatch dinámico*, que es cuando el compilador
no puede decir en tiempo de compilación a qué método está llamando. En los
casos de envío dinámico, el compilador emite un código que en el tiempo de
ejecución determinará qué método llamar.

Cuando utilizamos *trait objects*, Rust debe usar el despacho dinámico. El
compilador no conoce todos los tipos que podrían usarse con el código que usa
*trait objects*, por lo que no sabe qué método se implementó en qué tipo
llamar. En cambio, en tiempo de ejecución, Rust utiliza los punteros dentro
del *trait objects* para saber qué método llamar. Hay un costo en tiempo de
ejecución cuando ocurre esta búsqueda que no ocurre con el envío estático. El
envío dinámico también evita que el compilador elija alinear el código de un
método, lo que a su vez impide algunas optimizaciones. Sin embargo, obtuvimos
flexibilidad adicional en el código que escribimos en el listado 17-5 y
pudimos apoyarlo en el listado 17-9, por lo que es una contrapartida a
considerar.

### Se requiere seguridad de objetos para *Trait Objects*

Solo puede hacer *trait* *object-safe* en *trait objects*. Algunas reglas
complejas gobiernan todas las propiedades que hacen que un *trait objects*
sea seguro, pero en la práctica, solo dos reglas son relevantes. Un *trait*
es seguro para los objetos si todos los métodos definidos en el *trait*
tienen las siguientes propiedades:

* El tipo de devolución no es `Self`.
* No hay parámetros genéricos de tipo.

La palabra clave `Self` es un alias para el tipo en el que estamos
implementando los *trait* o métodos. Los *trait objects* deben ser seguros
para los objetos porque una vez que haya usado un *trait objects*, Rust ya no
sabe qué tipo concreto está implementando ese *trait*. Si un método de
*trait* devuelve el tipo concreto propio, pero un *trait objects* olvida
el tipo exacto que es el, no hay forma de que el método pueda
usar el tipo concreto original. Lo mismo es cierto para los parámetros de
tipo genérico que se rellenan con parámetros de tipo concreto cuando se usa
el *trait*: los tipos concreto se vuelven parte del tipo que implementa el
*trait*. Cuando el tipo se olvida mediante el uso de un *trait objects*, no
hay forma de saber qué tipos rellenar en el parámetros de tipo genérico con.

Un ejemplo de un *trait* cuyos métodos no son seguros para objetos es el
*trait* `Clone` de la biblioteca estándar. La firma del método `clone` en el
*trait* `Clone` se ve así:

```rust
pub trait Clone {
    fn clone(&self) -> Self;
}
```

El tipo `String` implementa el *trait* `Clone`, y cuando llamamos al método
`clone` en una instancia de `String` recuperamos una instancia de `String`.
De manera similar, si llamamos `clone` a una instancia de `Vec<T>`, obtenemos
una instancia de `Vec<T>`. La firma de `clone` necesita saber qué tipo
representará para `Self`, porque ese es el tipo de devolución.

El compilador indicará cuando intenta hacer algo que viola las reglas de
seguridad de objetos con respecto a los *trait objects*. Por ejemplo,
supongamos que intentamos implementar la estructura `Screen` en el Listado
17-4 para mantener los tipos que implementan el *trait* `Clonar` en lugar del
*trait* `Draw`, como este:

```rust,ignore
pub struct Screen {
    pub components: Vec<Box<Clone>>,
}
```

Obtendríamos este error:

```text
error[E0038]: the trait `std::clone::Clone` cannot be made into an object
 --> src/lib.rs:2:5
  |
2 |     pub components: Vec<Box<Clone>>,
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ the trait `std::clone::Clone` cannot be
made into an object
  |
  = note: the trait cannot require that `Self : Sized`
```

Este error significa que no puede usar este *trait* como un *trait object* de
esta manera. Si está interesado en más detalles sobre la seguridad de los
objetos, consulte [Rust RFC 255].

[Rust RFC 255]: https://github.com/rust-lang/rfcs/blob/master/text/0255-object-safety.md