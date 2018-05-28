## El operador de flujo de control `match`

Rust tiene un operador de flujo de control extremadamente poderoso llamado `match`
que le permite comparar un valor con una serie de patrones y luego ejecutar el código
según el patrón que coincida. Los patrones pueden estar formados por valores literales,
nombres de variables, comodines y muchas otras cosas; El Capítulo 18 cubre todos los diferentes
tipos de patrones y lo que hacen. El poder de `match` proviene de la expresividad de
los patrones y del hecho de que el compilador confirma que se manejan todos los casos posibles.

Piense en una expresión de `match` como una máquina clasificadora de monedas: las monedas se
deslizan por una pista con orificios de diferentes tamaños a lo largo de ella, y cada moneda
cae por el primer orificio que encuentra que encaja. De la misma manera, los valores pasan
por cada patrón en una coincidencia `match`, y en el primer patrón en el que el valor encaje,
el valor cae en el bloque de código asociado para ser utilizado durante la ejecución.

¡Debido a que acabamos de mencionar monedas, utilicémoslas como ejemplo usando `match`!
Podemos escribir una función que puede tomar una moneda desconocida de los Estados Unidos y,
de manera similar a la máquina de conteo, determinar qué moneda es y devolvera su valor en
centavos, como se muestra aquí en el Listado 6-3.

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

<span class="caption">Listing 6-3: Un enum y una expresión de `match` que
tiene las variantes de la enumeración como sus patrones</span>

Vamos a desglosar el `match` en la función `value_in_cents`. Primero, listamos
la palabra clave `match` seguida de una expresión, que en este caso es el valor
`coin`. Esto parece muy similar a una expresión utilizada con `if`, pero hay una
gran diferencia: con `if`, la expresión necesita devolver un valor booleano, pero
aquí, puede ser de cualquier tipo. El tipo de `coin` en este ejemplo es el *enum* de
`coin` que definimos en la línea 1.

A continuación están los brazos del `match`. Un brazo tiene dos partes: un patrón y algo de código.
El primer brazo aquí tiene un patrón que es el valor `Coin::Penny` y luego el `=> `
operador que separa el patrón y el código para ejecutar. El código en este caso
es solo el valor `1`. Cada brazo está separado del siguiente con una coma.

Cuando se ejecuta la expresión `match`, compara el valor resultante contra
el patrón de cada brazo, en orden. Si un patrón coincide con el valor, el código
asociado con ese patrón se ejecuta. Si ese patrón no coincide con el
valor, la ejecución continúa al siguiente brazo, al igual que en una máquina
clasificadora de monedas. Podemos tener tantas brazos como necesitemos:
en el Listado 6-3, nuestro `match` tiene cuatro brazos.

El código asociado con cada brazo es una expresión, y el valor resultante de
la expresión en el brazo correspondiente es el valor que se devuelve para el
expresión completa del `match`.

Las llaves normalmente no se usan si el código del *match* es corto, ya que
en el listado 6-3 donde cada brazo simplemente devuelve un valor. Si quieres ejecutar múltiples
líneas de código en un brazo de coincidencia, puede usar llaves. Por ejemplo, el
siguiente código imprimirá “Lucky penny!” cada vez que se llame al método con
un `Coin::Penny` pero aún devolvería el último valor del bloque, `1`:

```rust
# enum Coin {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter,
# }
#
fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        },
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

### Patrones que se unen a los valores

Otra característica útil de los brazos del *match* es que pueden unirse a las partes
de los valores que coinciden con el patrón. Así es como podemos extraer valores de las variantes *enum*.

Como ejemplo, cambiemos una de nuestras variantes *enum* para contener datos dentro de ella.
De 1999 a 2008, los Estados Unidos acuñaron cuartos con diferentes diseños para cada uno de
los 50 estados en un lado. Ninguna otra moneda tiene diseños de estado, por lo que solo
los cuartos tienen este valor adicional. Podemos agregar esta información a nuestro `enum`
cambiando la variante `Quarter` para incluir un valor `UsState` almacenado dentro de él,
lo que hemos hecho aquí en el Listado 6-4.

```rust
#[derive(Debug)] // so we can inspect the state in a minute
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState),
}
```

<span class="caption">Listing 6-4: Una enumeración de `Coin` en la cual la variante `Quarter`
también tiene un valor `UsState`</span>

Imaginemos que un amigo nuestro está tratando de reunir los 50 *Quarters* (*cuartos)* estatales.
Mientras clasificamos nuestro cambio suelto por tipo de moneda, también llamaremos el nombre
del estado asociado con cada *quarter*, por lo que si es uno que nuestro amigo no tiene,
pueden agregarlo a su colección.

En la expresión de coincidencia para este código, agregamos una variable llamada `state` al
patrón que coincide con los valores de la variante` Coin::Quarter`. Cuando coincide un `Coin::Quarter`,
la variable `state` se vinculará al valor del estado de ese *quarter*. 
Entonces podemos usar `state` en el código para ese brazo, así:

```rust
# #[derive(Debug)]
# enum UsState {
#    Alabama,
#    Alaska,
# }
#
# enum Coin {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter(UsState),
# }
#
fn value_in_cents(coin: Coin) -> u32 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        },
    }
}
```

Si tuviéramos que llamar a `value_in_cents(Coin::Quarter(UsState::Alaska))`,
`coin` sería `Coin::Quarter(UsState::Alaska)`. Cuando comparamos ese valor
con cada uno de los brazos de coincidencia, ninguno de ellos coincide hasta
que llegamos a `Coin::Quarter(state)`. En ese punto, el enlace para `state`
será el valor `UsState::Alaska`. Entonces podemos usar ese enlace en la expresión
`println!`, Obteniendo así el valor de estado interno de la variante *enum*
`Coin` para `Quarter`.

### Emparejando con `Option<T>`

En la sección anterior, queríamos obtener el valor interno de `T` fuera del caso `Some`
cuando usamos `Option <T>`; también podemos manejar `Option <T>` usando `match` como
lo hicimos con la enumeración `Coin`! En lugar de comparar monedas, compararemos
las variantes de `Option <T>`, pero la forma en que funciona la expresión `match`
sigue siendo la misma.

Digamos que queremos escribir una función que tenga una `Opción <i32>` y, si hay un
valor dentro, agrega 1 a ese valor. Si no hay un valor dentro, la función debe devolver
el valor `None` y no intentar realizar ninguna operación.

Esta función es muy fácil de escribir, gracias a `match`, y se verá como el Listado 6-5.

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

let five = Some(5);
let six = plus_one(five);
let none = plus_one(None);
```

