## Sintaxis de patrones

A lo largo del libro, has visto ejemplos de muchos tipos de patrones. En esta
sección, reunimos toda la sintaxis válida en los patrones y discutimos por
qué es posible que desee usar cada uno.

### Matching Literals

Como viste en el Capítulo 6, puedes unir patrones contra literales
directamente. El siguiente código brinda algunos ejemplos:

```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

Este código imprime `one` porque el valor en `x` es 1. Esta sintaxis es útil
cuando desea que su código realice una acción si obtiene un valor concreto en
particular.

### Coincidencia de variables con nombre

Las variables con nombre son patrones irrefutables que coinciden con
cualquier valor, y los hemos utilizado muchas veces en el libro. Sin embargo,
hay una complicación cuando usa variables con nombre en expresiones `match`.
Debido a que `match` inicia un nuevo ámbito, las variables declaradas como
parte de un patrón dentro de la expresión `match` sombrearán aquellas con el
mismo nombre fuera de la construcción `match`, como es el caso de todas las
variables. En el listado 18-11, declaramos una variable llamada `x` con el
valor `Some(5)` y una variable `y` con el valor `10`. Luego creamos una
expresión `match` en el valor `x`. Mire los patrones en los brazos de partido
y `println!`. Al final, e intente descubrir qué se imprimirá el código antes
de ejecutar este código o leer más.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(y) => println!("Matched, y = {:?}", y),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {:?}", x, y);
}
```

<span class="caption">Listado 18-11: Una expresión `match` con un brazo que
introduce un *shadowed variable* `y`</span>

Veamos qué pasa cuando se ejecuta la expresión `match`. El patrón
en el primer brazo coincidente no coincide con el valor definido de `x`, por
lo que el código continúa. El patrón en el brazo del segundo *match*
introduce una nueva variable llamada `y` que coincidirá con cualquier valor
dentro de un valor `Some`. Porque estamos en un nuevo alcance dentro
la expresión `match`, esta es una nueva variable `y`, no la `y` que
declaramos en el comienzo con el valor 10. Este nuevo enlace `y` coincidirá
con cualquier valor dentro de `Some`, que es lo que tenemos en `x`. Por lo
tanto, este nuevo `y` se une a el valor interno de `Some` en `x`. Ese valor
es `5`, por lo que la expresión de ese brazo ejecuta e imprime
`Matched, y = 5`.

Si `x` ha sido un valor `None` en lugar de `Some(5)`, los patrones en el
primer dos brazos no habrían coincidido, por lo que el valor habría
coincidido con el guion bajo. No introdujimos la variable `x` en el patrón del
subrayar el brazo, por lo que la `x` en la expresión sigue siendo la `x`
externa que no ha sido *shadowed*. En este caso hipotético, el `match`
imprimiría `Default case, x = None`.

Cuando se completa la expresión `match`, su alcance finaliza, y también lo
hace el alcance del `y` interno. El último `println!` produce
`at the end: x = Some(5), y = 10`.

Para crear una expresión de `match` que compare los valores de las `x` y las `y` externas, en lugar de introducir una variable *shadowed*, tendríamos que
usar un *match guard conditional* en su lugar. Hablaremos sobre los protectores de partido más adelante en la sección “Condicionales adicionales con *Match Guards*”.

### Patrones múltiples

En las expresiones `match`, puede hacer coincidir varios patrones con la
sintaxis `|`, lo que significa *o*. Por ejemplo, el siguiente código coincide
con el valor de `x` contra los brazos del `match`, el primero de los cuales
tiene una opción *o*, lo que significa que si el valor de `x` coincide con
cualquiera de los valores en ese brazo, el código de ese brazo se ejecutará:

```rust
let x = 1;

match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

Este código imprime `uno o dos`.

### Coincidencia de rangos de valores con `...`

La sintaxis `...` nos permite coincidir con un rango de valores inclusivo. En
el siguiente código, cuando un patrón coincide con cualquiera de los valores
dentro del rango, ese brazo se ejecutará:

```rust
let x = 5;

