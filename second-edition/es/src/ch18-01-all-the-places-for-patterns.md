## All the Places Patterns Can Be Used

¡Los patrones aparecen en varios lugares en Rust, y los has estado usando
mucho sin darte cuenta! Esta sección trata sobre todos los lugares donde los
patrones son válidos.

### `match` Brazos

Como se discutió en el Capítulo 6, usamos patrones en los brazos de las
expresiones `match`. Formalmente, las expresiones `match` se definen como la
palabra clave `match`, un valor para coincidir y uno o más brazos de
coincidencia que consisten en un patrón y una expresión para ejecutar si el
valor coincide con el patrón de ese brazo, como este:

```text
match VALUE {
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
    PATTERN => EXPRESSION,
}
```

Un requisito para las expresiones `match` es que deben ser *exhaustivas* en
el sentido de que todas las posibilidades para el valor en la expresión
`match` deben ser contabilizadas. Una forma de asegurarse de haber cubierto
todas las posibilidades es tener un patrón de catchall para el último brazo:
por ejemplo, un nombre de variable que coincida con cualquier valor nunca
puede fallar y, por lo tanto, cubre todos los casos restantes.

Un patrón particular `_` coincidirá con cualquier cosa, pero nunca se une a
una variable, por lo que a menudo se usa en el último brazo de coincidencia.
El patrón `_` puede ser útil cuando quiere ignorar cualquier valor no
especificado, por ejemplo. Cubriremos el patrón `_` con más detalle en la
sección" Ignorar valores en un patrón "más adelante en este capítulo.

### Expresiones condicional `if let`

En el Capítulo 6 discutimos cómo usar las expresiones `if let` principalmente
como una forma más corta de escribir el equivalente de un `match` que solo
coincide con un caso. Opcionalmente, `if let` puede tener un código que
contenga `else` correspondiente para ejecutar si el patrón en `if let` no
coincide.

El listado 18-1 muestra que también es posible mezclar y combinar las
expresiones `if let`, `else if`, y `else if let`. Hacerlo nos da más
flexibilidad que una expresión de `match` en la que podemos expresar un solo
valor para comparar con los patrones. Además, las condiciones en una serie de
brazos `if let`,`else if`, `else if let` no están obligados a relacionarse
entre sí.

El código en el listado 18-1 muestra una serie de comprobaciones para varias
condiciones que deciden cuál debe ser el color de fondo. Para este ejemplo,
hemos creado variables con valores codificados que un programa real podría
recibir de la entrada del usuario.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let favorite_color: Option<&str> = None;
    let is_tuesday = false;
    let age: Result<u8, _> = "34".parse();

    if let Some(color) = favorite_color {
        println!("Using your favorite color, {}, as the background", color);
    } else if is_tuesday {
        println!("Tuesday is green day!");
    } else if let Ok(age) = age {
        if age > 30 {
            println!("Using purple as the background color");
        } else {
            println!("Using orange as the background color");
        }
    } else {
        println!("Using blue as the background color");
    }
}
```

<span class="caption">Listado 18-1: Mezcla `if let`, `else if`, `else if let`,
y `else`</span>

Si el usuario especifica un color favorito, ese color es el color de fondo.
Si hoy es martes, el color de fondo es verde. Si el usuario especifica su
edad como un *string* y podemos analizarla como un número con éxito, el color
es púrpura o naranja dependiendo del valor del número. Si no se cumple
ninguna de estas condiciones, el color de fondo es azul.

Esta estructura condicional nos permite soportar requerimientos complejos.
Con los valores codificados que tenemos aquí, este ejemplo imprimirá
`Using purple as the background color`.

Puedes ver que `if let` también puede introducir variables sombreadas de la
misma manera que` match` arms can: la línea `if let Ok(age) = age` introduce
una nueva variable sombreada `age` que contiene el valor dentro del `Ok`
variante. Esto significa que debemos ubicar la condición `if age > 30` dentro
de ese bloque: no podemos combinar estas dos condiciones en
`if let Ok(age) = age && age > 30`. La `age` sombreada que queremos comparar con 30 no es válida hasta que el nuevo alcance comience con llaves.

La desventaja de usar expresiones `if let` es que el compilador no verifica
la exhaustividad, mientras que con las expresiones `match` sí lo hace. Si
omitimos el último bloque `else` y, por lo tanto, fallamos al manejar algunos
casos, el compilador no nos alertaría sobre la posible falla lógica.

### `while let` Bucles condicionales

Similar en la construcción a `if let`, el bucle condicional `while let`
permite que un bucle `while` se ejecute mientras el patrón continúe
coincidiendo. El ejemplo en el Listado 18-2 muestra un bucle `while let` que
usa un vector como una pila e imprime los valores en el vector en el orden
opuesto en el que fueron empujados.

```rust
let mut stack = Vec::new();

stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
    println!("{}", top);
}
```

<span class="caption">Listado 18-2: Usando un bucle `while let` para imprimir
valores mientras `stack.pop()` devuelva `Some`</span>

Este ejemplo imprime 3, 2 y luego 1. El método `pop` toma el último elemento
del vector y devuelve `Some(value)`. Si el vector está vacío, `pop` devuelve
`None`. El bucle `while` continúa ejecutando el código en su bloque siempre
que `pop` devuelva `Some`. Cuando `pop` devuelve `None`, el ciclo se detiene.
Podemos usar `while let` para hacer estallar cada elemento de nuestra pila.

### `for` Loops

En el Capítulo 3, mencionamos que el bucle `for` es la construcción de bucle
más común en el código Rust, pero aún no hemos discutido el patrón que `for`
toma. En un ciclo `for`, el patrón es el valor que sigue directamente a la
palabra clave `for`, por lo que en `for x in y` la `x` es el patrón.

El listado 18-3 muestra cómo usar un patrón en un ciclo `for` para
desestructurar o separar una tupla como parte del ciclo `for`.

```rust
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
```

<span class="caption">Listado 18-3: Usar un patrón en un ciclo `for` para
desestructurar una tupla</span>

El código en el Listado 18-3 imprimirá lo siguiente:

```text
a is at index 0
b is at index 1
c is at index 2
```

Usamos el método `enumerate` para adaptar un iterador para producir un valor
y el índice de ese valor en el iterador, colocado en una tupla. La primera
llamada a `enumerar` produce la tupla `(0, 'a')`. Cuando este valor se
corresponde con el patrón `(index, value)`, `index` será `0` y `value`
será `'a'`, imprimiendo la primera línea de la salida.

### `let` Statements

Antes de este capítulo, solo habíamos discutido explícitamente el uso de
patrones con `match` y `if let`, pero de hecho, también hemos usado patrones
en otros lugares, incluso en sentencias `let`. Por ejemplo, considere esta
asignación de variable directa con `let`:

```rust
let x = 5;
```

A lo largo de este libro, hemos usado `let` de esta manera cientos de veces,
y aunque no te hayas dado cuenta, ¡estabas usando patrones!. Más formalmente,
una declaración `let` se ve así:

```text
let PATTERN = EXPRESSION;
```

En declaraciones como `let x = 5;` con un nombre de variable en la ranura
`PATTERN`, el nombre de la variable es simplemente una forma particularmente
simple de un patrón. Rust compara la expresión con el patrón y asigna los
nombres que encuentra. Entonces en el ejemplo `let x = 5;`, `x` es un patrón
que significa “Vincula lo que coincide aquí con la variable `x`”. Debido a
que el nombre `x` es el patrón completo, este patrón significa efectivamente
“enlazar todo a la variable `x`, cualquiera que sea el valor”.

Para ver el aspecto de coincidencia de patrones de `let` con mayor claridad,
considere el Listado 18-4, que utiliza un patrón con `let` para
desestructurar una tupla.

```rust
let (x, y, z) = (1, 2, 3);
```

<span class="caption">Listado 18-4: Usar un patrón para desestructurar una
tupla y crear tres variables a la vez</span>

Aquí, hacemos coincidir una tupla con un patrón. Rust compara el valor
`(1, 2, 3)` con el patrón `(x, y, z)` y ve que el valor coincide con el
patrón, por lo que Rust vincula `1` a `x`, `2` a `y`, y `3` a` z`. Puedes
pensar en este patrón de tuplas como anidando tres patrones variables
individuales dentro de él.

Si el número de elementos en el patrón no coincide con el número de elementos
en la tupla, el tipo general no coincidirá y obtendremos un error de
compilación. Por ejemplo, el Listado 18-5 muestra un intento de
desestructurar una tupla con tres elementos en dos variables, que no
funcionarán.

```rust,ignore
let (x, y) = (1, 2, 3);
```

<span class="caption">Listado 18-5: Construcción incorrecta de un patrón
cuyas variables no coinciden con el número de elementos en la tupla</span>

Intentar compilar este código da como resultado este tipo de error:

```text
error[E0308]: mismatched types
 --> src/main.rs:2:9
  |
2 |     let (x, y) = (1, 2, 3);
  |         ^^^^^^ expected a tuple with 3 elements, found one with 2 elements
  |
  = note: expected type `({integer}, {integer}, {integer})`
             found type `(_, _)`
```

Si quisiéramos ignorar uno o más de los valores en la tupla, podríamos usar
`_` o `..`, como verá en la sección “Ignorar valores en un patrón”. Si el
problema es que tenemos demasiadas variables en el patrón, la solución es
hacer coincidir los tipos mediante la eliminación de variables para que el
número de variables sea igual al número de elementos en la tupla.

### Function Parameters

Los parámetros de función también pueden ser patrones. El código en el
Listado 18-6, que declara una función llamada `foo` que toma un parámetro
llamado `x` de tipo `i32`, ahora debería parecer familiar.

```rust
fn foo(x: i32) {
    // code goes here
}
```

<span class="caption">Listado 18-6: Una firma de función usa patrones en los
parámetros</span>

¡La parte `x` es un patrón! Como hicimos con `let`, podríamos hacer coincidir
una tupla en los argumentos de una función con el patrón. El listado 18-7
divide los valores en una tupla cuando lo pasamos a una función.

<span class="filename">Filename: src/main.rs</span>

```rust
fn print_coordinates(&(x, y): &(i32, i32)) {
    println!("Current location: ({}, {})", x, y);
}

fn main() {
    let point = (3, 5);
    print_coordinates(&point);
}
```

<span class="caption">Listado 18-7: Una función con parámetros que
desestructuran una tupla</span>

Este código imprime  `Current location: (3, 5)`. Los valores `&(3, 5)`
coinciden con el patrón `&(x, y)`, por lo que `x` es el valor `3` y `y` es el
valor `5`.

También podemos usar patrones en las listas de parámetros de *closure* de la
misma manera que en las listas de parámetros de función, porque los
*closures* son similares a las funciones, como se discutió en el Capítulo 13.

En este punto, ha visto varias formas de usar patrones, pero los patrones no
funcionan igual en todos los lugares donde podemos usarlos. En algunos
lugares, los patrones deben ser irrefutables; en otras circunstancias, pueden
ser refutables. Discutiremos estos dos conceptos a continuación.