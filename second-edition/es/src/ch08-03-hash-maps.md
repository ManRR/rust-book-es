## Almacenamiento de claves con valores asociados en *Hash Maps* (*mapa hash*)

La última de nuestras colecciones comunes es el *hash map*. El tipo
`HashMap<K, V>` almacena una asignación de claves de tipo `K` a valores de
tipo` V`. Lo hace mediante una *función de hash*, que determina cómo coloca
estas claves y valores en la memoria. Muchos lenguajes de programación
admiten este tipo de estructura de datos, pero a menudo usan un nombre
diferente, como hash, mapa, objeto, tabla hash o matriz asociativa, solo por
nombrar algunos.

Los mapas Hash (*hash map*) son útiles cuando se quiere buscar datos no
usando un índice, como se puede hacer con vectores, sino usando una clave que
puede ser de cualquier tipo. Por ejemplo, en un juego, puede hacer un
seguimiento del puntaje de cada equipo en un mapa hash en el que cada clave
es el nombre de un equipo y los valores son el puntaje de cada equipo. Dado
el nombre de un equipo, puedes recuperar su puntaje.

Repasaremos la API básica de los mapas hash en esta sección, pero muchas más
cosas se esconden en las funciones definidas en `HashMap <K, V>` por la
biblioteca estándar. Como siempre, consulte la documentación estándar de la
biblioteca para obtener más información.

### Crear un nuevo *mapa hash* (*Hash Map*)

Puede crear un mapa hash vacío con `new` y agregar elementos con `insert`. En
el Listado 8-20, estamos haciendo un seguimiento de las puntuaciones de dos
equipos cuyos nombres son “Blue” y “Yellow”. El equipo “Blue” comienza con 10
puntos, y el equipo “Yellow” comienza con 50.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);
```

<span class="caption">Listing 8-20: Creando un nuevo mapa hash e insertando
algunas claves y valores</span>

Tenga en cuenta que necesitamos primero `use` el `HashMap` de la porción de
colecciones de la biblioteca estándar. De nuestras tres colecciones comunes,
esta es la menos utilizada, por lo que no está incluida en las
características introducidas automáticamente en el preludio. Los mapas Hash
también tienen menos soporte de la biblioteca estándar; no hay una macro
incorporada para construirlos, por ejemplo.

Al igual que los vectores, los mapas hash almacenan sus datos en el montículo
(*heap*). Este `HashMap` tiene claves de tipo `String` y valores de tipo
`i32`. Al igual que los vectores, los mapas hash son homogéneos: todas las
claves deben tener el mismo tipo, y todos los valores deben tener el mismo
tipo.

Otra forma de construir un mapa hash es usar el método `collect` en un vector
de tuplas, donde cada tupla consiste en una clave y su valor. El método
`collect` reúne datos en varios tipos de colección, incluido `HashMap`. Por
ejemplo, si tuviéramos los nombres de los equipos y puntajes iniciales en dos
vectores separados, podríamos usar el método `zip` para crear un vector de
tuplas donde “Blue” está emparejado con 10, y así sucesivamente. Entonces
podríamos usar el método `collect` para convertir ese vector de tuplas en un
mapa hash, como se muestra en el Listado 8-21.

```rust
use std::collections::HashMap;

let teams  = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];

let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();
```

<span class="caption">Listing 8-21: Crear un mapa hash a partir de una lista
de equipos y una lista de puntajes</span>

La anotación de tipo `HashMap <_, _>` es necesaria aquí porque es posible
`recopilar` en muchas estructuras de datos diferentes y Rust no sabe cuál
quiere a menos que usted especifique. Para los parámetros para la clave y los
tipos de valor, sin embargo, usamos guiones bajos, y Rust puede inferir los
tipos que contiene el hash map en función de los tipos de datos en los
vectores.

### *Hash Maps* (*mapa hash*) y Propiedad

Para los tipos que implementan el *trait* `Copy`, como `i32`, los valores se
copian en el mapa hash. Para valores de propiedad como `String`, los valores
se moverán y el mapa de hash será el propietario de esos valores, como se
muestra en el Listado 8-22.

```rust
use std::collections::HashMap;

let field_name = String::from("Favorite color");
let field_value = String::from("Blue");

