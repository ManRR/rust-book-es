## Almacenamiento de listas de valores con vectores

El primer tipo de colección que veremos es `Vec <T>`, también conocido como
*vector*. Los vectores le permiten almacenar más de un valor en una sola
estructura de datos que coloca todos los valores uno al lado del otro en la
memoria. Los vectores solo pueden almacenar valores del mismo tipo. Son
útiles cuando tiene una lista de elementos, como las líneas de texto en un
archivo o los precios de los artículos en un carrito de compras.

### Creando un nuevo Vector

Para crear un nuevo vector vacío, podemos llamar a la función `Vec::new`,
como se muestra en el Listado 8-1.

```rust
let v: Vec<i32> = Vec::new();
```

<span class="caption">Listing 8-1: Creando un nuevo vector vacío para
contener valores del tipo `i32`</span>

Tenga en cuenta que agregamos una anotación de tipo aquí. Como no estamos
insertando ningún valor en este vector, Rust no sabe qué tipo de elementos pretendemos almacenar. Éste es un punto importante. Los vectores se
implementan usando genéricos; cubriremos cómo usar genéricos con sus propios
tipos en el Capítulo 10. Por ahora, sepa que el tipo `Vec <T>` proporcionado
por la biblioteca estándar puede contener cualquier tipo, y cuando un vector
específico tiene un tipo específico, el tipo se especifica dentro de los
corchetes angulares. En el listado 8-1, le hemos dicho a Rust que `Vec <T>`
en `v` contendrá elementos del tipo `i32`.

En un código más realista, Rust a menudo puede inferir el tipo de valor que
desea almacenar una vez que inserta los valores, por lo que rara vez necesita
hacer esta anotación de tipo. Es más común crear un `Vec <T>` que tiene
valores iniciales, y Rust proporciona la macro `vec!` Para mayor comodidad.
La macro creará un nuevo vector que contiene los valores que le das. El
listado 8-2 crea un nuevo `Vec <i32>` que contiene los valores `1`,`2` y `3`.

```rust
let v = vec![1, 2, 3];
```

<span class="caption">Listing 8-2: Creando un nuevo vector que contiene
valores</span>

Como hemos dado valores iniciales de `i32`, Rust puede inferir que el tipo
de `v` es `Vec <i32>`, y la anotación de tipo no es necesaria. A continuación
veremos cómo modificar un vector.

### Actualizando un Vector

Para crear un vector y luego agregarle elementos, podemos usar el método `push`, como se muestra en el Listado 8-3.

```rust
let mut v = Vec::new();

v.push(5);
v.push(6);
v.push(7);
v.push(8);
```

<span class="caption">Listing 8-3: Usando el método `push` para agregar
valores a un vector</span>

Como con cualquier variable, si queremos poder cambiar su valor, tenemos que
hacerlo mutable usando la palabra clave `mut`, como se discutió en el
Capítulo 3. Los números que colocamos dentro son todos de tipo `i32`, y Rust
infiere esto de los datos, por lo que no necesitamos la anotación `Vec <i32>`.

### Dropping un Vector *Drops* sus elementos

Como cualquier otra `struct`, un vector se libera cuando sale del alcance, como se indica en el Listado 8-4.


```rust
{
    let v = vec![1, 2, 3, 4];

    // hacer cosas con v

} // <- v sale del alcance y se libera aquí
```

<span class="caption">Listing 8-4: Mostrando donde el vector y sus elementos son caídos (*dropped*)</span>

Cuando el vector se descarta, todos sus contenidos también se descartan, lo
que significa que los enteros que contiene se limpiarán. Esto puede parecer
un punto directo, pero puede ser un poco más complicado cuando comiences a
introducir referencias a los elementos del vector. ¡Vamos a abordar eso lo
próximo!

### Lectura de elementos de vectores

Ahora que sabe cómo crear, actualizar y destruir vectores, saber cómo leer
sus contenidos es un buen paso. Hay dos formas de referenciar un valor
almacenado en un vector. En los ejemplos, hemos anotado los tipos de valores
que se devuelven de estas funciones para una mayor claridad.

