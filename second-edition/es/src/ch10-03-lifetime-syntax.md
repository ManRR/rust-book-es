## Validación de referencias con *Lifetimes*

Un detalle que no discutimos en la sección “Referencias y préstamos” en el
Capítulo 4 es que cada referencia en Rust tiene un *lifetime*, que es el
alcance para el cual esa referencia es válida. La mayoría de las veces, las
vidas son implícitas e inferidas, al igual que la mayoría de las veces, los
tipos son inferidos. Debemos anotar tipos cuando múltiples tipos son
posibles. De manera similar, debemos anotar las vidas cuando la vida de las
referencias se puede relacionar de diferentes maneras. Rust nos exige anotar
las relaciones utilizando parámetros genéricos de por vida para garantizar
que las referencias reales utilizadas en el tiempo de ejecución sean
definitivamente válidas.

El concepto de *tiempo de vida* (*lifetimes*) es algo diferente de las
herramientas en otros
lenguajes de programación, lo que podría decirse que hace que la
característica más distintiva de Rust sea la vida. Aunque no cubriremos las
vidas útiles en su totalidad en este capítulo, discutiremos las formas
comunes en que puede encontrar la sintaxis de por vida para que pueda
familiarizarse con los conceptos. Consulte la sección “Advanced Lifetimes” en
el Capítulo 19 para obtener información más detallada.

### Previniendo *Dangling References* con *Lifetimes*

El objetivo principal de los *tiempos de vida* (*lifetimes*) es evitar las
referencias pendientes, que hacen que un programa haga referencia a datos
distintos de los datos a los que se pretende hacer referencia. Considere el
programa en el listado 10-17, que tiene un alcance externo y un alcance
interno.

```rust,ignore
{
    let r;

    {
        let x = 5;
        r = &x;
    }

    println!("r: {}", r);
}
```

<span class="caption">Listado 10-17: un intento de usar una referencia cuyo
valor ha salido del alcance</span>

> Nota: Los ejemplos en los listados 10-17, 10-18 y 10-24 declaran variables
> sin darles un valor inicial, por lo que el nombre de la variable existe en
> el ámbito externo. A primera vista, esto podría parecer en conflicto con que
> Rust no tenga valores nulos. Sin embargo, si tratamos de usar una variable
> antes de darle un valor, obtendremos un error en tiempo de compilación, que
> muestra que Rust no permite valores nulos.

El alcance externo declara una variable llamada `r` sin valor inicial, y el
alcance interno declara una variable llamada `x` con el valor inicial de 5.
Dentro del alcance interno, intentamos establecer el valor de `r` como
referencia a `x`. Luego, el alcance interno finaliza e intentamos imprimir el
valor en `r`. Este código no se compilará porque el valor `r` hace referencia
a que se ha salido del alcance antes de intentar usarlo. Aquí está el mensaje
de error:

```text
error[E0597]: `x` does not live long enough
  --> src/main.rs:7:5
   |
6  |         r = &x;
   |              - borrow occurs here
7  |     }
   |     ^ `x` dropped here while still borrowed
...
10 | }
   | - borrowed value needs to live until here
```

La variable `x` no “vive lo suficiente”. La razón es que `x` estará fuera del
alcance cuando el alcance interno termine en la línea 7. Pero `r` sigue
siendo válido para el alcance externo; debido a que su alcance es mayor,
decimos que “vive más tiempo”. Si Rust permitía que este código funcionara,
`r` estaría haciendo referencia a la memoria que fue desasignada cuando `x`
salió del alcance, y todo lo que tratamos de hacer con `r` no funcionaría
correctamente. Entonces, ¿cómo determina Rust que este código no es válido?
utiliza un *comprobador de préstamos* (*borrow checker*).

### El *comprobador de préstamos* (*Borrow Checker*)

El compilador Rust tiene un *comprobador de préstamos* que compara los
ámbitos para determinar si todos los préstamos son válidos. El Listado 10-18
muestra el mismo código que el Listado 10-17 pero con anotaciones que
muestran la vida útil de las variables.

