# Tipos genéricos, *Traits*, y *Lifetimes*

Cada lenguaje de programación tiene herramientas para manejar con eficacia la
duplicación de conceptos. En Rust, una de esas herramientas es los
*genéricos*. Los genéricos son complementos abstractos para tipos de concreto
u otras propiedades. Cuando estamos escribiendo código, podemos expresar el
comportamiento de los genéricos o cómo se relacionan con otros genéricos sin
saber qué habrá en su lugar al compilar y ejecutar el código.

Similar a la forma en que una función toma parámetros con valores
desconocidos para ejecutar el mismo código en múltiples valores concretos,
las funciones pueden tomar parámetros de algún tipo genérico en lugar de un
tipo concreto, como `i32` o `String`. De hecho, ya hemos usado genéricos en
el Capítulo 6 con `Opción <T>`, Capítulo 8 con `Vec <T>` y `HashMap <K, V>`,
y Capítulo 9 con `Result <T, E>`. ¡En este capítulo, explorará cómo definir
sus propios tipos, funciones y métodos con genéricos!.

Primero, repasaremos cómo extraer una función para reducir la duplicación de
código. A continuación, utilizaremos la misma técnica para hacer una función
genérica a partir de dos funciones que difieren solo en los tipos de sus
parámetros. También explicaremos cómo usar los tipos genéricos en las
definiciones de estructura y enumeración.

Luego aprenderá cómo usar *trait* para definir el comportamiento de una
manera genérica. Puede combinar rasgos con tipos genéricos para restringir un
tipo genérico solo a aquellos tipos que tienen un comportamiento particular,
a diferencia de cualquier tipo.

Finalmente, estudiaremos *lifetimes*, una variedad de genéricos que brindan
al compilador información sobre cómo las referencias se relacionan entre sí.
Los tiempos de vida nos permiten tomar valores prestados en muchas
situaciones y al mismo tiempo permitir que el compilador verifique que las
referencias sean válidas.

## Eliminar la duplicación mediante la extracción de una función

Antes de sumergirse en la sintaxis de los genéricos, veamos primero cómo
eliminar la duplicación que no involucra tipos genéricos extrayendo una
función. ¡Entonces aplicaremos esta técnica para extraer una función
genérica! De la misma forma que reconoce el código duplicado para extraer en
una función, comenzará a reconocer el código duplicado que puede usar
genéricos.

Considere un programa corto que encuentre el número más grande en una lista,
como se muestra en el Listado 10-1.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = number_list[0];

    for number in number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
#  assert_eq!(largest, 100);
}
```

<span class="caption">Listado 10-1: codigo para encontrar el número más grande en una lista de números</span>

Este código almacena una lista de enteros en la variable `number_list` y
coloca el primer número en la lista en una variable llamada `largest`. Luego
itera a través de todos los números en la lista, y si el número actual es
mayor que el número almacenado en `largest`, reemplaza el número en esa
variable. Sin embargo, si el número actual es menor que el número más grande
visto hasta ahora, la variable no cambia y el código pasa al siguiente número
de la lista. Después de considerar todos los números en la lista, `largest`
debería contener el número más grande, que en este caso es 100.

Para encontrar el número más grande en dos listas diferentes de números,
podemos duplicar el código en el Listado 10-1 y usar la misma lógica en dos
lugares diferentes en el programa, como se muestra en el Listado 10-2.

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let mut largest = number_list[0];

    for number in number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let mut largest = number_list[0];

    for number in number_list {
        if number > largest {
            largest = number;
        }
    }

    println!("The largest number is {}", largest);
}
```

<span class="caption">Listado 10-2: codigo para encontrar el número más
grande en *dos* listas de números</span>

Aunque este código funciona, la duplicación del código es tediosa y propensa
a errores. También tenemos que actualizar el código en varios lugares cuando
queremos cambiarlo.

Para eliminar esta duplicación, podemos crear una abstracción definiendo una
función que opera en cualquier lista de enteros que se le otorguen en un
parámetro. Esta solución hace que nuestro código sea más claro y nos permite
expresar el concepto de encontrar el número más grande en una lista de forma
abstracta.

En el Listado 10-3, extrajimos el código que encuentra el número más grande
en una función llamada `largest`. A diferencia del código en el listado 10-1,
que puede encontrar el número más grande en una sola lista en particular,
este programa puede encontrar el número más grande en dos listas diferentes.

<span class="filename">Filename: src/main.rs</span>

```rust
fn largest(list: &[i32]) -> i32 {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}

fn main() {
    let number_list = vec![34, 50, 25, 100, 65];

    let result = largest(&number_list);
    println!("The largest number is {}", result);
#    assert_eq!(result, 100);

    let number_list = vec![102, 34, 6000, 89, 54, 2, 43, 8];

    let result = largest(&number_list);
    println!("The largest number is {}", result);
#    assert_eq!(result, 6000);
}
```

<span class="caption">Listado 10-3: código resumido para encontrar el número más grande en dos listas</span>

La función `largest` tiene un parámetro llamado `list`, que representa
cualquier porción concreta de valores `i32` que podríamos pasar a la función.
Como resultado, cuando llamamos a la función, el código se ejecuta en los
valores específicos que pasamos.

En resumen, aquí están los pasos que tomamos para cambiar el código del
Listado 10-2 al Listado 10-3:

1. Identificar código duplicado.
2. Extraiga el código duplicado en el cuerpo de la función y especifique las
 entradas y los valores de retorno de ese código en la firma de la función.
3. Actualice las dos instancias de código duplicado para llamar a la función
 en su lugar.

A continuación, utilizaremos estos mismos pasos con los genéricos para
reducir la duplicación de código de diferentes maneras. De la misma manera
que el cuerpo de la función puede operar en una "lista" abstracta en lugar de
valores específicos, los genéricos permiten que el código opere en tipos
abstractos.

Por ejemplo, supongamos que tenemos dos funciones: una que encuentra el
elemento más grande en una porción de valores `i32` y otra que encuentra el
elemento más grande en una porción de valores `char`. ¿Cómo eliminaríamos esa
duplicación? ¡Vamos a averiguar!