let mut map = HashMap::new();
map.insert(field_name, field_value);
// field_name y field_value no son válidos en este punto, intente usarlos y
// ¡mira qué error de compilación recibes!
```

<span class="caption">Listing 8-22: Mostrar que las claves y los valores son
propiedad del mapa hash una vez que se insertan</span>

No podemos usar las variables `field_name` y `field_value` después de que se
hayan movido al mapa hash con la llamada a `insert`.

Si insertamos referencias a valores en el mapa hash, los valores no se
moverán al mapa hash. Los valores a los que apuntan las referencias deben ser
válidos por lo menos mientras el mapa hash sea válido. Hablaremos más sobre
estos temas en la sección “Validación de referencias con períodos de
vigencia” en el Capítulo 10.

### Acceso a valores en un mapa hash (*Hash Map*)

Podemos obtener un valor del *hash map* (*mapa hash*) proporcionando su clave
para el método `get`, como se muestra en el Listado 8-23.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

let team_name = String::from("Blue");
let score = scores.get(&team_name);
```

<span class="caption">Listing 8-23: Accediendo al puntaje para el equipo azul
almacenado en el mapa hash</span>

Aquí, `score` tendrá el valor asociado con el equipo Blue, y el resultado
será `Some(&10)`. El resultado se envuelve en `Some` porque `get` devuelve
una `Opción <&V>`; si no hay ningún valor para esa clave en el mapa hash,`get` devolverá `None`. El programa deberá manejar la `Option` en una de las
formas que cubrimos en el Capítulo 6.

Podemos iterar sobre cada par clave / valor en un mapa hash de manera similar a como lo hacemos con los vectores, usando un bucle `for`:

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

Este código imprimirá cada par en un orden arbitrario:

```text
Yellow: 50
Blue: 10
```

### Actualización de un mapa Hash (*Hash Map*)

Aunque la cantidad de claves y valores puede crecer, cada clave solo puede
tener un valor asociado con ella a la vez. Cuando desee cambiar los datos en
un *hash map* (*mapa hash*), debe decidir cómo manejar el caso cuando una
clave ya tiene un valor asignado. Puede reemplazar el valor anterior por el
nuevo valor, sin tener en cuenta el valor anterior. Puede mantener el valor
anterior e ignorar el nuevo valor, solo agregando el nuevo valor si la clave
*no* tiene ya un valor. O puede combinar el valor anterior y el nuevo valor.
¡Veamos cómo hacer cada uno de estos!

#### Sobrescribir un valor

Si insertamos una clave y un valor en un mapa hash e insertamos esa misma
clave con un valor diferente, el valor asociado con esa clave será
reemplazado. Aunque el código en el Listado 8-24 llama `insert` dos veces, el
hash map solo contendrá un par de clave/valor porque estamos insertando el
valor para la clave del equipo azul en ambas ocasiones.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();

scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Blue"), 25);

println!("{:?}", scores);
```

<span class="caption">Listing 8-24: Reemplazar un valor almacenado con una
clave particular</span>

Este código imprimirá `{"Blue": 25}`. El valor original de `10` ha sido
sobrescrito.

#### Solo insertar un valor si la clave no tiene valor

Es común verificar si una clave en particular tiene un valor y, si no lo
tiene, insertar un valor para ella. Los mapas hash tienen una API especial
para esta llamada `entry` que toma la clave que desea verificar como
parámetro. El valor de retorno del método `entry` es una enumeración llamada
`Entry` que representa un valor que podría existir o no. Digamos que
queremos verificar si la clave del equipo amarillo tiene un valor asociado.
Si no es así, queremos insertar el valor 50, y lo mismo para el equipo azul.
Usando la API `entry`, el código se ve como el Listado 8-25.

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);

scores.entry(String::from("Yellow")).or_insert(50);
scores.entry(String::from("Blue")).or_insert(50);

println!("{:?}", scores);
```

<span class="caption">Listing 8-25: Usando el método `entry` para insertar
solo si la clave ya no tiene un valor</span>

El método `or_insert` en `Entry` se define para devolver una referencia
mutable al valor de la tecla `Entry` correspondiente si esa clave existe, y
si no, inserta el parámetro como el nuevo valor para esta clave y devuelve
una referencia mutable al nuevo valor. Esta técnica es mucho más limpia que
escribir la lógica nosotros mismos y, además, juega mejor con el comprobador
de préstamos.