```rust,ignore
{
    let r;                // ---------+-- 'a
                          //          |
    {                     //          |
        let x = 5;        // -+-- 'b  |
        r = &x;           //  |       |
    }                     // -+       |
                          //          |
    println!("r: {}", r); //          |
}                         // ---------+
```

<span class="caption">Listado 10-18: Anotaciones de los *lifetimes* de `r` y
`x`, llamadas `'a` y`'b`, respectivamente</span>

Aquí, hemos anotado el *lifetime* de `r` con `'a` y la *lifetime* de `x`
con `'b`. Como puede ver, el bloque interno `'b` es mucho más pequeño que el
bloque de *lifetime* externo `'a`. En tiempo de compilación, Rust compara el tamaño
de los dos *lifetimes* y ve que `r` tiene un *lifetime* de `'a` pero que se refiere a la
memoria con un *lifetime* de `'b`. El programa se rechaza porque `'b` es más corto
que`'a`: el sujeto de la referencia no vive tanto tiempo como la referencia.

El Listado 10-19 corrige el código para que no tenga una referencia que cuelga y se compila sin ningún error.

```rust
{
    let x = 5;            // ----------+-- 'b
                          //           |
    let r = &x;           // --+-- 'a  |
                          //   |       |
    println!("r: {}", r); //   |       |
                          // --+       |
}                         // ----------+
```

<span class="caption">Listado 10-19: Una referencia válida porque los datos
tienen una *lifetimes* más larga que la referencia</span>

Aquí, `x` tiene la duración ``b`, que en este caso es más grande que `'a`.
Esto significa que `r` puede hacer referencia a `x` porque Rust sabe que la
referencia en `r` siempre será válida, mientras que `x` es válida.

Ahora que sabe dónde están los *lifetimes* de las referencias y cómo Rust
analiza los tiempos de vida para garantizar que las referencias siempre serán
válidas, exploremos los *lifetimes* genéricos de los parámetros y los
valores de retorno en el contexto de las funciones.

### *Generic Lifetimes* en funciones

Vamos a escribir una función que devuelva el mayor de dos *string slices*.
Esta función tomará dos *string slices* y devolverá un *string slice*.
Después de que hemos implementado la función `longest`, el código
en el Listado 10-20 debería imprimir
`The longest string is abcd`.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}
```

<span class="caption">Listado 10-20: Una función `main` que llama a la
función `longest` para encontrar el más largo de dos *string slices*</span>

Tenga en cuenta que queremos que la función tome *string slices*, que son
referencias, porque no queremos que la función `longest` tome posesión de sus
parámetros. Queremos permitir que la función acepte *slices* de un `String`
(el tipo almacenado en la variable `string1`) así como de literales de
*string* (que es lo que contiene la variable `string2`).

Consulte la sección “*String Slices* como parámetros” en el Capítulo 4 para
obtener más información sobre por qué los parámetros que utilizamos en el
listado 10-20 son los que queremos.

Si tratamos de implementar la función `longest` como se muestra en el Listado
10-21, no se compilará.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

<span class="caption">Listado 10-21: una implementación de la función
`longest` que devuelve el mayor de dos *string slices* pero aún no
compila</span>

En cambio, obtenemos el siguiente error que habla de *lifetimes*:

```text
error[E0106]: missing lifetime specifier
 --> src/main.rs:1:33
  |
