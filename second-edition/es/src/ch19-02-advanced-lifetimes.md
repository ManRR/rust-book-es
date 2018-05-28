## *Lifetimes* avanzado

En el Capítulo 10 de la sección “Validación de referencias con Lifetimes*”,
aprendió a anotar referencias con parámetros de *lifetime* para decirle a
Rust cómo se relacionan los *lifetimes* de diferentes referencias. Viste cómo
cada referencia tiene un *lifetime*, pero la mayoría de las veces, Rust te
dejará elidir *lifetimes*. Ahora veremos tres características avanzadas de *lifetimes* que aún no hemos cubierto:

* Subtipo de *lifetime*: asegura que una vida dura más que otra vida
* Límites de *lifetime*: especifica una duración para una referencia a un
 tipo genérico
* Inferencia de *lifetimes* de los *trait object*: permite al compilador
 inferir el *lifetime* de los *trait object y cuando necesitan ser
  especificados

### Garantizar un *Lifetime* sobrevivir a otra con *Lifetime Subtyping*

*Lifetime subtyping* especifica que un *lifetime* debería sobrevivir a otra
*lifetime*. Para explorar el subtipado de *lifetime*, imagina que queremos
escribir un analizador sintáctico. Usaremos una estructura llamada `Context`
que contiene una referencia al *string* que estamos analizando. Escribiremos
un analizador que analizará este *string* y devolverá el éxito o el fracaso.
El analizador necesitará tomar prestado el `Contexto` para hacer el análisis
sintáctico. El listado 19-12 implementa este código de analizador, excepto
que el código no tiene las anotaciones de *lifetime* requeridas, por lo que
no se compilará.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
struct Context(&str);

struct Parser {
    context: &Context,
}

impl Parser {
    fn parse(&self) -> Result<(), &str> {
        Err(&self.context.0[1..])
    }
}
```

<span class="caption">Listado 19-12: Definición de un analizador sin
anotaciones de *lifetime*</span>

La compilación del código da como resultado errores porque Rust espera
parámetros de *lifetime*
en el *string slice* en `Context` y la referencia a un `Context` en
`Parser`.

Para simplificar, la función `parse` devuelve `Result<(), &str>`. Es decir,
la función no tendrá éxito y, en caso de error, devolverá la parte del
*string slice* que no se analizó correctamente. Una implementación real
proporcionaría más información de error y devolvería un tipo de datos
estructurados cuando el análisis tenga éxito. No discutiremos esos detalles
porque no son relevantes para la parte de *lifetimes* de este ejemplo.

Para mantener este código simple, no escribiremos ninguna lógica de análisis.
Sin embargo, es muy es probable que en algún lugar de la lógica de análisis
manejemos la entrada no válida por devolver un error que hace referencia a la
parte de la entrada que no es válida; esta referencia es lo que hace que el
ejemplo del código sea interesante en lo que respecta a los *lifetimes*.
Supongamos que la lógica de nuestro analizador es que la entrada no es válida
después del primer byte. Tenga en cuenta que este código puede entrar en
pánico si el primer byte no está en un límite de caracteres válido;
nuevamente, estamos simplificando el ejemplo para enfocarnos en los
*lifetime* involucradas.

Para obtener este código para compilar, debemos completar los parámetros de
*lifetime* para el *string slic*e en `Context` y la referencia al `Context`
en `Parser`. La forma más sencilla de hacerlo es usar el mismo nombre de
*lifetime* en todas partes, como se muestra en el listado 19-13. Recuerde la
sección “Anotaciones de *lifetime* en definiciones de la Estructura” en el
Capítulo 10 que cada uno de `struct Context<'a>`, `struct Parser<'a>`, y
`impl<'a>`  está declarando un nuevo parámetro de *lifetime*.
Mientras que sus nombres pasan a ser todos iguales, los tres parámetros de
*lifetime* declarados en este ejemplo no están relacionados.

<span class="filename">Filename: src/lib.rs</span>

