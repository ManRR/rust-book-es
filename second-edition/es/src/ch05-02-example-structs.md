## Un programa de ejemplo que usa estructuras

Para entender cuándo podríamos querer usar estructuras, escribamos un programa
que calcule el área de un rectángulo. Comenzaremos con variables únicas y luego
refactorizaremos el programa hasta que usemos structs.

Hagamos un nuevo proyecto binario con Cargo llamado *rectangles*  que tomará el
ancho y la altura de un rectángulo especificado en píxeles y calculará el área del
rectángulo. El listado 5-8 muestra un programa corto con una forma de hacer exactamente
eso en *src/main.rs* de nuestro proyecto.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let width1 = 30;
    let height1 = 50;

    println!(
        "El área del rectángulo es {} píxeles cuadrados.",
        area(width1, height1)
    );
}

fn area(width: u32, height: u32) -> u32 {
    width * height
}
```

<span class="caption">Listing 5-8: Cálculo del área de un rectángulo
especificado por variables de ancho y altura separadas</span>

Ahora, ejecute este programa usando `cargo run`:

```text
El área del rectángulo es 1500 píxeles cuadrados.
```

Aunque el listado 5-8 funciona y calcula el área del rectángulo llamando a
la función `area` con cada dimensión, podemos hacerlo mejor. El ancho y la altura
están relacionados entre sí porque juntos describen un rectángulo.

El problema con este código es evidente en la firma de `area`:

```rust,ignore
fn area(width: u32, height: u32) -> u32 {
```

La función `area` se supone que calcula el área de un rectángulo, pero la función
que escribimos tiene dos parámetros. Los parámetros están relacionados, pero eso
no se expresa en ningún lugar de nuestro programa. Sería más legible y más manejable
para agrupar ancho y alto juntos. Ya hemos discutido una forma en que podríamos hacer
eso en la sección “El tipo *Tuple*” del Capítulo 3: mediante el uso de tuplas.

### Refactorización con Tuplas

Listing 5-9  muestra otra versión de nuestro programa que usa tuplas.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let rect1 = (30, 50);

    println!(
        "El área del rectángulo es {} píxeles cuadrados.",
        area(rect1)
    );
}

fn area(dimensions: (u32, u32)) -> u32 {
    dimensions.0 * dimensions.1
}
```

<span class="caption">Listing 5-9: Especificando el ancho y alto del rectángulo
con una tupla</span>

De una manera, este programa es mejor. *Tuples* nos permite agregar un poco de estructura,
y ahora estamos pasando solo un argumento. Pero de otra manera, esta versión es
menos clara: las tuplas no nombran sus elementos, por lo que nuestro cálculo se ha vuelto
más confuso porque tenemos que indexar las partes de la tupla.

No importa si mezclamos ancho y alto para el cálculo del área, pero si queremos dibujar
el rectángulo en la pantalla, ¡importaría! Tendríamos que tener en cuenta que `ancho` es
el índice de tupla `0` y `altura` es el índice de tupla `1`. Si alguien más trabajó en
este código, tendrían que resolverlo y tenerlo en cuenta también. Sería fácil olvidar o
mezclar estos valores y causar errores, porque no hemos transmitido el significado de
nuestros datos en nuestro código.

### Refactorización con estructuras: agregar más significado

Usamos estructuras para agregar significado al etiquetar los datos. Podemos
transformar la tupla que estamos utilizando en un tipo de datos con un nombre
para el conjunto, así como los nombres de las partes, como se muestra en el
Listado 5-10.

<span class="filename">Filename: src/main.rs</span>

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "El área del rectángulo es {} píxeles cuadrados.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

<span class="caption">Listing 5-10: Definiendo una estructura `Rectangle`</span>

Aquí hemos definido una estructura y la hemos llamado `Rectangle`. Dentro de las llaves,
definimos los campos como `ancho` y `alto`, ambos tienen el tipo `u32`. Luego, en `main`,
creamos una instancia particular de `Rectangle` que tiene un ancho de 30 y una altura de 50.

Nuestra función `area` ahora se define con un parámetro, que hemos llamado `rectangle`,
cuyo tipo es un préstamo inmutable de una instancia struct `Rectangle`. Como se mencionó en
el Capítulo 4, queremos tomar prestada la estructura en lugar de tomar posesión de ella.
De esta manera, `main` conserva su propiedad y puede continuar usando `rect1`, que es la
razón por la que usamos `&` en la firma de la función y donde llamamos a la función.

La función `area` accede a los campos `width` y `height` de la instancia `Rectangle`.
Nuestra firma de funciones para `area` ahora dice exactamente lo que queremos decir:
calcule el área de `Rectangle`, usando sus campos `width` y `height`. Esto transmite
que el ancho y la altura están relacionados entre sí, y da nombres descriptivos a los
valores en lugar de usar los valores de índice de tupla de `0` y `1`. Esto es una
victoria para la claridad.

### Añadiendo funcionalidad útil con *traits* derivados

Sería bueno poder imprimir una instancia de `Rectangle` mientras depuramos
nuestro programa y vemos los valores para todos sus campos. El listado 5-11
intenta usar la macro `println!` Como hemos usado en capítulos anteriores.
Esto no funcionará, sin embargo.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {}", rect1);
}
```

<span class="caption">Listing 5-11: Intentando imprimir una instancia de `Rectangle`</span>

Cuando ejecutamos este código, obtenemos un error con este mensaje central:

```text
error[E0277]: the trait bound `Rectangle: std::fmt::Display` is not satisfied
```

La macro `println!` Puede hacer muchos tipos de formateo, y de forma predeterminada,
las llaves indican `println!` Para usar el formato conocido como `Display`:
salida destinada para el consumo directo del usuario final.
Los tipos primitivos que hemos visto hasta ahora implementan `Display` de
forma predeterminada, ya que solo hay una forma en la que le gustaría mostrar un `1`
o cualquier otro tipo primitivo a un usuario. Pero con structs, la forma en que `println!`
Debe formatear el resultado es menos clara porque hay más posibilidades de visualización:
¿Quieres comas o no? ¿Quieres imprimir las llaves? ¿Deberían mostrarse todos los campos?
Debido a esta ambigüedad, Rust no intenta adivinar lo que queremos, y las estructuras no tienen
una implementación provista de `Display`.

Si continuamos leyendo los errores, encontraremos esta nota útil:

```text
`Rectangle` cannot be formatted with the default formatter; try using
`:?` instead if you are using a format string
```

¡Vamos a intentarlo! La llamada a la macro `println!` Se verá como
`println (" rect1 es {:?} ", Rect1);`. Poner el especificador `:?`
Dentro de las llaves queremos usar un formato de salida llamado `Debug`.
El *trait* `Debug` nos permite imprimir nuestra estructura de una
manera que es útil para los desarrolladores para que podamos ver su
valor mientras depuramos nuestro código.

Ejecuta el código con este cambio. ¡Arrg! Todavía recibimos un error:

```text
error[E0277]: the trait bound `Rectangle: std::fmt::Debug` is not satisfied
```

Pero, de nuevo, el compilador nos da una nota útil:

```text
`Rectangle` cannot be formatted using `:?`; if it is defined in your
crate, add `#[derive(Debug)]` or manually implement it
```

Rust *no* incluye funcionalidad para imprimir información de depuración,
pero tenemos que optar explícitamente para que esa funcionalidad esté disponible
para nuestra estructura. Para hacer eso, agregamos la anotación `#[derive(Debug)]`
justo antes de la definición de la estructura, como se muestra en el Listado 5-12.