1 | fn longest(x: &str, y: &str) -> &str {
  |                                 ^ expected lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the
signature does not say whether it is borrowed from `x` or `y`
```

El texto de ayuda revela que el tipo de devolución necesita un parámetro
genérico de *lifetimes* porque Rust no puede decir si la referencia que se
devuelve se refiere a `x` o `y`. En realidad, tampoco lo sabemos, porque el
bloque `if` en el cuerpo de esta función devuelve una referencia a `x` y el
bloque `else` devuelve una referencia a `y`!

Cuando estamos definiendo esta función, no conocemos los valores concretos
que se pasarán a esta función, por lo que no sabemos si se ejecutará el caso
`if` o el caso `else`. Tampoco conocemos los *lifetimes* concretos de las
referencias que se enviarán, por lo que no podemos mirar los ámbitos como lo hicimos en los listados 10-18 y 10-19 para determinar si la referencia que devolvemos siempre será válida. . El comprobador de préstamos tampoco puede determinar esto, porque no sabe cómo los *lifetimes* de `x` y `y` se relacionan con el *lifetimes* del valor de retorno. Para corregir este error, agregaremos parámetros genéricos de *lifetimes* que definan la relación entre las referencias para que el corrector de préstamos pueda realizar su análisis.

### *Lifetime* sintaxis de anotación

Las anotaciones de *lifetime* no cambian la duración de ninguna de las
referencias. Del mismo modo que las funciones pueden aceptar cualquier tipo
cuando la firma especifica un parámetro de tipo genérico, las funciones
pueden aceptar referencias de cualquier *lifetime* especificando un parámetro
genérico de *lifetime*. Las anotaciones de *lifetime* describen las relaciones de
*lifetime* de referencias múltiples entre sí sin afectar al *lifetimes*.

Las anotaciones de *lifetime* tienen una sintaxis levemente inusual: los
nombres de los parámetros de *lifetime* deben comenzar con un apóstrofo (`'`)
y generalmente son todos minúsculos y muy cortos, como los tipos genéricos.
La mayoría de las personas usa el nombre `'a`. Colocamos anotaciones de
parámetros de *lifetime* después de `&` de una referencia, usando un espacio
para separar la anotación del tipo de referencia.

Aquí hay algunos ejemplos: una referencia a un `i32` sin un parámetro
*lifetime*, una referencia a un `i32` que tiene un parámetro de *lifetime*
llamado `'a`, y una referencia mutable a un `i32` que también tiene el
*lifetime* `'a`.

```rust,ignore
&i32        // a reference
&'a i32     // a reference with an explicit lifetime
&'a mut i32 // a mutable reference with an explicit lifetime
```

Una anotación de por vida por sí misma no tiene mucho significado, porque las
anotaciones están destinadas a decirle a Rust cómo se relacionan entre sí los
parámetros genéricos de *lifetime* de las múltiples referencias. Por ejemplo,
supongamos que tenemos una función con el parámetro `first` que es una
referencia a `i32` con un valor de *lifetime* `'a`. La función también tiene
otro parámetro llamado `segundo` que es otra referencia a un `i32` que
también tiene el *lifetime* `'a`. Las anotaciones de *lifetime* indican que
las referencias `first` y `second` deben vivir tanto tiempo como el
*lifetime* genérico.

### Anotaciones *Lifetime* en firmas de funciones

Ahora examinemos las anotaciones de *lifetime* en el contexto de la función
`longest`. Al igual que con los parámetros de tipo genérico, debemos declarar
los parámetros genéricos de *lifetime* dentro de corchetes angulares entre el
nombre de la función y la lista de parámetros. La restricción que deseamos
expresar en esta firma es que todas las referencias en los parámetros y el
valor de retorno deben tener el mismo *lifetime*. Le asignaremos el nombre del
*lifetime* `'a` y luego lo agregaremos a cada referencia, como se muestra en
el listado 10-22.

<span class="filename">Filename: src/main.rs</span>

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

<span class="caption">Listado 10-22: La definición de la función `longest`
que especifica que todas las referencias en la firma deben tener el mismo
*lifetime* `'a`</span>

Este código debe compilar y producir el resultado que queremos cuando lo
usamos con el función `main` en el listado 10-20.

La firma de la función ahora le dice a Rust que durante algún *lifetime* `'a`,la función toma dos parámetros, ambos son *string slice* que viven al menos
tanto como el *lifetime* `'a`. La firma de la función también le dice a Rust
que la *string slices* devuelta de la función vivirá al menos tanto como el
*lifetime* `'a`.
Estas restricciones son lo que queremos que Rust haga cumplir. Recuerde,
cuando especificamos los parámetros de *lifetime* en esta firma de función,
no estamos cambiando el *lifetime* de cualquier valor pasado o devuelto. Por
el contrario, estamos especificando que el comprobador de préstamos debería
rechazar cualquier valor que no se adhiera a estas restricciones. Tenga en cuenta que la función `longest` no necesita saber exactamente durante cuánto tiempo `x` y `y` van a vivir, solo que algún alcance puede ser sustituido
por `'a` que satisfará esta firma.

Al anotar *lifetimes* en funciones, las anotaciones van en la firma de
función, no en el cuerpo de la función.Rust puede analizar el código dentro
de la función sin ayuda. Sin embargo, cuando una función tiene referencias hacia o desde un código fuera de esa función, es casi imposible para Rust
calcular los *lifetimes* de los parámetros o valores de retorno por sí mismo.
Los *lifetimes* pueden ser diferentes cada vez que se llama a la función. Es
por eso que necesitamos anotar los *lifetimes* manualmente.

Cuando pasamos referencias concretas a `longest`, el *lifetimes* concreto es
sustituido por `'a` es la parte del alcance de `x` que se superpone con el
alcance de `y` En otras palabras, el *lifetimes* `'a` obtendrá el *lifetimes*
concreto que es igual a la más pequeña de los *lifetimes* de `x` y `y`.
Debido a que hemos anotado la referencia devuelta con el mismo parámetro de
*lifetimes* `'a`, la referencia devuelta también será válida para el
*lifetimes* de la menor de las vidas de `x` y `y`.

Veamos cómo las anotaciones de *lifetimes* restringen la función `longest`
pasando referencias que tienen diferentes *lifetimes* concretos. El listado
10-23 es un ejemplo directo.

<span class="filename">Filename: src/main.rs</span>

```rust
# fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
#     if x.len() > y.len() {
#         x
#     } else {
#         y
#     }
# }
#
fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("The longest string is {}", result);
    }
}
```

<span class="caption">Listado 10-23: Uso de la función `longest` con
referencias a valores `String` que tienen diferentes *lifetimes*
concretos</span>

En este ejemplo, `string1` es válido hasta el final del alcance externo,
`string2` es válido hasta el final del alcance interno, y `result` hace
referencia a algo que es válido hasta el final del alcance interno. Ejecute
este código, y verá que el verificador de préstamos aprueba este código;
compilará e imprimirá
`The longest string is long string is long`.

A continuación, probemos con un ejemplo que muestra que el *lifetime* de la
referencia en `result` debe ser el menor *lifetime* de los dos argumentos.
Vamos a mover la declaración de la variable `result` fuera del alcance
interno, pero dejaremos la asignación del valor a la variable `result` dentro del alcance con `string2`. Luego moveremos el `println!` que usa `result`
fuera del alcance interno, después de que el alcance interno haya finalizado.
El código en el listado 10-24 no se compilará.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let string1 = String::from("long string is long");
    let result;
    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }
    println!("The longest string is {}", result);
}
```

<span class="caption">Listing 10-24: Intentar utilizar `result` después de
`string2` ha salido del alcance</span>

Cuando intentemos compilar este código, obtendremos este error:

```text
error[E0597]: `string2` does not live long enough
  --> src/main.rs:15:5
   |
