## Sintaxis del método

*Los métodos* son similares a las funciones: se declaran con la palabra clave `fn` y su nombre,
pueden tener parámetros y un valor de retorno, y contienen algún código que se ejecuta cuando
se llaman desde otro lugar. Sin embargo, los métodos son diferentes de las funciones en que están
definidos dentro del contexto de una estructura (o un objeto enum o un *trait*, que cubrimos en los
Capítulos 6 y 17, respectivamente), y su primer parámetro es siempre `self`, que representa la
instancia de la estructura a la que se llama el método.

### Definición de métodos

Cambiemos la función `area` que tiene una instancia `Rectangle` como parámetro y,
en su lugar, hagamos que un método `area` se defina en la estructura `Rectangle`,
como se muestra en el Listado 5-13.

<span class="filename">Filename: src/main.rs</span>

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };

    println!(
        "El área del rectángulo es {} píxeles cuadrados.",
        rect1.area()
    );
}
```

<span class="caption">Listing 5-13: Definir un método `area` en la estructura
`Rectangle`</span>

Para definir la función dentro del contexto de `Rectangle`, comenzamos un bloque `impl`
(implementación). Luego movemos la función `area` dentro de las llaves de `impl`
y cambiar el primer parámetro (y en este caso, solo) a ser `self` en la firma y
en todas partes dentro del cuerpo. En `main`, donde llamó a la función `area` y
pasó `rect1` como argumento, podemos en su lugar usar *method syntax* para llamar
al método `area` en nuestra instancia `Rectangle`.
La sintaxis del método va después de una instancia: agregamos un punto seguido por el método
nombre, paréntesis y cualquier argumento.

En la firma de `area`, usamos `&self` en lugar de `rectangle: &Rectangle`
porque Rust sabe que el tipo de `self` es `Rectangle` debido a que este método es
dentro del contexto `impl Rectangle`. Tenga en cuenta que todavía necesitamos usar el `&`
antes de `self`, tal como lo hicimos en `&Rectangle`. Los métodos pueden tomar posesión de
`self`, tomar `self` inmutablemente como lo hemos hecho aquí, o tomar `self` mutable,
al igual que cualquier otro parámetro.

Hemos elegido `&self` aquí por la misma razón que usamos `&Rectangle` en
versión de función: no queremos tomar posesión, y solo queremos leer el
datos en la estructura, no escribir en ella. Si quisiéramos cambiar la instancia que
llamamos al método como parte de lo que hace el método, usamos `&mut self`
como el primer parámetro. Tener un método que tome posesión del
instancia usando solo `self` ya que el primer parámetro es raro; esta técnica se usa generalmente
cuando el método se transforma `self` en otra cosa y quieres
para evitar que la persona que llama use la instancia original después de la transformación.

El principal beneficio de usar métodos en lugar de funciones, además de usar
sintaxis del método y no tener que repetir el tipo de `self` en cada método
firma, es para la organización. Hemos puesto todas las cosas que podemos hacer con una
instancia de un tipo en un bloque `impl` en lugar de hacer que los futuros usuarios de nuestro
código busquen las capacidades de `Rectangle` en varios lugares de la biblioteca
que proporcionamos.

> ### ¿Dónde está el operador `->`?
>
> En C y C ++, se utilizan dos operadores diferentes para los métodos de llamada: utiliza `.`
> si está llamando directamente a un método en el objeto y `-> ` si llama al método en un puntero al objeto
> y necesita para desreferenciar el puntero primero. En otras palabras, si `object` es un puntero,
> `object-> something()` es similar a `(* object).something () `.
>
> Rust no tiene un equivalente al operador `->`; en cambio, Rust tiene una característica llamada
> *automatic referencing and dereferencing* (*referencia automática y eliminación de referencias*).
> Los métodos de llamada son uno de los pocos
> lugares en Rust que tienen este comportamiento.
>
> Así es como funciona: cuando llamas a un método con `object.something ()`,
> Rust automáticamente agrega `&`, `&mut`, o `*`para que `object` coincida con
> la firma del método. En otras palabras, los siguientes son los mismos:
>
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> El primero parece mucho más limpio. Este comportamiento de referencia automática funciona porque
> los métodos tienen un receptor claro, el tipo de `self`. Dado el receptor y el nombre de un método,
> Rust puede determinar definitivamente si el método es la lectura (`& self`), la mutación (`&mut self`),
> o el consumo (`self`). El hecho de que Rust haga que los préstamos sean implícitos para los receptores
> de métodos es una gran parte de hacer que la propiedad sea ergonómica en la práctica.

### Métodos con más parámetros

Practiquemos el uso de métodos implementando un segundo método en la estructura `Rectangle`.
Esta vez, queremos una instancia de `Rectangle` para tomar otra instancia de `Rectangle` y
devolver `true` si el segundo `Rectangle` puede caber completamente dentro de `self`; de lo
contrario, debería devolver `false`. Es decir, queremos poder escribir el programa que se
muestra en el listado 5-14, una vez que hayamos definido el método `can_hold`.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let rect1 = Rectangle { width: 30, height: 50 };
    let rect2 = Rectangle { width: 10, height: 40 };
    let rect3 = Rectangle { width: 60, height: 45 };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

<span class="caption">Listing 5-14: Usando el método aún no escrito `can_hold`</span>

Y la salida esperada se vería como la siguiente, porque ambas dimensiones de `rect2`
son más pequeñas que las dimensiones de` rect1` pero `rect3` es más ancha que` rect1`:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Sabemos que queremos definir un método, por lo que estará dentro del bloque `impl Rectangle`.
El nombre del método será `can_hold`, y tomará un préstamo inmutable de otro `Rectangle` como parámetro.
Podemos decir cuál será el tipo de parámetro mirando el código que llama al método:
`rect1.can_hold(&rect2)` pasa en `&rect2`, que es un préstamo inmutable a `rect2`,
una instancia de `Rectangle`. Esto tiene sentido porque solo necesitamos leer `rect2`
(en lugar de escribir, lo que significaría que necesitaríamos un préstamo mutable),
y queremos que `main` retenga la propiedad de `rect2` para que podamos usarlo
nuevamente después de llamar el método `can_hold`. El valor de retorno de `can_hold`
será un booleano, y la implementación comprobará si el ancho y la altura de `self`
son mayores que el ancho y el alto del otro `Rectangle`, respectivamente.
Agreguemos el nuevo método `can_hold` al bloque `impl` del Listado 5-13,
que se muestra en el Listado 5-15.

<span class="filename">Filename: src/main.rs</span>

```rust
# #[derive(Debug)]
# struct Rectangle {
#     width: u32,
#     height: u32,
# }
#
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