match x {
    1 ... 5 => println!("one through five"),
    _ => println!("something else"),
}
```

Si `x` es 1, 2, 3, 4 o 5, el primer brazo coincidirá. Esta sintaxis es más
conveniente que usar el operador `|` para expresar la misma idea; en lugar
de `1 ... 5`, tendríamos que especificar `1 | 2 | 3 | 4 | 5` si usamos `|`.
Especificar un rango es mucho más corto, especialmente si queremos hacer
coincidir, por ejemplo, cualquier número entre 1 y 1.000.

Los rangos solo se permiten con valores numéricos o valores `char`, porque
el compilador verifica que el rango no esté vacío en tiempo de compilación.
Los únicos tipos que Rust puede decir si un rango está vacío o no son `char`
y valores numéricos.

Aquí hay un ejemplo que usa rangos de valores `char`:

```rust
let x = 'c';

match x {
    'a' ... 'j' => println!("early ASCII letter"),
    'k' ... 'z' => println!("late ASCII letter"),
    _ => println!("something else"),
}
```

Rust puede decir que `c` está dentro del rango del primer patrón e imprime
`early ASCII letter`.

### Destructuring to Break Apart Values

También podemos usar patrones para desestructurar estructuras, enumeraciones
tuplas y referencias para usar diferentes partes de estos valores. Veamos
cada valor.

#### Destructuring Structs

El listado 18-12 muestra una estructura `Point` con dos campos, `x` y `y`,
que podemos separar utilizando un patrón con una declaración `let`.

<span class="filename">Filename: src/main.rs</span>

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
}
```

<span class="caption">Listado 18-12: Desestructuración de los campos de una
estructura en variables separadas</span>

Este código crea las variables `a` y `b` que coinciden con los valores de
los campos `x` y `y` de la variable `p`. Este ejemplo muestra que los
nombres de las variables en el patrón no tienen que coincidir con los
nombres de campo de la estructura. Pero es común querer que los nombres de
las variables coincidan con los nombres de los campos para que sea más fácil
recordar qué variables provienen de qué campos.

Debido a que tener nombres de variables coinciden con los campos es común y
porque al escribir `let Point { x: x, y: y } = p;` contiene mucha
duplicación, hay una forma abreviada de patrones que coinciden con los
campos de estructura: solo necesita listar el nombre del campo de estructura
y las variables creadas a partir del patrón tendrán los mismos nombres. El
listado 18-13 muestra código que se comporta de la misma manera que el
código en el listado 18-12, pero las variables creadas en el patrón `let`
son `x` y `y` en lugar de `a` y `b`.

<span class="filename">Filename: src/main.rs</span>

```rust
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y } = p;
    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

<span class="caption">Listing 18-13: Destructuring struct fields using struct
field shorthand</span>

Este código crea las variables `x` y `y` que coinciden con los campos `x` y
`y` de la variable `p`. El resultado es que las variables `x` y `y`
contienen los valores de la estructura `p`.

También podemos desestructurar con valores literales como parte del patrón
*struct* en lugar de crear variables para todos los campos. Hacerlo nos
permite probar algunos de los campos para valores particulares al crear
variables para desestructurar los otros campos.

El listado 18-14 muestra una expresión de `match` que separa los valores de
`Point` en tres casos: puntos que se encuentran directamente en el eje `x`
(que es verdadero cuando `y = 0`), en el eje `y` (`x = 0`), o ninguno.

<span class="filename">Filename: src/main.rs</span>

```rust
# struct Point {
#     x: i32,
#     y: i32,
# }
#
fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        Point { x: 0, y } => println!("On the y axis at {}", y),
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
}
```

<span class="caption">Listado 18-14: *Destructuring* y *matching* valores
literales en un patrón</span>

El primer brazo coincidirá con cualquier punto que se encuentre en el eje
`x` al especificar que el campo `y` coincide si su valor coincide con el
literal `0`. El patrón aún crea una variable `x` que podemos usar en el
código para este brazo.

De forma similar, el segundo brazo coincide con cualquier punto del eje `y`
al especificar que el campo `x` coincide si su valor es `0` y crea una
variable `y` para el valor del campo `y`. El tercer brazo no especifica
ningún literal, por lo que coincide con cualquier otro `Point` y crea
variables para los campos `x` y `y`.

En este ejemplo, el valor `p` coincide con el segundo brazo en virtud de
`x` que contiene un 0, por lo que este código imprimirá `On the y axis at 7`.

#### Destructuring Enums

Hemos desestructurado las enumeraciones anteriormente en este libro, por
ejemplo, cuando desestructuramos `Option<i32>` en el Listado 6-5 en el
Capítulo 6. Un detalle que no hemos mencionado explícitamente es que el
patrón para desestructurar una enumeración debe corresponder a la forma en
que se definen los datos almacenados dentro de la enumeración. Como ejemplo,
en el Listado 18-15 utilizamos la enumeración `Message` del Listado 6-2 y
escribimos un `match` con patrones que desestructurarán cada valor interno.

<span class="filename">Filename: src/main.rs</span>

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            println!("The Quit variant has no data to destructure.")
        },
        Message::Move { x, y } => {
            println!(
                "Move in the x direction {} and in the y direction {}",
                x,
                y
            );
        }
        Message::Write(text) => println!("Text message: {}", text),
        Message::ChangeColor(r, g, b) => {
            println!(
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
    }
}
```