14 |         result = longest(string1.as_str(), string2.as_str());
   |                                            ------- borrow occurs here
15 |     }
   |     ^ `string2` dropped here while still borrowed
16 |     println!("The longest string is {}", result);
17 | }
   | - borrowed value needs to live until here
```

El error muestra que para que `result` sea válido para la instrucción
`println!`, `String2` debería ser válido hasta el final del alcance externo.
Rust lo sabe porque anotamos las *lifetimes* de los parámetros de función y los valores de retorno utilizando el mismo parámetro *lifetimes* `'a`.

Como humanos, podemos mirar este código y ver que `string1` es más largo que
`string2` y por lo tanto `result` contendrá una referencia a `string1`.
Debido a que `string1` no ha salido del ámbito, una referencia a `string1`
seguirá siendo válida para la instrucción `println!`. Sin embargo, el
compilador no puede ver que la referencia es válida en este caso. Le hemos
dicho a Rust que la *lifetime* de la referencia devuelta por la función
`longest` es la misma que la menor de las *lifetimes* de las referencias pasadas. Por lo tanto, el verificador de préstamos no permite el código en el Listado 10-24 como posiblemente tener una referencia inválida.

Intente diseñar más experimentos que varíen los valores y los *lifetimes* de
las referencias pasadas a la función `longest` y cómo se utiliza la
referencia devuelta. Haga hipótesis sobre si sus experimentos pasarán o no el
comprobador de préstamos antes de compilar; ¡luego verifica si tienes razón!