```rust
struct Context<'a>(&'a str);

struct Parser<'a> {
    context: &'a Context<'a>,
}

impl<'a> Parser<'a> {
    fn parse(&self) -> Result<(), &str> {
        Err(&self.context.0[1..])
    }
}
```

<span class="caption">Listado 19-13: anotando todas las referencias en
`Context` y `Parser` con parámetros de *lifetime*</span>

Este código compila muy bien. Le dice a Rust que un `Parser` contiene una
referencia a `Context` con un  `'a` de *lifetime* y que `Context` contiene un
*string slice* que también dura tanto como la referencia al `Context` en
`Parser`. El mensaje de error del compilador de Rust establecía que se
requerían parámetros de *lifetime* para estas referencias, y ahora hemos
agregado parámetros de *lifetime*.

A continuación, en el listado 19-14, agregaremos una función que toma una
instancia de `Context`, usa un `Parser` para analizar ese contexto y devuelve
lo que `parse` devuelve. Este código no funciona del todo.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
fn parse_context(context: Context) -> Result<(), &str> {
    Parser { context: &context }.parse()
}
```

<span class="caption">Listado 19-14: Un intento de agregar una función
`parse_context` que toma ` ontext` y usa `Parser`</span>

Obtenemos dos errores detallados cuando tratamos de compilar el código con la
adición de la función `parse_context`:

```text
error[E0597]: borrowed value does not live long enough
  --> src/lib.rs:14:5
   |
14 |     Parser { context: &context }.parse()
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^ does not live long enough
15 | }
   | - temporary value only lives until here
   |
note: borrowed value must be valid for the anonymous lifetime #1 defined on the function body at 13:1...
  --> src/lib.rs:13:1
   |
13 | / fn parse_context(context: Context) -> Result<(), &str> {
14 | |     Parser { context: &context }.parse()
15 | | }
   | |_^

error[E0597]: `context` does not live long enough
  --> src/lib.rs:14:24
   |
14 |     Parser { context: &context }.parse()
   |                        ^^^^^^^ does not live long enough
15 | }
   | - borrowed value only lives until here
   |
note: borrowed value must be valid for the anonymous lifetime #1 defined on the function body at 13:1...
  --> src/lib.rs:13:1
   |
13 | / fn parse_context(context: Context) -> Result<(), &str> {
14 | |     Parser { context: &context }.parse()
15 | | }
   | |_^
```

Estos errores indican que la instancia `Parser` que se crea y el parámetro
`context` solo se muestran vivos hasta el final de la función
`parse_context`. Pero ambos necesitan vivir durante toda la vida de la
función.

En otras palabras, `Parser` y `context` necesitan *sobrevivir* a la función
completa y ser válidos antes de que comience la función, así como después de
que termine, para que todas las referencias en este código sean siempre
válidas. El `Parser` que estamos creando y el parámetro `context` salen del
ámbito al final de la función, porque `parse_context` toma posesión de
`context`.

Para descubrir por qué ocurren estos errores, veamos nuevamente las
definiciones en el Listado 19-13, específicamente las referencias en la firma
del método `parse`:

```rust,ignore
    fn parse(&self) -> Result<(), &str> {
```

¿Recuerdas las reglas de elisión?. Si anotamos los *lifetimes* de las
referencias en lugar de elidir, la firma sería la siguiente:

```rust,ignore
    fn parse<'a>(&'a self) -> Result<(), &'a str> {