<span class="caption">Listing 6-5: Una función que usa una expresión `match` en un `Option <i32>`</span>

Examinemos la primera ejecución de `plus_one` con más detalle. Cuando llamamos a `plus_one(five)`,
la variable `x` en el cuerpo de `plus_one` tendrá el valor `Some(5)`. Luego lo
comparamos con cada brazo del *match*.

```rust,ignore
None => None,
```

El valor `Some(5)` no coincide con el patrón `None`, por lo que continuamos al siguiente brazo.

```rust,ignore
Some(i) => Some(i + 1),
```

¿`Some(5)` coincide con `Some(i)`? Porque sí lo hace! Tenemos la misma variante.
El `i` se une al valor contenido en `Some`, entonces `i` toma el valor `5`.
El código en el brazo de coincidencia se ejecuta luego, por lo que agregamos 1 al
valor de `i` y creamos un nuevo valor `Some` con nuestro total de '6' dentro.

Ahora consideremos la segunda llamada de `plus_one` en el Listado 6-5,
donde `x` es `None`. Ingresamos al `match` y lo comparamos con el primer brazo.

```rust,ignore
None => None,
```

¡Coincide! No hay ningún valor para agregar, por lo que el programa se detiene y
devuelve el valor `None` en el lado derecho de `=>`. Debido a que el primer brazo
coincide, no se comparan otros brazos.

Combinar `match` y *enums* es útil en muchas situaciones. Verá este patrón mucho
en el código Rust: `match` contra una enumeración, enlazará una variable a los datos
internos y luego ejecutará el código en función de ello. Es un poco complicado al
principio, pero una vez que se acostumbre, deseará tenerlo en todos los lenguajes.
Es constantemente un favorito del usuario.

### Matches Are Exhaustive

Hay otro aspecto de `match` que debemos discutir. Considere esta versión de
nuestra función `plus_one` que tiene un error y no compilará:

```rust,ignore
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
    }
}
```

No manejamos el caso `None`, por lo que este código provocará un error.
Afortunadamente, es un error que Rust sabe atrapar. Si tratamos de compilar
este código, obtendremos este error:

```text
error[E0004]: non-exhaustive patterns: `None` not covered
 -->
  |
6 |         match x {
  |               ^ pattern `None` not covered
```

¡Rust sabe que no cubrimos todos los casos posibles e incluso sabemos qué patrón
olvidamos! Las coincidencias en Rust son *exhaustivas*: debemos agotar todas las
posibilidades para que el código sea válido. Especialmente en el caso de `Option <T>`,
cuando Rust evita que olvidemos manejar explícitamente el caso `None`,
nos protege de asumir que tenemos un valor cuando podríamos tener nulo, y
así cometer el error billonario Discutido antes.

### The `_` Placeholder

Rust también tiene un patrón que podemos usar cuando no queremos enumerar todos
los valores posibles. Por ejemplo, un `u8` puede tener valores válidos de 0 a 255.
Si solo nos importan los valores 1, 3, 5 y 7, no queremos tener que enumerar
0, 2, 4, 6, 8, 9 hasta 255. Afortunadamente, no es necesario: podemos usar el
patrón especial `_` en su lugar:

```rust
let some_u8_value = 0u8;
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```

El patrón `_` coincidirá con cualquier valor. Al ponerlo después de nuestros otros
brazos, el `_` coincidirá con todos los casos posibles que no se hayan especificado
antes. El `()` es solo el valor unitario, por lo que no sucederá nada en el caso `_`.
Como resultado, podemos decir que no queremos hacer nada para todos los valores posibles
que no enumeramos antes del marcador de posición `_`.

Sin embargo, la expresión `match` puede ser un poco prolija en una situación en la
que solo nos importa *uno* de los casos. Para esta situación, Rust proporciona `if let`.