### Pensando en términos de *Lifetimes*

La forma en que necesita especificar parámetros de *lifetime* depende de lo que esté haciendo su función. Por ejemplo, si cambiamos la implementación de la función `longest` para devolver siempre el primer parámetro en lugar del *string slice* más largo, no necesitaríamos especificar un *lifetime* en el parámetro `y`. El siguiente código se compilará:

<span class="filename">Filename: src/main.rs</span>

```rust
fn longest<'a>(x: &'a str, y: &str) -> &'a str {
    x
}
```

En este ejemplo, hemos especificado un parámetro de *lifetime* `'a` para el
parámetro `x` y el tipo de retorno, pero no para el parámetro `y`, porque la
duración de `y` no tiene ninguna relación con la duración de `x` o el valor
de retorno.

Al devolver una referencia desde una función, el parámetro *lifetime* para el
tipo de retorno debe coincidir con el parámetro de duración de uno de los
parámetros. Si la referencia devuelta *no* hace referencia a uno de los
parámetros, debe hacer referencia a un valor creado dentro de esta función,
que sería una referencia colgante porque el valor saldrá del alcance al final
de la función. Considere esta implementación intentada de la función
`longest` que no compilará:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn longest<'a>(x: &str, y: &str) -> &'a str {
    let result = String::from("really long string");
    result.as_str()
}
```

Aquí, aunque hemos especificado un parámetro de *lifetime* `'a` para el tipo de devolución, esta implementación no podrá compilarse porque la duración del
valor de retorno no está relacionada con la duración de los parámetros. Aquí está el mensaje de error que obtenemos:

```text
error[E0597]: `result` does not live long enough
 --> src/main.rs:3:5
  |
3 |     result.as_str()
  |     ^^^^^^ does not live long enough
4 | }
  | - borrowed value only lives until here
  |
note: borrowed value must be valid for the lifetime 'a as defined on the
function body at 1:1...
 --> src/main.rs:1:1
  |
1 | / fn longest<'a>(x: &str, y: &str) -> &'a str {
2 | |     let result = String::from("really long string");
3 | |     result.as_str()
4 | | }
  | |_^
```

El problema es que `result` sale del alcance y se limpia al final de la
función `longest`. También estamos tratando de devolver una referencia al
`result` de la función. No hay forma de que podamos especificar parámetros
de *lifetime* que cambiarían la *referencia colgante* (*dangling reference*),y Rust no nos permitirá crear una referencia colgante. En este caso, la
mejor solución sería devolver un tipo de datos de propiedad en lugar de una
referencia, por lo que la función de llamada es responsable de limpiar el valor.

En última instancia, la sintaxis de *lifetime* trata de conectar los
*lifetimes* de varios parámetros y devolver valores de funciones. Una vez
que están conectados, Rust tiene suficiente información para permitir
operaciones de memoria segura y no permitir operaciones que podrían crear
punteros colgantes o violar la seguridad de la memoria.

### Anotaciones *Lifetime* en las definiciones de la estructura

Hasta ahora, solo hemos definido estructuras para mantener los tipos de
propiedad. Es posible que las estructuras tengan referencias, pero en ese
caso necesitaríamos agregar una anotación *lifetime* en cada referencia en
la definición de la estructura. El listado 10-25 tiene una estructura
llamada `ImportantExcerpt` que contiene un *string slice*.