<span class="filename">Filename: src/main.rs</span>

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!("rect1 is {:?}", rect1);
}
```

<span class="caption">Listing 5-12: Añadiendo la anotación para derivar el *trait* `Debug`
e imprimir la instancia `Rectangle` utilizando el formato de depuración</span>

Ahora cuando ejecutamos el programa, no obtendremos ningún error, y veremos el siguiente resultado:

```text
rect1 is Rectangle { width: 30, height: 50 }
```

¡Bien!, no es el resultado más bonito, pero muestra los valores de todos los
campos para esta instancia, lo que definitivamente ayudaría durante la depuración.
Cuando tenemos estructuras más grandes, es útil tener una salida que sea un poco
más fácil de leer; en esos casos, podemos usar `{:#?}` en lugar de `{:?}` en el
*string* `println!`. Cuando usamos el estilo `{:#?}` En el ejemplo, la salida se verá así:


```text
rect1 is Rectangle {
    width: 30,
    height: 50
}
```

Rust nos ha proporcionado varios *traits* que podemos usar con la anotación `derivar '
que puede agregar un comportamiento útil a nuestros tipos personalizados. Esos *traits*
y sus comportamientos se enumeran en el Apéndice C. Cubriremos cómo implementar estos
*traits* con un comportamiento personalizado, así como también cómo crear sus propios
*traits* en el Capítulo 10.

Nuestra función `area` es muy específica: solo calcula el área de rectángulos. Sería
útil vincular este comportamiento más de cerca con nuestra estructura `Rectangle`, ya
que no funcionará con ningún otro tipo. Veamos cómo podemos continuar refactorizando
este código convirtiendo la función `area` en un *método* definido en nuestro tipo
`Rectangle`.