```

Es decir, la parte de error del valor de retorno de `parse` tiene un
*lifetime* que está vinculado al *lifetime* de la instancia `Parser`
(aquella de `&self` en la firma del método `parse`). Eso tiene sentido: el
*string slice* devuelto hace referencia al *string slice* en la instancia de
`Contexto` sostenido por el `Parser`, y la definición de la estructura
`Parser` especifica que el *lifetime* de la referencia a `Contexto` y el
*lifetime* del *string slice* que `Context` tiene que ser el mismo.

El problema es que la función `parse_context` devuelve el valor devuelto por
`parse`, por lo que el *lifetime* del valor de retorno de `parse_context`
está relacionado con el *lifetime* del `Parser` también. Pero la instancia
`Parser` creada en la función `parse_context` no sobrevivirá al final de la
función (es temporal), y `context` saldrá del ámbito al final de la función
(`parse_context` se apropia de eso).

Rust piensa que estamos intentando devolver una referencia a un valor que
sale del alcance al final de la función, porque anotamos todos los *lifetime*
con el mismo parámetro de *lifetime*. Las anotaciones le dijeron a Rust que
la duración del *string slice* que `Context` contiene es la misma que la
duración de la referencia al `Context` que `Parser` contiene.

La función `parse_context` no puede ver que dentro de la función `parse`, el
*string slice* devuelto sobrevivirá `Context` y `Parser` y que la referencia
`parse_context` devuelta se refiere al *string slice*, no a `Context` o
`Parser`.

Al saber lo que hace la implementación de `parse`, sabemos que la única razón
por la que el valor de retorno de `parse` está vinculado a la instancia
`Parser` es que hace referencia al `Context` de la instancia `Parser`, que
hace referencia al *string slice*. Por lo tanto, es realmente el *lifetime*
del *string slice* el que `parse_context` necesita preocuparse. Necesitamos
una manera de decirle a Rust que el *string slice* en `Context` y la
referencia al `Context` en `Parser` tienen diferentes *lifetime* y que el
valor de retorno de `parse_context` está vinculado a la duración del *string
slice* en `Context`.

Primero, trataremos de darle a `Parser` y `Context` diferentes parámetros de
*lifetime*, como se muestra en el Listado 19-15. Utilizaremos `'s` y `'c`
como nombres de parámetros de *lifetime* para aclarar qué duración va con el
*string slice* en `Context` y que va con la referencia a `Context` en
`Parser`. Tenga en cuenta que esta solución no solucionará completamente el
problema, pero es un comienzo. Veremos por qué esta solución no es suficiente
cuando intentamos compilar.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
struct Context<'s>(&'s str);

struct Parser<'c, 's> {
    context: &'c Context<'s>,
}

impl<'c, 's> Parser<'c, 's> {
    fn parse(&self) -> Result<(), &'s str> {
        Err(&self.context.0[1..])
    }
}

fn parse_context(context: Context) -> Result<(), &str> {
    Parser { context: &context }.parse()
}
```

<span class="caption">Listado 19-15: Especificación de diferentes parámetros
de *lifetime* para las referencias al string slice y al `Context`</span>

Hemos anotado los *lifetimes* de las referencias en todos los mismos lugares
donde las anotamos en el listado 19-13. Pero esta vez usamos diferentes
parámetros dependiendo de si la referencia va con el *string slice* o con
`Context`. También hemos agregado una anotación a la parte del *string slice*
del valor de retorno de `parse` para indicar que va con el *lifetimes* del
*string slice* en `Context`.

Cuando intentamos compilar ahora, obtenemos el siguiente error:

```text
error[E0491]: in type `&'c Context<'s>`, reference has a longer lifetime than the data it references
 --> src/lib.rs:4:5
  |
4 |     context: &'c Context<'s>,
  |     ^^^^^^^^^^^^^^^^^^^^^^^^
  |
note: the pointer is valid for the lifetime 'c as defined on the struct at 3:1
 --> src/lib.rs:3:1
  |
3 | / struct Parser<'c, 's> {
4 | |     context: &'c Context<'s>,
5 | | }
  | |_^
note: but the referenced data is only valid for the lifetime 's as defined on the struct at 3:1
 --> src/lib.rs:3:1
  |
3 | / struct Parser<'c, 's> {
4 | |     context: &'c Context<'s>,
5 | | }
  | |_^
```

Rust no sabe de ninguna relación entre `'c` y `'s`. Para que sea válida, los
datos a los que se hace referencia en `Context` con `'s` de *lifetime* deben
restringirse para garantizar que vivan más tiempo que la referencia con `'c`
de *lifetime*. Si `'s` no es más largo que `'c`, la referencia a `Context`
podría no ser válida.