<span class="filename">Filename: src/main.rs</span>

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.')
        .next()
        .expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
}
```

<span class="caption">Listado 10-25: Una estructura que contiene una
referencia, por lo que su definición necesita una anotación de
*lifetime*</span>

Esta estructura tiene un campo, `part`, que contiene un *string slice*,
que es una referencia. Al igual que con los tipos de datos genéricos,
declaramos el nombre del parámetro genérico *lifetime* dentro de corchetes
angulares después del nombre de la estructura para que podamos usar el
parámetro de duración en el cuerpo de la definición de estructura. Esta
anotación significa que una instancia de `ImportantExcerpt` no puede
sobrevivir a la referencia que contiene en su campo `part`.

La función `main` aquí crea una instancia de la estructura
`ImportantExcerpt` que contiene una referencia a la primera *sentence* de
`String` propiedad de la variable `novel`. Los datos en `novel` existen
antes de que se cree la instancia `ImportantExcerpt`. Además, `novel` no
sale del alcance hasta que el `ImportantExcerpt` salga del ámbito, por lo
que la referencia en la instancia `ImportantExcerpt` es válida.

### Lifetime Elision

Aprendió que cada referencia tiene una *lifetime* y que necesita especificar
parámetros de *lifetime* para funciones o estructuras que usan referencias.
Sin embargo, en el Capítulo 4 teníamos una función en el Listado 4-9, que se
muestra nuevamente en el Listado 10-26, compilada sin anotaciones de
*lifetime*.

<span class="filename">Filename: src/lib.rs</span>

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

<span class="caption">Listado 10-26: una función que definimos en el listado
4-9 que compilamos sin anotaciones de *lifetime*, aunque el parámetro y el
tipo de retorno son referencias</span>

La razón por la que esta función se compila sin anotaciones de *lifetime* es
histórica: en las primeras versiones (anteriores a la 1.0) de Rust, este
código no se habría compilado porque cada referencia necesitaba una
*lifetime* explícita. En ese momento, la firma de la función se habría
escrito así:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Después de escribir mucho código de Rust, el equipo de Rust descubrió que
los programadores de Rust estaban ingresando las mismas anotaciones de
*lifetime* una y otra vez en situaciones particulares. Estas situaciones
eran predecibles y seguían algunos patrones deterministas. Los
desarrolladores programaron estos patrones en el código del compilador para
que el verificador de préstamos pudiera inferir los *lifetime* en estas
situaciones y no necesitara anotaciones explícitas.

Esta parte de la historia de Rust es relevante porque es posible que surjan patrones más deterministas y se agreguen al compilador. En el futuro, aún se necesitarán menos anotaciones de *lifetime*.

Los patrones programados en el análisis de referencias de Rust se llaman
*lifetime elision rules*. Estas no son reglas que los programadores deben
seguir; son un conjunto de casos particulares que el compilador considerará
y si su código se ajusta a estos casos, no necesita escribir los *lifetime*
de forma explícita.

Las reglas de *elisión* no proporcionan una inferencia completa. Si Rust
aplica las reglas de manera determinista pero todavía hay ambigüedad en
cuanto a los *lifetimes* que tienen las referencias, el compilador no
adivinará cuál debería ser el *lifetime* de las referencias restantes. En
este caso, en lugar de adivinar, el compilador le dará un error que puede
resolver agregando las anotaciones de *lifetimes* que especifican cómo se
relacionan las referencias entre sí.

Los periodos de *lifetime* de la función o los parámetros del método se
denominan *input lifetimes*, y los *lifetime* en los valores de retorno se
denominan *output lifetimes*.

El compilador utiliza tres reglas para determinar qué referencias de
*lifetime* tienen cuando no hay anotaciones explícitas. La primera regla se
aplica a los *lifetimes* de las entradas, y las reglas segunda y tercera se
aplican a los *lifetimes* de las salidas. Si el compilador llega al final de
las tres reglas y todavía hay referencias para las cuales no puede calcular
las duraciones, el compilador se detendrá con un error.

La primera regla es que cada parámetro que es una referencia obtiene su
propio parámetro de *lifetimes*. En otras palabras, una función con un
parámetro obtiene un parámetro de *lifetimes*: `fn foo <'a> (x: &' a i32)`;
una función con dos parámetros obtiene dos parámetros de *lifetimes*
separados: `fn foo <'a,' b> (x: & 'a i32, y: &' b i32)`; y así.

La segunda regla es si hay exactamente un parámetro de *lifetime* útil de
entrada, ese tiempo de *lifetimes* está asignado a todos los parámetros de
*lifetimes* útil de salida: `fn foo <'a> (x: &' a i32) -> & 'a i32`.