El Listado 8-5 muestra ambos métodos de acceso a un valor en un vector, ya
sea con sintaxis de indexación o el método `get`.

```rust
let v = vec![1, 2, 3, 4, 5];

let third: &i32 = &v[2];
let third: Option<&i32> = v.get(2);
```

<span class="caption">Listing 8-5: Usar la sintaxis de indexación o el método
`get` para acceder a un elemento en un vector</span>

Tenga en cuenta dos detalles aquí. Primero, usamos el valor de índice de `2`
para obtener el tercer elemento: los vectores están indexados por número,
comenzando por cero. Segundo, las dos formas de obtener el tercer elemento
son usando `&` y `[]`, que nos da una referencia, o usando el método `get`
con el índice pasado como argumento, lo que nos da una` Opción <&T> `.

Rust tiene dos formas de hacer referencia a un elemento para que pueda elegir
cómo se comporta el programa cuando intenta usar un valor de índice para el
que el vector no tiene un elemento. Como ejemplo, veamos qué hará un programa
si tiene un vector que contiene cinco elementos y luego intenta acceder a un
elemento en el índice 100, como se muestra en el Listado 8-6.

```rust,should_panic
let v = vec![1, 2, 3, 4, 5];

let does_not_exist = &v[100];
let does_not_exist = v.get(100);
```

<span class="caption">Listing 8-6: Intentando acceder al elemento en el
índice 100 en un vector que contiene cinco elementos</span>

Cuando ejecutamos este código, el primer método `[]` hará que el programa
entre en pánico porque hace referencia a un elemento inexistente. Este método
se usa mejor cuando quiere que su programa se bloquee si hay un intento de
acceder a un elemento más allá del fin del vector.

Cuando el método `get` se pasa un índice que está fuera del vector, regresa
`None` sin entrar en pánico. Deberías usar este método si accedes a un
elemento más allá del rango del vector ocurre ocasionalmente bajo
circunstancias normales. Su código tendrá lógica para manejar tener
`Some(&element)` o `None`, como se discutió en el Capítulo 6. Por ejemplo, el
índice podría venir de una persona que ingresa un número. Si ingresan
accidentalmente un número que también es grande y el programa obtiene un
valor `None`, podría decirle al usuario cuántos artículos están en el vector
actual y les da otra oportunidad de ingresar un válido valor. Eso sería más
fácil de usar que estrellar el programa debido a un error tipográfico.

Cuando el programa tiene una referencia válida, el comprobador de préstamos
impone las reglas de propiedad y de endeudamiento (cubiertas en el Capítulo 4)para asegurar esta referencia y cualquier otra referencia a los contenidos
del vector sigue siendo válida. Recuerda la regla establece que no puede haber referencias mutables e inmutables en el mismo alcance. Esa regla se
aplica en el Listado 8-7, donde tenemos una referencia inmutable a el primer elemento en un vector e intenta agregar un elemento al final, que no trabajó.

```rust,ignore
let mut v = vec![1, 2, 3, 4, 5];

let first = &v[0];

v.push(6);
```

<span class="caption">Listing 8-7: Intentando agregar un elemento a un vector
mientras se mantiene una referencia a un elemento</span>

Compilar este código dará como resultado este error:

```text
error[E0502]: cannot borrow `v` as mutable because it is also borrowed as immutable
 -->
  |
4 |     let first = &v[0];
  |                  - immutable borrow occurs here
5 |
6 |     v.push(6);
  |     ^ mutable borrow occurs here
7 |
8 | }
  | - immutable borrow ends here
```

El código en el listado 8-7 podría parecer que debería funcionar: ¿por qué
una referencia al primer elemento se preocupa por qué cambios al final del
vector? Este error se debe a la forma en que funcionan los vectores: agregar
un nuevo elemento al final del vector puede requerir asignar nueva memoria y
copiar los elementos antiguos al nuevo espacio, si no hay espacio suficiente
para poner todos los elementos al lado de cada uno otro donde el vector es
actualmente. En ese caso, la referencia al primer elemento estaría apuntando
a la memoria desasignada. Las reglas de endeudamiento evitan que los
programas terminen en esa situación.

