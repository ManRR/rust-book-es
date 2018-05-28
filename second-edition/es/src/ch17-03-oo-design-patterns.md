## Implementación de un patrón de diseño orientado a objetos

El *state pattern* es un patrón de diseño orientado a objetos. La clave del
patrón es que un valor tiene un estado interno, que se representa mediante un
conjunto de *state objects*, y el comportamiento del valor cambia en función
del interno estado. Los objetos de estado comparten funcionalidad: en Rust,
por supuesto, usamos estructuras y *traits* en lugar de objetos y herencia.
Cada objeto de estado es responsable de su propio comportamiento y de
gobernar cuando debería cambiar a otro Estado. El valor que contiene un
*state object* (*objeto de estado*) no sabe nada sobre comportamiento
diferente de los estados o cuándo hacer la transición entre estados.

Usar el patrón de estado significa que cuando cambien los requisitos
comerciales del programa, no necesitaremos cambiar el código del valor que
contiene el estado o el código que usa el valor. Solo necesitaremos
actualizar el código dentro de uno de los objetos de estado para cambiar sus
reglas o quizás agregar más objetos de estado. Veamos un ejemplo del patrón
de diseño del estado y cómo usarlo en Rust.

Implementaremos un flujo de trabajo de publicación de blog de forma
incremental. El final del blog la funcionalidad se verá así:

1. Una publicación de blog comienza como un borrador en blanco.
2. Cuando se finaliza el borrador, se solicita una revisión de la publicación.
3. Cuando se aprueba la publicación, se publica.
4. Solo las publicaciones de blog publicadas devuelven contenido para
 imprimir, por lo que las publicaciones no aprobadas no pueden
 accidentalmente ser publicadas.

Cualquier otro cambio que se intente en una publicación no debería tener
ningún efecto. Por ejemplo, si tratamos de aprobar un borrador de entrada de
blog antes de solicitar una revisión, la publicación debe seguir siendo un
borrador no publicado.

El listado 17-11 muestra este flujo de trabajo en forma de código: este es un
ejemplo de uso de la API que implementaremos en una caja de la biblioteca
denominada `blog`. Esto aún no se compilará porque aún no hemos implementado
el *crate* `blog`.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate blog;
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());

    post.request_review();
    assert_eq!("", post.content());

    post.approve();
    assert_eq!("I ate a salad for lunch today", post.content());
}
```

<span class="caption">Listado 17-11: Código que demuestra el comportamiento
deseado que queremos que tenga nuestro *crate* `blog`</span>

Queremos permitirle al usuario crear una nueva publicación borrador de blog
con `Post::new`. Entonces, queremos permitir que el texto se agregue a la
publicación del blog mientras está en el borrador estado. Si tratamos de
obtener el contenido de la publicación inmediatamente, antes de su aprobación,
no debería pasar nada porque la publicación sigue siendo un borrador. Hemos
agregado `assert_eq!` en el código con fines de demostración. Una excelente
prueba unitaria para esto sería afirmar que un borrador de una publicación de
blog devuelve un *string* vacía del método `content`, pero no vamos a
escribir pruebas para este ejemplo.

A continuación, queremos habilitar una solicitud de revisión de la
publicación, y queremos `content` para devolver un *string* vacía mientras
espera la revisión. Cuando la publicación recibe la aprobación, debe
publicarse, lo que significa que el texto de la publicación se devolverá
cuando se llame a `content`.

Tenga en cuenta que el único tipo con el que estamos interactuando desde el
*crate* es el `Post` tipo. Este tipo usará el patrón de estado y mantendrá un
valor que será uno de los tres objetos de estado que representan los diversos
estados en los que se puede publicar una publicación en borrador, en espera
de revisión o publicado. Cambiar de un estado a otro se gestionará
internamente dentro del tipo `Post`. Los estados cambian en respuesta a los
métodos llamados por los usuarios de nuestra biblioteca en la instancia
`Post`, pero no tienen que administrar los cambios de estado directamente.
Además, los usuarios no pueden cometer un error con los estados, como
publicar una publicación antes de que se revise.

### Definición de `Post` y Creación de una nueva instancia en el *borrador de estado* (*Draft State*)

¡Comencemos con la implementación de la biblioteca! Sabemos que necesitamos
una estructura pública de `Post` que contenga algo de contenido, así que
comenzaremos con la definición de la estructura y una función pública
asociada `new` para crear una instancia de `Post`, como se muestra en el
Listado 17-12 . También haremos un *trait* privado de `State`. Entonces
`Post` mantendrá un *trait object* de `Box<State>` dentro de un `Option` en
un campo privado llamado `state`. Verás por qué el `Option` es necesario en
un momento.

<span class="filename">Filename: src/lib.rs</span>

```rust
pub struct Post {
    state: Option<Box<State>>,
    content: String,
}