La tercera regla es si hay múltiples parámetros de *lifetimes* de entrada,
pero uno de ellos es `&self` o `&mut self` porque este es un método, el
*lifetime* de `self` se asigna a todos los parámetros de *lifetime* de
salida. Esta tercera regla hace que los métodos sean mucho más agradables de
leer y escribir porque son necesarios menos símbolos.

Hagamos como si fuéramos el compilador. Aplicaremos estas reglas para averiguar cuáles son las duraciones de las referencias en la firma de la función `first_word` en el listado 10-26. La firma comienza sin ningún *tiempo de vida* (*lifetimes*) asociado con las referencias:

```rust,ignore
fn first_word(s: &str) -> &str {
```

Luego el compilador aplica la primera regla, que especifica que cada
parámetro obtiene su propia *lifetime*. Lo llamaremos `'a` como de costumbre
así que ahora la firma es esta:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
```

La segunda regla se aplica porque hay exactamente una *input lifetime*. La segunda regla especifica que el *lifetime* de un parámetro de entrada se asigna a la *output lifetime*, por lo que la firma es ahora la siguiente:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Ahora todas las referencias en esta firma de función tienen tiempos de vida,
y el compilador puede continuar su análisis sin necesidad de que el programador anote los *lifetimes* en esta firma de función.

Veamos otro ejemplo, esta vez usando la función `longest` que no tenía
parámetros de *lifetimes* cuando comenzamos a trabajar con ella en el Listado
10-21:

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
```

Vamos a aplicar la primera regla: cada parámetro tiene su propio *lifetime*.
Esta vez tenemos dos parámetros en lugar de uno, así que tenemos dos *lifetime*:

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

Puede ver que la segunda regla no se aplica porque hay más de un
*input lifetime*. La tercera regla tampoco se aplica porque `longest` es una
función en lugar de un método, por lo que ninguno de los parámetros es
`self`. Después de trabajar en las tres reglas, todavía no hemos descubierto
cuál es la duración del tipo de devolución. Esta es la razón por la cual
obtuvimos un error al tratar de compilar el código en el Listado 10-21: el
compilador trabajó a través de las reglas de *elisión* de *lifetime*, pero
aún no pudo determinar todos los *lifetimes* de las referencias en la firma.

Debido a que la tercera regla realmente solo se aplica a las firmas de
métodos, veremos los *lifetimes* en ese contexto al lado para ver por qué la
tercera regla significa que no tenemos que anotar *lifetime* en las firmas
de métodos muy a menudo.

### Anotaciones de *Lifetime* en las definiciones de métodos

Cuando implementamos métodos en una estructura con *lifetimes*, utilizamos
la misma sintaxis que la de los parámetros de tipo genérico que se muestran
en el Listado 10-11. Donde declaramos y usamos los parámetros de *lifetime*
depende de si están relacionados con los campos de estructura o los
parámetros de método y los valores de retorno.

Los nombres de *lifetime* para los campos de estructura siempre tienen que
declararse después de la palabra clave `impl` y luego usarse después del
nombre de la estructura, porque esos tiempos de *lifetime* son parte del
tipo de estructura.

En las firmas de métodos dentro del bloque `impl`, las referencias pueden
estar vinculadas a la duración de las referencias en los campos de la
estructura, o pueden ser independientes. Además, las reglas de *elisión* de
*lifetime* a menudo hacen que las anotaciones de *lifetime* no sean
necesarias en las firmas de métodos. Veamos algunos ejemplos usando la
estructura llamada `ImportantExcerpt` que definimos en el Listado 10-25.

Primero, usaremos un método llamado `level` cuyo único parámetro es una
referencia a `self` y cuyo valor de retorno es `i32`, que no es una
referencia a nada:

```rust
# struct ImportantExcerpt<'a> {
#     part: &'a str,
# }
#
impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

La declaración del parámetro de *lifetime* después de `impl` y su uso
después del nombre del tipo es obligatoria, pero no es necesario que
anotemos el *lifetime* de la referencia a `self` debido a la primera regla
de *elisión*.

Aquí hay un ejemplo donde se aplica la tercera regla de *elisión* de *lifetime*:

```rust
# struct ImportantExcerpt<'a> {
#     part: &'a str,
# }
#
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part(&self, announcement: &str) -> &str {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```

Hay dos *input lifetimes*, por lo que Rust aplica la primera regla de
*elisión lifetime* y otorga a ambos `&self` y al `announcement` su propia
*lifetimes*. Entonces, debido a que uno de los parámetros es `&self`, el
tipo de retorno obtiene el *lifetime* de `&self`, y todos los *lifetimes*
han sido contabilizados.

### La *Lifetime* estático

Una *lifetime* especial que debemos analizar es `'estática`, que denota la
duración total del programa. Todos los literales de string tienen la
duración de *lifetime* `'static`, que podemos anotar de la siguiente manera:

```rust
let s: &'static str = "I have a static lifetime.";
```

El texto de este *string* se almacena directamente en el binario de tu
programa, que siempre está disponible. Por lo tanto, el *lifetime* de todos
los literales de cadena es `'static`.

Es posible que vea sugerencias para utilizar el *lifetime* `'static` en los
mensajes de error. Pero antes de especificar `'static` como el *lifetime* de
una referencia, piense si la referencia en realidad la ha vivido durante
toda la vida de su programa o no. Puede considerar si desea que viva tanto
tiempo, incluso si pudiera. La mayoría de las veces, el problema es el
resultado de intentar crear una referencia colgante o un desajuste de los
*lifetimes* disponibles. En tales casos, la solución es solucionar esos
problemas, sin especificar el *lifetime* `'static`.

## Parámetros genéricos de tipo, *Trait Bounds* y *Lifetime* juntos

¡Veamos brevemente la sintaxis de especificar los parámetros de tipo
genérico, los *trait bounds* y los *lifetimes*, todo en una
función!

```rust
use std::fmt::Display;

fn longest_with_an_announcement<'a, T>(x: &'a str, y: &'a str, ann: T) -> &'a str
    where T: Display
{
    println!("Announcement! {}", ann);
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

Esta es la función `longest` del Listado 10-22 que devuelve el mayor de dos
*string slices*. Pero ahora tiene un parámetro extra llamado `ann` del tipo
genérico `T`, que puede rellenarse por cualquier tipo que implemente el
rasgo `Display` como se especifica en la cláusula `where`. Este parámetro
adicional se imprimirá antes de que la función compare las longitudes de los
*string slices*, por lo que es necesario el *trait bound* `Display`. Como
los *lifetime* son un tipo de genérico, las declaraciones del parámetro de
*lifetime* `'a` y el parámetro de tipo genérico `T` van en la misma lista
dentro de los corchetes angulares después del nombre de la función.

## Resumen

¡Cubrimos mucho en este capítulo! Ahora que sabe acerca de los parámetros de
tipo genérico, los *traits* los *trait bounds*, y los parámetros genéricos
de *lifetime*, está listo para escribir código sin repetición que funcione
en muchas situaciones diferentes. Los parámetros genéricos de tipo le
permiten aplicar el código a diferentes tipos. Los *trait* y los *trait
bounds* aseguran que, aunque los tipos son genéricos, tendrán el
comportamiento que el código necesita. Aprendió a usar anotaciones de por
vida para asegurarse de que este código flexible no tenga referencias
colgantes. ¡Y todo este análisis ocurre en tiempo de compilación, lo que no
afecta el rendimiento del tiempo de ejecución!

Créalo o no, hay mucho más que aprender sobre los temas que estudiamos en
este capítulo: el Capítulo 17 muestra los objetos de *trait*, que son otra
forma de usar los *traits*. El Capítulo 19 cubre escenarios más complejos
que implican anotaciones de *lifetime*, así como algunas características
avanzadas del sistema de tipo. Pero a continuación, aprenderá cómo escribir
pruebas en Rust para asegurarse de que su código funcione como debería.