<span class="caption">Listado 18-15: *Destructuring enum* variantes que contienen diferentes tipos de valores</span>

Este código imprimirá `Change the color to red 0, green 160, and blue 255`.
Intente cambiar el valor de `msg` y ejecutarlo para ver el código de los
otros brazos.

Para las variantes *enum* sin ningún dato, como `Message::Quit`, no podemos
desestructurar el valor más. Solo podemos hacer coincidir el valor literal
`Message::Quit`, y no hay variables en ese patrón.

Para las variantes de enumeración tipo *struct*, como `Message::Move`,
podemos usar un patrón similar al patrón que especificamos para que coincida
con las estructuras. Después del nombre de la variante, colocamos llaves
y luego enumeramos los campos con variables para separar las piezas y
usarlas en el código para este brazo. Aquí usamos la forma abreviada como lo
hicimos en el listado 18-13.

Para las variantes *enum* tipo tupla, como `Message::Write` que contiene una
tupla con un elemento y `Message::ChangeColor` que contiene una tupla con
tres elementos, el patrón es similar al patrón que especificamos para hacer
coincidir tuplas. El número de variables en el patrón debe coincidir con la
cantidad de elementos en la variante que estamos combinando.

#### Destructuring References

Cuando el valor que estamos combinando con nuestro patrón contiene una
referencia, necesitamos desestructurar la referencia del valor, lo cual
podemos hacer especificando un `&` en el patrón. Hacerlo nos permite obtener
una variable que contenga el valor al que apunta la referencia en lugar de
obtener una variable que contenga la referencia. Esta técnica es
especialmente útil en *closures* donde tenemos iteradores que iteran sobre
referencias, pero queremos usar los valores en el *closure* en lugar de las
referencias.

El ejemplo en el listado 18-16 itera sobre referencias a instancias `Point`
en un vector, desestructurando la referencia y la estructura para que
podamos realizar cálculos sobre los valores `x` y `y` fácilmente.

```rust
# struct Point {
#     x: i32,
#     y: i32,
# }
#
let points = vec![
    Point { x: 0, y: 0 },
    Point { x: 1, y: 5 },
    Point { x: 10, y: -3 },
];

let sum_of_squares: i32 = points
    .iter()
    .map(|&Point { x, y }| x * x + y * y)
    .sum();
```

<span class="caption">Listado 18-16: *Destructuring* una referencia a una
estructura en los valores del campo *struct*</span>

Este código nos da la variable `sum_of_squares` que contiene el valor 135,
que es el resultado de cuadrar el valor `x` y el valor `y`, sumarlos y luego
sumar el resultado de cada `Point` en los `points` vector para obtener un
número.

Si no hubiésemos incluido `&` en `&Punto { x, y }`, obtendríamos un error de
coincidencia de tipo, porque `iter` repetiría las referencias a los
elementos en el vector en lugar de los valores reales. El error sería así:

```text
error[E0308]: mismatched types
  -->
   |
14 |         .map(|Point { x, y }| x * x + y * y)
   |               ^^^^^^^^^^^^ expected &Point, found struct `Point`
   |
   = note: expected type `&Point`
              found type `Point`
```

Este error indica que Rust esperaba que nuestro *closure* coincidiera con
`&Point`, pero tratamos de hacer coincidir directamente con un valor de
`Point`, no con una referencia a un `Point`.

#### Destructuring Structs and Tuples

Podemos mezclar, unir y jerarquizar patrones de desestructuración de formas
aún más complejas. El siguiente ejemplo muestra una desestructuración
complicada donde anidamos estructuras y tuplas dentro de una tupla y
desestructuramos todos los valores primitivos:

```rust
# struct Point {
#     x: i32,
#     y: i32,
# }
#
let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });
```

Este código nos permite dividir los tipos complejos en sus componentes para
que podamos usar los valores que nos interesan por separado.

La desestructuración con patrones es una forma conveniente de usar
fragmentos de valores, como el valor de cada campo en una estructura,
separadamente el uno del otro.

### Ignorar valores en un patrón

Has visto que a veces es útil ignorar los valores en un patrón, como en el
último brazo de una `match`, para obtener una captura que en realidad no
hace nada, pero que da cuenta de todos los valores posibles restantes. Hay
algunas maneras de ignorar valores enteros o partes de valores en un patrón:
usando el patrón `_` (que has visto), usando el patrón `_` dentro de otro
patrón, usando un nombre que comienza con un guión bajo, o usando `..` para
ignorar las partes restantes de un valor. Exploremos cómo y por qué usar
cada uno de estos patrones.

#### Ignorar un valor completo con `_`

Hemos utilizado el guión bajo (`_`) como un patrón comodín que coincidirá
con cualquier valor pero no se vinculará al valor. Aunque el patrón de
subrayado `_` es especialmente útil como el último brazo en una expresión de
`match`, podemos usarlo en cualquier patrón, incluidos los parámetros de
función, como se muestra en el listado 18-17.

<span class="filename">Filename: src/main.rs</span>

```rust
fn foo(_: i32, y: i32) {
    println!("This code only uses the y parameter: {}", y);
}

fn main() {
    foo(3, 4);
}
```

<span class="caption">Listado 18-17: Usando `_` en una firma de
función</span>

Este código ignorará por completo el valor pasado como primer argumento, `3`
e imprimirá `This code only uses the y parameter: 4`.

En la mayoría de los casos, cuando ya no necesita un parámetro de función en
particular, debe cambiar la firma para que no incluya el parámetro no
utilizado. Ignorar un parámetro de función puede ser especialmente útil en
algunos casos, por ejemplo, cuando se implementa un *trait* cuando se
necesita una determinada firma de tipo, pero el cuerpo de la función en su
implementación no necesita uno de los parámetros. El compilador no advertirá
sobre los parámetros de la función no utilizados, como lo haría si usara un
nombre en su lugar.

#### Ignorar partes de un valor con un anidado `_`

También podemos usar `_` dentro de otro patrón para ignorar solo parte de un
valor, por ejemplo, cuando queremos probar solo una parte de un valor, pero
no tenemos uso para las otras partes en el código correspondiente que
queremos ejecutar. El listado 18-18 muestra el código responsable de
administrar el valor de una configuración. Los requisitos comerciales son
que no se debe permitir que el usuario sobrescriba una personalización
existente de una configuración, pero puede deshacer la configuración y puede
darle un valor a la configuración si está actualmente desarmada.

```rust
let mut setting_value = Some(5);
let new_setting_value = Some(10);

match (setting_value, new_setting_value) {
    (Some(_), Some(_)) => {
        println!("Can't overwrite an existing customized value");
    }
    _ => {
        setting_value = new_setting_value;
    }
}

println!("setting is {:?}", setting_value);
```

<span class="caption">Listado 18-18: Utilizando un guión bajo dentro de
patrones que coinciden con las variantes `Some` cuando no necesitamos usar
el valor dentro de `Some`</span>