impl Post {
    pub fn new() -> Post {
        Post {
            state: Some(Box::new(Draft {})),
            content: String::new(),
        }
    }
}

trait State {}

struct Draft {}

impl State for Draft {}
```

<span class="caption">Listado 17-12: Definición de una estructura `Post` y
una función `new` que crea una nueva instancia `Post`, un *trait*
`State` y una estructura `Draft`</span>

El *trait* `State` define el comportamiento compartido por diferentes estados
de publicación, y los estados `Draft`, `PendingReview`, y `Published`
implementarán el atributo `State`. Por ahora, el *trait* no tiene ningún
método, y comenzaremos definiendo solo el estado `Draft` porque ese es el
estado en el que queremos que comience una publicación.

Cuando creamos un nuevo `Post`, establecemos su campo `state` en un valor
`Some` que contiene un `Box`. Este `Box` apunta a una nueva instancia de la
estructura `Draft`. Esto garantiza que cada vez que creamos una nueva
instancia de `Post`, comenzará como un borrador. Debido a que el campo
`state` de `Post` es privado, ¡no hay forma de crear un `Post` en ningún otro
estado! En la función `Post::new`, establecemos el campo `content` en un
nuevo `String` vacío.

### Almacenar el texto del contenido del *Post*

El listado 17-11 mostró que queremos poder llamar a un método llamado
`add_text` y pasarle un `&str` que luego se agrega al contenido de texto de
la publicación del blog. Implementamos esto como un método en lugar de
exponer el campo `content` como `pub`. Esto significa que podemos implementar
un método más adelante que controlará cómo se leen los datos del campo
`content`. El método `add_text` es bastante sencillo, así que agreguemos la
implementación en el Listado 17-13 al bloque `impl Post`:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub struct Post {
#     content: String,
# }
#
impl Post {
    // --snip--
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}
```

<span class="caption">Listado 17-13: Implementando el método `add_text` para
agregar texto al `content` de una publicación</span>

El método `add_text` toma una referencia mutable a `self`, porque estamos
cambiando la instancia `Post` a la que estamos llamando `add_text`. Luego
llamamos a `push_str` en `String` en `content` y pasamos el argumento `text`
para agregarlo al `content` guardado. Este comportamiento no depende del
estado en que se encuentra la publicación, por lo que no forma parte del
patrón de estado. El método `add_text` no interactúa con el campo `state`,
pero es parte del comportamiento que queremos admitir.

### Asegurar que el contenido de un *Draft Post* (*borrador de publicación*) esté vacío