<span class="caption">Listing 5-15: Implementando el método `can_hold` en `Rectangle`
que toma otra instancia `Rectangle` como parámetro</span>

Cuando ejecutamos este código con la función `main` en el listado 5-14,
obtendremos nuestro resultado deseado. Los métodos pueden tomar múltiples
parámetros que agregamos a la firma después del parámetro `self`, y esos parámetros
funcionan igual que los parámetros en las funciones.

### Funciones asociadas

Otra característica útil de los bloques `impl` es que podemos definir funciones
dentro de bloques `impl` que *no* tomen `self` como parámetro. Estas se llaman
*funciones asociadas* porque están asociadas con la estructura. Siguen siendo funciones,
no métodos, porque no tienen una instancia de la estructura con la que trabajar. Ya has
usado la función asociada `String::from`.

Las funciones asociadas se utilizan a menudo para los constructores que devolverán
una nueva instancia de la estructura. Por ejemplo, podríamos proporcionar una función
asociada que tuviera un parámetro de una dimensión y usarla como ancho y alto, lo que
facilitaría la creación de un cuadrado `Rectangle` en lugar de tener que especificar el
mismo valor dos veces:

<span class="filename">Filename: src/main.rs</span>

```rust
# #[derive(Debug)]
# struct Rectangle {
#     width: u32,
#     height: u32,
# }
#
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}
```

Para llamar a esta función asociada, usamos la sintaxis `::` con el nombre de la estructura;
`let sq = Rectangle::square(3);` es un ejemplo. Esta función tiene el espacio de nombres
asignado por la estructura: la sintaxis `::` se usa para las funciones asociadas y los espacios
de nombres creados por los módulos. Discutiremos los módulos en el Capítulo 7.

### Múltiples bloques `impl`

Cada estructura puede tener múltiples bloques `impl`. Por ejemplo, el
Listado 5-15 es equivalente al código que se muestra en el Listado 5-16,
que tiene cada método en su propio bloque `impl`.

```rust
# #[derive(Debug)]
# struct Rectangle {
#     width: u32,
#     height: u32,
# }
#
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

<span class="caption">Listing 5-16: Reescribiendo el Listado 5-15 usando múltiples
bloques `impl`</span>

No hay ninguna razón para separar estos métodos en múltiples bloques `impl` aquí,
pero esta es una sintaxis válida. Veremos un caso en el que varios bloques `impl`
son útiles en el Capítulo 10, donde discutimos los tipos genéricos y *traits*.

## Resumen

Las estructuras le permiten crear tipos personalizados que son significativos para
su dominio. Mediante el uso de estructuras, puede mantener las piezas asociadas de
datos conectadas entre sí y el nombre de cada pieza para hacer que su código sea claro.
Los métodos le permiten especificar el comportamiento que tienen las instancias de sus
estructuras, y las funciones asociadas le permiten una funcionalidad de espacio de nombres
que es particular de su estructura sin tener una instancia disponible.

Pero las estructuras no son la única forma en que puede crear tipos personalizados:
pasemos a la función enum de Rust para agregar otra herramienta a su caja de herramientas.