Ahora llegamos al punto de esta sección: la función Rust *lifetime subtyping*
especifica que un parámetro de *lifetime* dura al menos tanto como otro. En
los paréntesis angulares donde declaramos los parámetros de *lifetime*,
podemos declarar un `'a`  de *lifetime* como de costumbre y declarar un `'b`
vitalicio que viva al menos tan largo como `'a` declarando `'b` usando la
sintaxis `'b:'a`.

En nuestra definición de `Parser`, para decir que `'s` (el *lifetime* útil
del *string slice*) se garantiza que durará al menos tanto como `'c` (el
*lifetime* de la referencia al `Contexto`), cambiamos las declaraciones de
*lifetime* para que se vean así:

<span class="filename">Filename: src/lib.rs</span>

```rust
# struct Context<'a>(&'a str);
#
struct Parser<'c, 's: 'c> {
    context: &'c Context<'s>,
}
```

Ahora la referencia a `Context` en `Parser` y la referencia al *string
slice* en `Context` tienen diferentes *lifetimes*; nos hemos asegurado de que
la duración del *string slice* sea más larga que la referencia al `Context`.

Ese fue un ejemplo muy largo, pero como mencionamos al comienzo de este
capítulo, las características avanzadas de Rust son muy específicas. A menudo
no necesitará la sintaxis que describimos en este ejemplo, pero en tales
situaciones, sabrá cómo referirse a algo y darle la vida necesaria.

### *Lifetime Bounds* en las referencias a los tipos genéricos

En la sección “Trait Bounds” del Capítulo 10, discutimos el uso de *trait
bounds* en los tipos genéricos. También podemos agregar parámetros de
*lifetime* como restricciones en tipos genéricos; estos se llaman *lifetime
bounds*. Los *lifetime bounds* ayudan a Rust a verificar que las referencias
en tipos genéricos no sobrevivan datos a los que hacen referencia.

Como ejemplo, considere un tipo que sea un contenedor de referencias.
Recuerde el tipo “`RefCell<T>` de la sección `RefCell<T>` y el Patrón de
Mutabilidad Interior” en el Capítulo 15: sus métodos `borrow` y `borrow_mut`
devuelven los tipos `Ref` y `RefMut`, respectivamente . Estos tipos son
envoltorios sobre referencias que hacen un seguimiento de las reglas de
endeudamiento en tiempo de ejecución. La definición de la estructura `Ref` se
muestra en el listado 19-16, sin *lifetime bounds* por el momento.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
struct Ref<'a, T>(&'a T);
```

<span class="caption">Listado 19-16: Definición de una estructura para
envolver una referencia a un tipo genérico, sin *lifetime bounds*</span>

Sin restringir explícitamente el ciclo de *lifetime* `'a` en relación con el
parámetro genérico `T`, Rust tendrá un error porque no sabe por cuánto tiempo
vivirá el tipo genérico `T`:

```text
error[E0309]: the parameter type `T` may not live long enough
 --> src/lib.rs:1:19
  |
1 | struct Ref<'a, T>(&'a T);
  |                   ^^^^^^
  |
  = help: consider adding an explicit lifetime bound `T: 'a`...
note: ...so that the reference type `&'a T` does not outlive the data it points at
 --> src/lib.rs:1:19
  |
1 | struct Ref<'a, T>(&'a T);
  |                   ^^^^^^
```

Debido a que `T` puede ser de cualquier tipo,`T` podría ser una referencia o
un tipo que contenga una o más referencias, cada una de las cuales podría
tener su propio *lifetimes*. Rust no puede estar seguro de que `T` viva tanto
tiempo como `'a`.

Afortunadamente, el error proporciona consejos útiles sobre cómo especificar
el *lifetime bound* en este caso:

```text
consider adding an explicit lifetime bound `T: 'a` so that the reference type
`&'a T` does not outlive the data it points at
```

El listado 19-17 muestra cómo aplicar este consejo especificando el *lifetime bound* cuando declaramos el tipo genérico `T`.

```rust
struct Ref<'a, T: 'a>(&'a T);
```

<span class="caption">Listado 19-17: Agregar *lifetime bounds* en `T` para
especificar que las referencias en `T` vivan al menos tanto como `'a`</span>

Este código ahora compila porque la sintaxis `T: 'a` especifica que `T` puede
ser de cualquier tipo, pero si contiene alguna referencia, las referencias
deben vivir al menos tan largo como `'a`.

Podríamos resolver este problema de una manera diferente, como se muestra en
la definición de una estructura `StaticRef` en el listado 19-18, al agregar
el *lifetime bound* `'static` vinculado a `T`. Esto significa que si `T`
contiene referencias, deben tener la duración `'estática`.

```rust
struct StaticRef<T: 'static>(&'static T);
```

<span class="caption">Listado 19-18: Agregar un `'static` *lifetime bound*
enlazado a `T` para restringir `T` a los tipos que tienen solo `'static` o no
referencias</span>

Porque `'static` significa que la referencia debe vivir todo el
programa completo, un tipo que no contiene referencias, cumpla con los
criterios de todas las referencias que viven tanto como todo el programa
(porque no hay referencias). Para el comprobador de préstamos preocupado por
referencias que viven lo suficiente, no existe una distinción real entre un
tipo que no tiene referencias y un tipo que tiene referencias que viven para
siempre: las dos son las mismas para determinar si una referencia tiene una
vida más corta a la que se refiere.

### Inferencia de los tiempos de vida del *Trait Object*

En el capítulo 17 en la sección “Uso de *Trait Objects* que permiten valores
de diferentes tipos”, discutimos los *trait objects*, que consisten en un
*trait* detrás de una referencia, que nos permiten usar el
*despacho dinámico*. Todavía no hemos discutido qué sucede si el tipo que
implementa el *trait objects* de *trait* tiene un *lifetime*. Considere el
Listado 19-19 donde tenemos un *trait* `Red` y una estructura `Ball`. La
estructura `Ball` contiene una referencia (y por lo tanto tiene un parámetro
de *lifetime*) y también implementa el *trait* `Red`. Queremos usar una
instancia de `Ball` como el trait object `Box<Red>`.

<span class="filename">Filename: src/main.rs</span>

```rust
trait Red { }

struct Ball<'a> {
    diameter: &'a i32,
}

impl<'a> Red for Ball<'a> { }

fn main() {
    let num = 5;

    let obj = Box::new(Ball { diameter: &num }) as Box<Red>;
}
```

<span class="caption">Listado 19-19: Usar un tipo que tiene un parámetro de
*lifetime* con un *trait object*</span>

Este código compila sin ningún error, aunque no hemos anotado explícitamente
los *lifetimes* involucradas en `obj`. Este código funciona porque hay reglas
para trabajar con *lifetimes* y *trait objects*:

* La *lifetime* por defecto de un *trait object* es `'static`.
* Con `&'a Trait` o `&'a mut Trait`, la duración predeterminada del
 *trait object* es `'a`.
* Con una sola cláusula `T: 'a`, la duración predeterminada del
 *trait object* es `'a`.
* Con múltiples cláusulas como `T: 'a`, no hay un *lifetime* por defecto;
 debemos ser explícitos

Cuando debemos ser explícitos, podemos agregar un *lifetime bound* en un
*trait object* como `Box<Red>` usando la sintaxis `Box<Red + 'static>` o
`Box<Red + 'a>`, dependiendo de si la referencia vive todo el programa o no.
Al igual que con los otros límites, la sintaxis que agrega un *lifetime bound*
significa que cualquier implementador del *trait* `Red` que tiene referencias
dentro del tipo debe tener la misma *lifetime* especificada en los *trait
object bounds* que esas referencias.

A continuación, veamos algunas otras características avanzadas que
administran *trait*.