Este código imprimirá `Can't overwrite an existing customized value` y
luego `setting is Some(5)`. En el primer brazo de coincidencia, no es
necesario que coincidamos ni utilicemos los valores dentro de la variante
`Some`, pero sí debemos probar el caso cuando `setting_value` y
`new_setting_value` son la variante `Some`. En ese caso, imprimimos por qué
no estamos cambiando `setting_value`, y no se cambia.

En todos los demás casos (si `setting_value` o `new_setting_value` son
`None`) expresados por el patrón `_` en el segundo brazo, queremos permitir
que `new_setting_value` se convierta en `setting_value`.

También podemos usar guiones bajos en varios lugares dentro de un patrón
para ignorar valores particulares. El listado 18-19 muestra un ejemplo de
ignorar los valores segundo y cuarto en una tupla de cinco elementos.

```rust
let numbers = (2, 4, 8, 16, 32);

match numbers {
    (first, _, third, _, fifth) => {
        println!("Some numbers: {}, {}, {}", first, third, fifth)
    },
}
```

<span class="caption">Listado 18-19: Ignorar varias partes de una
tupla</span>

Este código imprimirá `Some numbers: 2, 8, 32`, y los valores 4 y 16 serán
ignorados.

#### Ignorar una variable no utilizada iniciando su nombre con `_`

Si creas una variable pero no la usas en ningún lado, Rust generalmente
emitirá una advertencia porque podría ser un error. Pero a veces es útil
crear una variable que no usará todavía, como cuando está creando prototipos
o simplemente iniciando un proyecto. En esta situación, puede decirle a Rust
que no le advierta sobre la variable no utilizada comenzando el nombre de la
variable con un guión bajo. En el listado 18-20, creamos dos variables sin
usar, pero cuando ejecutamos este código, solo deberíamos recibir una
advertencia sobre una de ellas.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let _x = 5;
    let y = 10;
}
```

<span class="caption">Listado 18-20: Iniciar un nombre de variable con un
guión bajo para evitar recibir advertencias de variables no utilizadas</span>

Aquí recibimos una advertencia acerca de no usar la variable `y`, pero no
recibimos una advertencia acerca de no usar la variable precedida por el
guión bajo.

Tenga en cuenta que hay una diferencia sutil entre usar solo `_` y usar un
nombre que comience con un guión bajo. La sintaxis `_x` todavía vincula el
valor a la variable, mientras que `_` no se une en absoluto. Para mostrar un
caso en el que importa esta distinción, el Listado 18-21 nos proporcionará
un error.

```rust,ignore
let s = Some(String::from("Hello!"));

if let Some(_s) = s {
    println!("found a string");
}

println!("{:?}", s);
```

<span class="caption">Listado 18-21: una variable no utilizada que comience
con un guión bajo aún vincula el valor, que podría tomar posesión del
valor</span>

Recibiremos un error porque el valor `s` se moverá a `_s`, lo que nos impide
usar `s` nuevamente. Sin embargo, usar el guión bajo por sí mismo nunca se
vincula al valor. El listado 18-22 se compilará sin ningún error porque `s`
no se mueve a `_`.

```rust
let s = Some(String::from("Hello!"));

if let Some(_) = s {
    println!("found a string");
}

println!("{:?}", s);
```

<span class="caption">Listado 18-22: El uso de un guión bajo no vincula el
valor</span>

Este código funciona bien porque nunca vinculamos `s` a nada; no se mueve.

#### Ignorar las partes restantes de un valor con `..`

Con valores que tienen muchas partes, podemos usar la sintaxis `..` para
usar solo algunas partes e ignorar el resto, evitando la necesidad de
enumerar guiones bajos para cada valor ignorado. El patrón `..` ignora
cualquier parte de un valor que no hayamos igualado explícitamente en el
resto del patrón. En el listado 18-23, tenemos una estructura `Point` que
contiene una coordenada en el espacio tridimensional. En la expresión
`match`, queremos operar solo en la coordenada `x` e ignorar los valores en
los campos `y` y `z`.

```rust
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}
```

<span class="caption">Listado 18-23: Ignorando todos los campos de un
`Point` a excepción de `x` usando `..`</span>

Enumeramos el valor `x` y luego solo incluimos el patrón `..`. Esto es más
rápido que tener que listar `y: _` y `z: _`, particularmente cuando estamos
trabajando con estructuras que tienen muchos campos en situaciones donde
solo uno o dos campos son relevantes.

La sintaxis `..` se ampliará a tantos valores como sea necesario. El listado
18-24 muestra cómo usar `..` con una tupla.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => {
            println!("Some numbers: {}, {}", first, last);
        },
    }
}
```