Incluso después de haber llamado `add_text` y agregado algo de contenido a
nuestra publicación, aún queremos que el método `content` devuelva un
*string slice* vacío porque la publicación aún está en el estado borrador,
como se muestra en la línea 8 del Listado 17- 11. Por ahora, implementemos el
método `content` con la cosa más simple que cumplirá este requisito: devolver
siempre un *string slice* vacío. Cambiaremos esto más tarde una vez que
implementemos la capacidad de cambiar el estado de una publicación para que
se pueda publicar. Hasta ahora, las publicaciones solo pueden estar en el
estado borrador, por lo que el contenido de la publicación siempre debe estar
vacío. El listado 17-14 muestra esta implementación de marcador de posición:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub struct Post {
#     content: String,
# }
#
impl Post {
    // --snip--
    pub fn content(&self) -> &str {
        ""
    }
}
```

<span class="caption">Listado 17-14: Agregar una implementación de marcador
de posición para el método `content` en `Post` que siempre devuelve un
*string slice* vacío</span>

Con este método agregado de `content`, todo en el Listado 17-11 hasta la
línea 8 funciona según lo previsto.

### Solicitar una revisión del estado *Post Changes*

A continuación, debemos agregar funcionalidad para solicitar una revisión de
una publicación, que debería cambiar su estado de `Draft` a `PendingReview`.
El listado 17-15 muestra este código:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub struct Post {
#     state: Option<Box<State>>,
#     content: String,
# }
#
impl Post {
    // --snip--
    pub fn request_review(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.request_review())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<State>;
}

struct Draft {}

impl State for Draft {
    fn request_review(self: Box<Self>) -> Box<State> {
        Box::new(PendingReview {})
    }
}

struct PendingReview {}

impl State for PendingReview {
    fn request_review(self: Box<Self>) -> Box<State> {
        self
    }
}
```

<span class="caption">Listado 17-15: Implementando los métodos
`request_review` en `Post` y el *trait* `State`</span>

Le damos a `Post` un método público llamado `request_review` que tomará una
referencia mutable a `self`. Luego llamamos a un método interno
`request_review` en el estado actual de `Post`, y este segundo método
`request_review` consume el estado actual y devuelve un nuevo estado.

Agregamos el método `request_review` al *trait* `State`; todos los tipos que
implementan el *trait* ahora necesitarán implementar el método
`request_review`. Tenga en cuenta que en lugar de tener `self`,`&self`, o
`&mut self` como el primer parámetro del método, tenemos `self:Box<Self>`.
Esta sintaxis significa que el método solo es válido cuando se le llama a un
`Box` que contiene el tipo. Esta sintaxis toma posesión de `Box<Self>`,
invalidando el estado anterior para que el valor de estado del `Post` pueda
transformarse en un nuevo estado.

Para consumir el estado anterior, el método `request_review` necesita tomar
posesión del valor del estado. Aquí es donde entra el `Option` en el campo
`state` de `Post`: llamamos al método `take` para sacar el valor `Some` del
campo `state` y dejar `None` en su lugar , porque Rust no nos permite tener
campos despoblados en las estructuras. Esto nos permite mover el valor de
`state` fuera de `Post` en lugar de tomarlo prestado. Luego estableceremos el
valor de `state` de la publicación en el resultado de esta operación.

Necesitamos establecer `state` en `None` temporalmente en lugar de
establecerlo directamente con un código como
`self.state = self.state.request_review();` para obtener la propiedad del
valor `state`. Esto garantiza que `Post` no pueda usar el antiguo valor
`state` una vez que lo hayamos transformado en un nuevo estado.

El método `request_review` en `Draft` necesita devolver una nueva instancia
encuadrada de una nueva estructura `PendingReview`, que representa el estado
cuando una publicación está esperando una revisión. La estructura
`PendingReview` también implementa el método `request_review`, pero no
realiza transformaciones. Más bien, se devuelve a sí mismo, porque cuando
solicitamos una revisión en una publicación que ya se encuentra en el estado
`PendingReview`, debe permanecer en el estado `PendingReview`.

Ahora podemos comenzar a ver las ventajas del patrón de estado: el método
`request_review` en `Post` es el mismo sin importar su valor `state`. Cada
estado es responsable de sus propias reglas.

Dejaremos el método `content` en `Post` tal como está, devolviendo un
*string slice* vacío. Ahora podemos tener un `Post` en el estado
`PendingReview`, así como en el estado `Draft`, pero queremos el mismo
comportamiento en el estado `PendingReview`. ¡El listado 17-11 ahora funciona
hasta la línea 11!.

### Agregar el método `approve` que cambia el comportamiento del `content`

El método `approve` será similar al método `request_review`: establecerá
`state` en el valor que el estado actual dice que debería tener cuando se
aprueba ese estado, como se muestra en el Listado 17-16:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub struct Post {
#     state: Option<Box<State>>,
#     content: String,
# }
#
impl Post {
    // --snip--
    pub fn approve(&mut self) {
        if let Some(s) = self.state.take() {
            self.state = Some(s.approve())
        }
    }
}

trait State {
    fn request_review(self: Box<Self>) -> Box<State>;
    fn approve(self: Box<Self>) -> Box<State>;
}

struct Draft {}

impl State for Draft {
#     fn request_review(self: Box<Self>) -> Box<State> {
#         Box::new(PendingReview {})
#     }
#
    // --snip--
    fn approve(self: Box<Self>) -> Box<State> {
        self
    }
}

struct PendingReview {}

impl State for PendingReview {
#     fn request_review(self: Box<Self>) -> Box<State> {
#         self
#     }
#
    // --snip--
    fn approve(self: Box<Self>) -> Box<State> {
        Box::new(Published {})
    }
}

struct Published {}

impl State for Published {
    fn request_review(self: Box<Self>) -> Box<State> {
        self
    }

    fn approve(self: Box<Self>) -> Box<State> {
        self
    }
}
```

<span class="caption">Listado 17-16: Implementando el método `approve` en
`Post` y el *trait* `State`</span>

Agregamos el método `approve` al *trait* `State` y agregamos una nueva
estructura que implementa `State`, el estado `Published`.

Similar a `request_review`, si llamamos al método `approve` en un `Draft`, no
tendrá ningún efecto porque devolverá `self`. Cuando llamamos a `approve` on
`PendingReview`, devuelve una nueva instancia encuadrada de la estructura
`Published`. La estructura `Publicada` implementa el *trait* `State`, y tanto
para el método `request_review` como para el método `approve`, se devuelve a
sí mismo, porque la publicación debe permanecer en el estado `Published` en
esos casos.

Ahora necesitamos actualizar el método `content` en `Post`: si el estado es
`Published`, queremos devolver el valor en el campo `content` de la
publicación; de lo contrario, queremos devolver un *string slice* vacío, como
se muestra en el Listado 17-17:

<span class="filename">Filename: src/lib.rs</span>

```rust
# trait State {
#     fn content<'a>(&self, post: &'a Post) -> &'a str;
# }
# pub struct Post {
#     state: Option<Box<State>>,
#     content: String,
# }
#
impl Post {
    // --snip--
    pub fn content(&self) -> &str {
        self.state.as_ref().unwrap().content(&self)
    }
    // --snip--
}
```

<span class="caption">Listado 17-17: Actualización del método `content` en
`Post` para delegar en un método `content` en `State`</span>

Porque el objetivo es mantener todas estas reglas dentro de las estructuras
que implementan `State`, llamamos a un método `content` sobre el valor en
`state` y pasamos la publicación instancia (es decir, `self`) como argumento.
Luego devolvemos el valor que es regresó de usar el método `content` en el
valor `state`.

Llamamos al método `as_ref` en la `Option` porque queremos una referencia al
valor dentro de la `Option` en lugar de la propiedad del valor. Porque
`state` es un `Option<Box<State>>`, cuando llamamos `as_ref`, un
`Option<&Box<State>>` es devuelto. Si no llamamos a `as_ref`, obtendríamos un
error porque no podemos mueve `state` fuera del `&self` prestado del
parámetro function.

Luego llamamos al método `unwrap`, que sabemos que nunca entrará en pánico,
porque conocer los métodos en 'Post' asegurar que `state` siempre contenga un
`Some` valor cuando esos métodos están hechos. Este es uno de los casos de
los que hablamos en la sección “Casos cuando tiene más información que el
compilador” del Capítulo 9 cuando sabemos que un valor `None` nunca es
posible, aunque el compilador no es capaz de entender eso.

En este punto, cuando llamemos a `content` en `&Box<State>`, la coerción
*deref* tendrá efecto en el `&` y en el `Box` por lo que el método `content`
será en última instancia llamado en el tipo que implementa el *trait*
`State`. Eso significa que tenemos que agregar `content` a la definición de
*trait* `State`, y ahí es donde pondremos la lógica de qué contenido devolver
dependiendo de qué estado tenemos, como se muestra en el Listado 17-18:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub struct Post {
#     content: String
# }
trait State {
    // --snip--
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        ""
    }
}

// --snip--
struct Published {}

impl State for Published {
    // --snip--
    fn content<'a>(&self, post: &'a Post) -> &'a str {
        &post.content
    }
}
```

<span class="caption">Listado 17-18: Agregar el método `content` al *trait*
`State`</span>

Agregamos una implementación predeterminada para el método `content` que
devuelve un *string slice* vacío. Eso significa que no necesitamos
implementar `content` en las estructuras `Draft` y `PendingReview`. La
estructura `Published` anulará el método `content` y devolverá el valor en
`post.content`.

Tenga en cuenta que necesitamos anotaciones de *lifetime* en este método,
como discutimos en el Capítulo 10. Estamos tomando como referencia un
`post` como argumento y devolviendo una referencia a parte de ese
`post`, por lo que la vida de la referencia devuelta está relacionado con el
*lifetime* del argumento `post`.

Y hemos terminado, ¡todo el Listado 17-11 ahora funciona!. Implementamos el
patrón de estado con las reglas del flujo de trabajo de publicación de blog.
La lógica relacionada con las reglas vive en los *objetos de estado*
(*state objects*) en lugar de dispersarse en `Post`.

### *Trade-offs* del patrón de estado

Hemos demostrado que Rust es capaz de implementar el patrón de estado
orientado a objetos para encapsular los diferentes tipos de comportamiento
que una publicación debe tener en cada estado. Los métodos en `Post` no saben
nada sobre los diversos comportamientos. La forma en que organizamos el
código, tenemos que buscar en un solo lugar para conocer las diferentes
formas en que se puede comportar una publicación publicada: la implementación
del *trait* `State` en la estructura `Published`.

Si tuviéramos que crear una implementación alternativa que no usara el patrón
de estado, podríamos usar expresiones `match` en los métodos en `Post` o
incluso en el código `main` que verifica el estado de la publicación y cambia
el comportamiento en esos lugares. ¡Eso significaría que tendríamos que
buscar en varios lugares para comprender todas las implicaciones de una
publicación en el estado publicado! Esto solo aumentaría a más estados que
agregamos: cada una de esas expresiones `match` necesitaría otro brazo.

Con el patrón de estado, los métodos `Post` y los lugares que usamos `Post`
no necesitan expresiones `match`, y para agregar un nuevo estado, solo
necesitaríamos agregar una nueva estructura e implementar los métodos del
*trait* en esa estructura.

La implementación que usa el patrón de estado es fácil de ampliar para
agregar más funcionalidad. Para ver la simplicidad de mantener el código que
usa el patrón de estado, pruebe algunas de estas sugerencias:

* Agregue un método `reject` que cambie el estado de la publicación de
 `PendingReview` de nuevo a `Draft`.
* Requiere dos llamadas para `approve` antes de que el estado pueda cambiarse
 a `Published`.
* Permitir a los usuarios agregar contenido de texto solo cuando una
 publicación está en el estado `Draft`. Sugerencia: haga que el objeto de
 estado sea responsable de lo que podría cambiar sobre el contenido, pero no
 será responsable de modificar el `Post`.

Una desventaja del patrón de estado es que, debido a que los estados
implementan las transiciones entre estados, algunos de los estados están
acoplados entre sí. Si agregamos otro estado entre `PendingReview` y
`Published`, como `Scheduled`, tendríamos que cambiar el código en
`PendingReview` para pasar a `Scheduled` en su lugar. Sería menos trabajo si
`PendingReview` no necesitara cambiarse con la adición de un nuevo estado,
pero eso significaría cambiar a otro patrón de diseño.

Otro inconveniente es que hemos duplicado algo de lógica. Para eliminar
parte de la duplicación, podemos intentar realizar implementaciones
predeterminadas para los métodos `request_review` y `approve` en el *trait*
`State` que devuelve `self`; sin embargo, esto violaría la seguridad de los
objetos, porque el *trait* no sabe cuál será exactamente el `self` concreto. Queremos poder utilizar `State` como un *trait object*, por lo que necesitamos que sus métodos sean seguros para los objetos.

Otra duplicación incluye las implementaciones similares de los métodos
`request_review` y `approve` en `Post`. Ambos métodos delegan a la
implementación del mismo método en el valor en el campo `state` de `Option`
y establecen el nuevo valor del campo `state` en el resultado. Si tuviéramos
muchos métodos en `Post` que siguieran este patrón, podríamos considerar la
definición de una macro para eliminar la repetición (ver el Apéndice D para
más información sobre macros).

Al implementar el patrón de estado exactamente como está definido para los
lenguajes orientados a objetos, no estamos aprovechando al máximo las
fortalezas de Rust como podríamos. Veamos algunos cambios que podemos hacer
en el *crate* `blog` que puede convertir los estados inválidos y las
transiciones en errores de tiempo de compilación.

#### *Codificación de estados* (*Encoding States*) y comportamiento como tipos

Le mostraremos cómo repensar el patrón de estado para obtener un conjunto
diferente de concesiones. En lugar de encapsular completamente los estados y
las transiciones, por lo que el código externo no los conoce, codificaremos
los estados en diferentes tipos. En consecuencia, el sistema de comprobación
de tipos de Rust evitará los intentos de utilizar borradores de
publicaciones donde solo se permiten publicaciones publicadas al emitir un
error de compilación.

Consideremos la primera parte de `main` en el Listado 17-11:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");
    assert_eq!("", post.content());
}
```

Todavía habilitamos la creación de nuevas publicaciones en el estado del
borrador utilizando `Post::new` y la posibilidad de agregar texto al
contenido de la publicación. Pero en lugar de tener un método `content` en
una publicación preliminar que devuelve un *string* vacío, lo haremos para
que las publicaciones preliminares no tengan el método `content` en
absoluto. De esta forma, si tratamos de obtener el contenido de un borrador,
obtendremos un error de compilación que nos indicará que el método no
existe. Como resultado, nos será imposible mostrar accidentalmente el
contenido del borrador de la publicación en producción, porque ese código ni
siquiera se compilará. El listado 17-19 muestra la definición de una
estructura `Post` y una estructura `DraftPost`, así como los métodos en cada
uno:

<span class="filename">Filename: src/lib.rs</span>

```rust
pub struct Post {
    content: String,
}

pub struct DraftPost {
    content: String,
}

impl Post {
    pub fn new() -> DraftPost {
        DraftPost {
            content: String::new(),
        }
    }

    pub fn content(&self) -> &str {
        &self.content
    }
}

impl DraftPost {
    pub fn add_text(&mut self, text: &str) {
        self.content.push_str(text);
    }
}
```

<span class="caption">Listado 17-19: Un `Post` con un método `content` y un
`DraftPost` sin un método `content`</span>

Las estructuras `Post` y `DraftPost` tienen un campo `content` privado que
almacena el texto de la entrada del blog. Las estructuras ya no tienen el
campo `state` porque estamos moviendo la codificación del estado a los tipos
de las estructuras. La estructura `Post` representará una publicación
publicada, y tiene un método `content` que devuelve el `content`.

Todavía tenemos una función `Post::new`, pero en lugar de devolver una
instancia de` Post`, devuelve una instancia de `DraftPost`. Como `content`
es privado y no hay funciones que devuelvan `Post`, no es posible crear una
instancia de `Post` en este momento.

La estructura `DraftPost` tiene un método `add_text`, por lo que podemos
agregar texto a `content` como antes, pero tenga en cuenta que `DraftPost`
no tiene definido un método `content`. Entonces, ahora el programa asegura
que todas las publicaciones comiencen como borradores de publicaciones, y
las publicaciones preliminares no tienen su contenido disponible para
mostrar. Cualquier intento de evitar estas restricciones dará como resultado
un error de compilación.

#### Implementando transiciones como transformaciones en diferentes tipos

Entonces, ¿cómo conseguimos una publicación publicada?. Queremos hacer
cumplir la regla de que un borrador debe ser revisado y aprobado antes de
que pueda publicarse. Una publicación en el estado de revisión pendiente aún
no debe mostrar ningún contenido. Implementemos estas restricciones
agregando otra estructura, `PendingReviewPost`, definiendo el método
`request_review` en `DraftPost` para devolver `PendingReviewPost`, y
definiendo un método `approve` en `PendingReviewPost` para devolver `Post`,
como se muestra en el listado 17-20:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub struct Post {
#     content: String,
# }
#
# pub struct DraftPost {
#     content: String,
# }
#
impl DraftPost {
    // --snip--

    pub fn request_review(self) -> PendingReviewPost {
        PendingReviewPost {
            content: self.content,
        }
    }
}

pub struct PendingReviewPost {
    content: String,
}

impl PendingReviewPost {
    pub fn approve(self) -> Post {
        Post {
            content: self.content,
        }
    }
}
```

<span class="caption">Listado 17-20: Un `PendingReviewPost` que se crea al
invocar `request_review` en `DraftPost` y un método `approve` que convierte
un `PendingReviewPost` en un `Post` publicado</span>

Los métodos `request_review` y `approve` toman la propiedad de `self`,
consumiendo así las instancias `DraftPost` y `PendingReviewPost` y
transformándolas en `PendingReviewPost` y en `Post` publicado,
respectivamente. De esta manera, no tendremos ninguna instancia `DraftPost`
persistente después de haber llamado `request_review` en ellos, y así
sucesivamente. La estructura `PendingReviewPost` no tiene un método
`content` definido, por lo que intentar leer su contenido genera un error de
compilación, como con `DraftPost`. Porque la única forma de obtener una
instancia de publicación publicada que tenga un método de `content` definido
es llamar al método `approve` en un `PendingReviewPost`, y la única forma de
obtener un `PendingReviewPost` es llamar al método `request_review` en un
`DraftPost`, ahora hemos codificado el flujo de trabajo de la publicación de
blog en el sistema de tipos.

Pero también tenemos que hacer algunos pequeños cambios en `main`. Los
métodos `request_review` y `approve` devuelven nuevas instancias en lugar de
modificar la estructura a la que están llamadas, por lo que necesitamos
agregar más asignaciones *shadowing* `let post =` para guardar las instancias
devueltas. Tampoco podemos tener las afirmaciones sobre el borrador y los
contenidos pendientes de revisión del mensaje como cadenas vacías, ni las
necesitamos: no podemos compilar código que intente usar el contenido de las
publicaciones en esos estados por más tiempo. El código actualizado en
`main` se muestra en el Listado 17-21:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
extern crate blog;
use blog::Post;

fn main() {
    let mut post = Post::new();

    post.add_text("I ate a salad for lunch today");

    let post = post.request_review();

    let post = post.approve();

    assert_eq!("I ate a salad for lunch today", post.content());
}
```

<span class="caption">Listado 17-21: Modificaciones a `main` para usar la
nueva implementación del flujo de trabajo de blog</span>

Los cambios que necesitamos hacer a `main` para reasignar `post` significan
que esta implementación ya no sigue el patrón de estado orientado a objetos:
las transformaciones entre los estados ya no están encapsuladas por completo
dentro de la implementación `Post`. Sin embargo, nuestra ganancia es que los
estados inválidos ahora son imposibles debido al sistema de tipos y la
verificación de tipos que ocurre en tiempo de compilación. Esto garantiza
que se descubrirán ciertos errores, como la visualización del contenido de
una publicación no publicada, antes de que lleguen a la producción.

Pruebe las tareas sugeridas para los requisitos adicionales que mencionamos
al comienzo de esta sección en el *crate* `blog`, ya que es posterior al
Listado 17-20 para ver qué piensa sobre el diseño de esta versión del
código. Tenga en cuenta que algunas de las tareas pueden completarse ya en
este diseño.

Hemos visto que, aunque Rust es capaz de implementar patrones de diseño
orientados a objetos, otros patrones, como el estado de codificación en el
sistema de tipos, también están disponibles en Rust. Estos patrones tienen
diferentes intercambios. Aunque es posible que esté muy familiarizado con
los patrones orientados a objetos, repensar el problema para aprovechar las
características de Rust puede proporcionar beneficios, como la prevención de
algunos errores en tiempo de compilación. Los patrones orientados a objetos
no siempre serán la mejor solución en Rust debido a ciertas características,
como la propiedad, que los lenguajes orientados a objetos no tienen.

## Resumen

No importa si cree que Rust es un lenguaje orientado a objetos después de
leer este capítulo, ahora sabe que puede usar *trait objects* para obtener
algunas características orientadas a objetos en Rust. El *dynamic dispatch*
(*despacho dinámico*) puede darle flexibilidad a su código a cambio de un
poco de rendimiento en el tiempo de ejecución. Puede utilizar esta
flexibilidad para implementar patrones orientados a objetos que pueden
ayudar al mantenimiento de su código. Rust también tiene otras
características, como la propiedad, que los lenguajes orientados a objetos
no tienen. Un patrón orientado a objetos no siempre será la mejor manera de
aprovechar las fortalezas de Rust, pero es una opción disponible.

A continuación, veremos patrones, que son otra de las características de
Rust que permiten mucha flexibilidad. Los hemos visto brevemente a lo largo
del libro, pero todavía no hemos visto su capacidad completa. ¡Vamonos!