Al ejecutar el código en el Listado 8-25 se imprimirá
`{"Yellow": 50, "Blue": 10}`. La primera llamada a `entry` insertará la
clave para el equipo Amarillo con el valor 50 porque el equipo Amarillo ya no
tiene un valor. La segunda llamada a `entry` no cambiará el *hash map*
(*mapa hash*) porque el equipo azul ya tiene el valor 10.

#### Actualización de un valor basado en el valor anterior

Otro caso de uso común para los mapas hash es buscar el valor de una clave y
luego actualizarla en función del valor anterior. Por ejemplo, el Listado
8-26 muestra un código que cuenta cuántas veces aparece cada palabra en algún
texto. Usamos un mapa hash con las palabras como teclas e incrementamos el
valor para realizar un seguimiento de cuántas veces hemos visto esa palabra.
Si es la primera vez que vemos una palabra, primero insertamos el valor 0.

```rust
use std::collections::HashMap;

let text = "hello world wonderful world";

let mut map = HashMap::new();

for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", map);
```

<span class="caption">Listing 8-26: Recuento de ocurrencias de palabras
usando un mapa hash que almacena palabras y cuenta</span>

Este código imprimirá `{"world": 2, "hello": 1, "wonderful": 1}`. El método
`or_insert` en realidad devuelve una referencia mutable (`&mut V`) al valor
de esta clave. Aquí almacenamos esa referencia mutable en la variable `count`,por lo que para asignar a ese valor, primero debemos eliminar la referencia
`count` utilizando el asterisco (`*`). La referencia mutable sale del alcance
al final del ciclo `for`, por lo que todos estos cambios son seguros y están
permitidos por las reglas de préstamo.

### Funciones *hash*

Por defecto, `HashMap` utiliza una función hashing criptográficamente segura
que puede proporcionar resistencia a los ataques de denegación de servicio
(DoS). Este no es el algoritmo de hashing más rápido disponible, pero vale la
pena la compensación para una mejor seguridad que viene con la caída en el
rendimiento. Si perfila su código y descubre que la función hash
predeterminada es demasiado lenta para sus propósitos, puede cambiar a otra
función especificando un *hasher* diferente. Un *hasher* es un tipo que
implementa el *trait* `BuildHasher`. Hablaremos sobre los *trait* y cómo
implementarlos en el Capítulo 10. No necesariamente tiene que implementar su
propio hasher desde cero; [crates.io] (https://crates.io) tiene bibliotecas
compartidas por otros usuarios de Rust que proporcionan *hashers* que
implementan muchos algoritmos *hash* comunes.

## Resumen

Los vectores, *string* y *hash maps* proporcionarán una gran cantidad de
funcionalidad necesaria en los programas cuando necesite almacenar, acceder y
modificar datos. Aquí hay algunos ejercicios que ahora debes estar preparado
para resolver:

* Dada una lista de enteros, usa un vector y regresa la media (el valor
  promedio), la media (cuando se clasifica, el valor en la posición media) y el modo (el valor que ocurre con mayor frecuencia, un *hash maps* será útil aquí ) de la lista.
* Convertir *strings to pig latin*. La primera consonante de cada palabra se
  mueve al final de la palabra y se agrega “ay”, por lo que “first” se convierte en “irst-fay.” Las palabras que comienzan con una vocal tienen “hay”agregado al final ("apple "Se convierte en “apple-hay”). ¡Tenga en cuenta los detalles sobre la codificación UTF-8!
* Utilizando un *hash maps* y vectores, cree una interfaz de texto para
  permitir que un usuario agregue nombres de empleados a un departamento de una empresa. Por ejemplo, "Agregar Sally a la ingeniería" o "Agregar Amir a las ventas". Luego, permita que el usuario obtenga una lista de todas las personas de un departamento o de todas las personas de la empresa por departamento, ordenados alfabéticamente.

¡La documentación estándar de la API de la biblioteca describe los métodos
que los vectores, los *string* y los *hash maps* ¡Eso será útil para estos
ejercicios!

Estamos entrando en programas más complejos en los que las operaciones pueden
fallar, por lo tanto, es un momento perfecto para analizar el manejo de
errores. ¡Haremos eso en el próximo!