<span class="caption">Listado 18-24: Coincidencia solo del primer y último
valor en una tupla e ignorando todos los demás valores</span>

En este código, el primer y último valor se corresponden con `first` y
`last`. El `..` coincidirá e ignorará todo en el medio.

Sin embargo, usar `..` no debe ser ambiguo. Si no está claro qué valores
están destinados a la correspondencia y cuáles deben ignorarse, Rust nos
dará un error. El listado 18-25 muestra un ejemplo de uso de `..`
ambiguamente, por lo que no se compilará.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (.., second, ..) => {
            println!("Some numbers: {}", second)
        },
    }
}
```

<span class="caption">Listado 18-25: Un intento de usar `..` de una manera
ambigua</span>

Cuando compilamos este ejemplo, obtenemos este error:

```text
error: `..` can only be used once per tuple or tuple struct pattern
 --> src/main.rs:5:22
  |
5 |         (.., second, ..) => {
  |                      ^^
```

Es imposible para Rust determinar cuántos valores en la tupla ignorar antes
de hacer coincidir un valor con `second` y cuántos valores adicionales
ignorar después. Este código podría significar que queremos ignorar `2`,
enlazar `second` a `4`, y luego ignorar `8`, `16` y `32`; o que queremos
ignorar `2` y `4`, enlazar `second` a `8`, y luego ignorar `16` y `32`;
Etcétera. El nombre de la variable `second` no significa nada especial para
Rust, así que obtenemos un error de compilación porque el uso de `..` en dos
lugares como este es ambiguo.

### Crear referencias en patrones con `ref` y `ref mut`

Veamos el uso de `ref` para hacer referencias para que la propiedad de los
valores no se mueva a las variables en el patrón. Por lo general, cuando
coincide con un patrón, las variables introducidas por el patrón están
vinculadas a un valor. Las reglas de propiedad de Rust significan que el
valor se moverá al `match` o al lugar en el que esté utilizando el patrón.
El listado 18-26 muestra un ejemplo de un `match` que tiene un patrón con
una variable y luego el uso de todo el valor en la instrucción `println!`.
Más tarde, después del `match`. Este código no podrá compilarse porque la
propiedad de parte del valor `robot_name` se transfiere a la variable `name`
en el patrón del primer brazo `match`.

```rust,ignore
let robot_name = Some(String::from("Bors"));

match robot_name {
    Some(name) => println!("Found a name: {}", name),
    None => (),
}

println!("robot_name is: {:?}", robot_name);
```

<span class="caption">Listado 18-26: Crear una variable en un patrón de
brazo `match` toma posesión del valor</span>

Como la propiedad de parte de `robot_name` se ha movido a `name`, ya no
podemos usar `robot_name` en `println!` después de la `coincidencia` porque
`robot_name` ya no tiene propiedad.

Para corregir este código, queremos hacer que el patrón `Some (name)`
*tome prestado* esa parte de `robot_name` en lugar de tomar posesión. Ya has
visto que, fuera de los patrones, la forma de tomar prestado un valor es
crear una referencia usando `&`, por lo que podrías pensar que la solución
está cambiando `Some(name)` a `Some(&name)`.

Sin embargo, como viste en la sección “Destructuring to Break Apart Values”,
la sintaxis `&` en patrones no *crea* una referencia sino *coincide* con una
referencia existente en el valor. Debido a que `y` ya tiene ese significado
en los patrones, no podemos usar `&` para crear una referencia en un patrón.

En cambio, para crear una referencia en un patrón, usamos la palabra clave
`ref` antes de la nueva variable, como se muestra en el Listado 18-27.

```rust
let robot_name = Some(String::from("Bors"));

match robot_name {
    Some(ref name) => println!("Found a name: {}", name),
    None => (),
}

println!("robot_name is: {:?}", robot_name);
```

<span class="caption">Listado 18-27: Crear una referencia para que una
variable de patrón no tome posesión de un valor</span>

Este ejemplo se compilará porque el valor en la variante `Some` en
`robot_name` no se mueve al `match`; el `match` solo tomó una referencia a
los datos en `robot_name` en lugar de moverlo.

Para crear una referencia mutable para que podamos mutar un valor
coincidente en un patrón, usamos `ref mut` en lugar de `&mut`. La razón es,
una vez más, que en los patrones, el último es para hacer coincidir las
referencias mutables existentes, no crear nuevas. El listado 18-28 muestra
un ejemplo de un patrón que crea una referencia mutable.

```rust
let mut robot_name = Some(String::from("Bors"));

match robot_name {
    Some(ref mut name) => *name = String::from("Another name"),
    None => (),
}

println!("robot_name is: {:?}", robot_name);
```

<span class="caption">Listado 18-28: Crear una referencia mutable a un valor
como parte de un patrón usando `ref mut`</span>

Este ejemplo compilará e imprimirá `robot_name is: Some("Another name")`.
Debido a que `name` es una referencia mutable, tenemos que eliminar la
referencia dentro del código del brazo de coincidencia usando el operador
`*` para mutar el valor.

### Condicionales adicionales con *Match Guards*

Un *match guard* es una condición adicional `if` especificada después del
patrón en un brazo `match` que también debe coincidir, junto con la
coincidencia de patrón, para que se elija ese brazo. Los *match guard* son útiles para expresar ideas más complejas de lo que permite un
patrón solo.

La condición puede usar variables creadas en el patrón. El listado 18-29
muestra un `match` donde el primer brazo tiene el patrón `Some(x)` y también
tiene un guarda partido de `if x < 5`.

```rust
let num = Some(4);

match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}
```

<span class="caption">Listado 18-29: Agregar un *match guard* a un
patrón</span>

Este ejemplo imprimirá `less than five: 4`. Cuando `num` se compara con el
patrón en el primer brazo, coincide, porque `Some(4)` coincide con
`Some(x)`. Luego, el *match guard* verifica si el valor en `x` es menor que
`5`, y como lo es, se selecciona el primer brazo.

Si `num` hubiera sido `Some(10)`en su lugar, el *match guard* en el primer
brazo habría sido falso porque 10 no es menor que 5. Rust luego iría al
segundo brazo, que coincidiría porque el segundo brazo no tiene un
*match guard* y por lo tanto coincide con cualquier variante de `Some`.

No hay forma de expresar la condición `if x < 5`dentro de un patrón, por lo
que el *match guard* nos da la capacidad de expresar esta lógica.

En el Listado 18-11, mencionamos que podríamos usar *match guard* para
resolver nuestro problema de *shadowing* de patrones. Recuerde que se creó
una nueva variable dentro del patrón en la expresión `match` en lugar de
usar la variable fuera del `match`. Esa nueva variable significaba que no
podíamos comparar con el valor de la variable externa. El listado 18-30
muestra cómo podemos usar un *match guard* para solucionar este problema.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {:?}", n),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {:?}", x, y);
}
```