> Nota: para obtener más información sobre los detalles de implementación del
tipo `Vec <T>`, “The
> Rustonomicon” en https://doc.rust-lang.org/stable/nomicon/vec.html.

### Iterando sobre los valores en un vector

Si queremos acceder a cada elemento en un vector a su vez, podemos iterar a
través de todos los elementos en lugar de usar índices para acceder uno a la
vez. El Listado 8-8 muestra cómo usar un bucle `for` para obtener referencias
inmutables a cada elemento en un vector de valores `i32` e imprimirlos.

```rust
let v = vec![100, 32, 57];
for i in &v {
    println!("{}", i);
}
```

<span class="caption">Listing 8-8: Imprimir cada elemento en un vector
iterando sobre los elementos usando un bucle `for`</span>

También podemos iterar sobre referencias mutables a cada elemento en un
vector mutable para hacer cambios a todos los elementos. El bucle `for` en el
Listado 8-9 agregará `50` a cada elemento.

```rust
let mut v = vec![100, 32, 57];
for i in &mut v {
    *i += 50;
}
```

<span class="caption">Listing 8-9: Iteración sobre referencias mutables a
elementos en un vector</span>

Para cambiar el valor al que se refiere la referencia mutable, debemos usar
el operador de desreferencia (`*`) para obtener el valor en `i` antes de
poder usar el operador `+=`.

### Usando un *Enum* para almacenar múltiples tipos

Al comienzo de este capítulo, dijimos que los vectores solo pueden almacenar
valores que son del mismo tipo. Esto puede ser un inconveniente;
definitivamente hay casos de uso para la necesidad de almacenar una lista de
artículos de diferentes tipos. Afortunadamente, las variantes de una
enumeración se definen bajo el mismo tipo de enumeración, de modo que cuando
necesitamos almacenar elementos de un tipo diferente en un vector, podemos
definir y usar una enumeración.

Por ejemplo, supongamos que queremos obtener valores de una fila en una hoja
de cálculo en la que algunas de las columnas de la fila contienen números
enteros, algunos números de coma flotante y algunas cadenas. Podemos definir
una enumeración cuyas variantes contendrán los diferentes tipos de valores, y
luego todas las variantes enum se considerarán del mismo tipo: la de la
enumeración. Entonces podemos crear un vector que contenga esa enumeración y
así, en última instancia, tenga diferentes tipos. Hemos demostrado esto en el
Listado 8-10.

```rust
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

<span class="caption">Listing 8-10: Definir un `enum` para almacenar valores
de diferentes tipos en un vector</span>

Rust necesita saber qué tipos estarán en el vector en el momento de la
compilación para que sepa exactamente cuánta memoria en el montículo (*heap*)
será necesaria para almacenar cada elemento. Una ventaja secundaria es que
podemos ser explícitos sobre qué tipos están permitidos en este vector. Si Rust permitiera que un vector retuviera cualquier tipo, existiría la
posibilidad de que uno o más de los tipos causen errores con las operaciones
realizadas en los elementos del vector. El uso de una expresión enum más
`match` significa que Rust se asegurará en el momento de la compilación de
que se manejen todos los casos posibles, como se discutió en el Capítulo 6.

Cuando estás escribiendo un programa, si no conoces el conjunto exhaustivo de
tipos que el programa obtendrá en tiempo de ejecución para almacenarlo en un vector, la técnica *enum* no funcionará. En cambio, puede usar un objeto
*trait*, que veremos en el Capítulo 17.

Ahora que hemos discutido algunas de las formas más comunes de usar vectores,
asegúrese de revisar la documentación de API para todos los muchos métodos
útiles definidos en `Vec <T>` por la biblioteca estándar. Por ejemplo, además
de `push`, un método `pop` elimina y devuelve el último elemento. Pasemos al
siguiente tipo de colección: `String`!