<span class="caption">Listado 18-30: Uso de un *match guard* para
probar la igualdad con una variable externa</span>

Este código ahora imprimirá `Default case, x = Some(5)`. El patrón en el
brazo del segundo *match* no introduce una nueva variable `y` que *shadow*
el` y` externo, lo que significa que podemos usar el `y` externo en el
*match guard*. En lugar de especificar el patrón como `Some(y)`, que habría
*shadowed* el `y` externo, especificamos `Some(n)`. Esto crea una nueva
variable `n` que no *shadow* nada porque no hay una variable `n` fuera del
`match`.

La función *match guard* `if n == y` no es un patrón y, por lo tanto, no
introduce nuevas variables. Este `y` *es* el `y` externo en vez de un nuevo
`y` *shadowed*, y podemos buscar un valor que tenga el mismo valor que el
`y` externo al comparar `n` con `y`.

También puede usar el operador *o* `|` en un *match guard* para especificar
múltiples patrones; la condición de *match guard* se aplicará a todos los
patrones. El listado 18-31 muestra la prioridad de combinar un *match guard*
con un patrón que usa `|`. La parte importante de este ejemplo es que el
*match guard* `if y` se aplica a `4`, `5`, *y* `6`, aunque podría parecer
`if y` solo se aplica a `6`.

```rust
let x = 4;
let y = false;

match x {
    4 | 5 | 6 if y => println!("yes"),
    _ => println!("no"),
}
```

<span class="caption">Listado 18-31: Combinando múltiples patrones con un
*match guard*</span>

La condición de coincidencia establece que el brazo solo coincide si el
valor de `x` es igual a `4`, `5`, o `6` *y* si `y` es `true`. Cuando se
ejecuta este código, el patrón del primer brazo coincide porque `x` es `4`,
pero el *match guard* `if y` es falso, por lo que no se elige el primer
brazo. El código pasa al segundo brazo, que no coincide, y este programa
imprime `no`. La razón es que la condición `if` se aplica a todo el patrón
`4 | 5 | 6`, no solo al último valor `6`. En otras palabras, la precedencia
de un *match guard* en relación con un patrón se comporta así:

```text
(4 | 5 | 6) if y => ...
```

rather than this:

```text
4 | 5 | (6 if y) => ...
```

Después de ejecutar el código, el comportamiento de precedencia es evidente:
si el *match guard* se aplicara solo al valor final en la lista de valores
especificada usando el operador `|`, el brazo se habría igualado y el
programa habría impreso `yes`.

### `@` Bindings

El operador *arroba* (`@`) nos permite crear una variable que contiene un
valor al mismo tiempo que probamos ese valor para ver si coincide con un
patrón. El listado 18-32 muestra un ejemplo donde queremos probar que un
campo `Message::Hello` `id` está dentro del rango `3 ... 7`. Pero también
queremos vincular el valor a la variable `id_variable` para que podamos
usarlo en el código asociado con el brazo. Podríamos nombrar esta variable
`id`, lo mismo que el campo, pero para este ejemplo utilizaremos un nombre
diferente.

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3...7 } => {
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10...12 } => {
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        println!("Found some other id: {}", id)
    },
}
```

<span class="caption">Listado 18-32: Usando `@` para enlazar a un valor en
un patrón mientras también lo prueba</span>

Este ejemplo imprimirá `Found an id in range: 5`. Al especificar
`id_variable @` antes del rango `3 ... 7`, estamos capturando cualquier
valor que coincida con el rango, al mismo tiempo que probamos que el valor
coincide con el *patrón de rango* (*range pattern*).

En el segundo brazo, donde solo tenemos un rango especificado en el patrón,
el código asociado con el brazo no tiene una variable que contenga el valor
real del campo `id`. El valor del campo `id` podría haber sido 10, 11 o 12,
pero el código que acompaña a ese patrón no sabe cuál es. El código de
patrón no puede usar el valor del campo `id`, porque no hemos guardado el
valor `id` en una variable.

En el último brazo, donde hemos especificado una variable sin un rango,
tenemos el valor disponible para usar en el código del brazo en una variable
llamada `id`. La razón es que hemos utilizado la sintaxis abreviada del
campo *struct*. Pero no hemos aplicado ninguna prueba al valor en el campo
`id` en este brazo, como hicimos con los primeros dos brazos: cualquier
valor coincidiría con este patrón.

Usar `@` nos permite probar un valor y guardarlo en una variable dentro de
un patrón.

## Resumen

Los patrones de Rust son muy útiles ya que ayudan a distinguir entre
diferentes tipos de datos. Cuando se usa en expresiones `match`, Rust
asegura que sus patrones cubren todos los valores posibles, o su programa no
compilará. Los patrones en las declaraciones `let` y los parámetros de
función hacen que esos constructos sean más útiles, lo que permite la
desestructuración de los valores en partes más pequeñas al mismo tiempo que
la asignación a las variables. Podemos crear patrones simples o complejos
para satisfacer nuestras necesidades.

Luego, para el penúltimo capítulo del libro, veremos algunos aspectos
avanzados de una variedad de características de Rust.
