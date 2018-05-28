## Referencias y préstamos (*References and Borrowing*)

El problema con el código de tupla en el Listado 4-5 es que tenemos que devolver
el `String` a la función de llamada para que podamos seguir usando el `String`
después de la llamada a `calculate_length`, porque el `String` se movió a `calculate_length`.

Aquí se explica cómo definiría y usaría una función `calculate_length` que hace referencia a
un objeto como parámetro en lugar de tomar posesión del valor:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

fn calculate_length(s: &String) -> usize {
    s.len()
}
```

Primero, observe que todo el código de tupla en la declaración de la variable
y el valor de retorno de la función han desaparecido. En segundo lugar, tenga
en cuenta que pasamos `&s1` en `calculate_length` y, en su definición, tomamos
`&String` en lugar de `String`.

Estos *ampersands* son *referencias*, y le permiten referirse a algún valor sin
tomar posesión de él. La figura 4-5 muestra un diagrama.

<img alt="&String s pointing at String s1" src="img/trpl04-05.svg" class="center" />

<span class="caption">Figure 4-5: Un diagrama de `& String s` apuntando a `String s1`</span>

> Nota: lo contrario de hacer referencia usando `&` es *dereferencing*, que se
> logra con el operador de desreferencia, `*`. Veremos algunos usos del operador de
> desreferencia en el Capítulo 8 y discutiremos los detalles de desreferenciación en
> el Capítulo 15.

Echemos un vistazo más de cerca a la llamada de función aquí:

```rust
# fn calculate_length(s: &String) -> usize {
#     s.len()
# }
let s1 = String::from("hello");

let len = calculate_length(&s1);
```

La sintaxis `&s1` nos permite crear una referencia que *se refiere* al valor de `s1`
pero que no le pertenece. Como no es el propietario, el valor al que apunta no se eliminará
cuando la referencia salga del alcance.

Del mismo modo, la firma de la función usa `&` para indicar que el tipo del parámetro `s`
es una referencia. Agreguemos algunas anotaciones explicativas:

```rust
fn calculate_length(s: &String) -> usize { // s es una referencia a un String
    s.len()
} // Aquí, s sale del alcance. Pero como no tiene propiedad de lo que se refiere,
  // no sucede nada.
```

El ámbito en el que la variable `s` es válida es el mismo que el del parámetro de
cualquier función, pero no descartamos los puntos de referencia cuando sale del alcance
porque no tenemos la propiedad. Cuando las funciones tienen referencias como parámetros
en lugar de los valores reales, no necesitaremos devolver los valores para devolver la
propiedad, porque nunca tuvimos la propiedad.

Llamamos tener referencias como parámetros de función *préstamos* (*borrowing*).
Como en la vida real, si una persona posee algo, puede pedir prestado de ellos.
Cuando hayas terminado, debes devolverlo.

Entonces, ¿qué sucede si tratamos de modificar algo que estamos pidiendo prestado?
Pruebe el código en el Listado 4-6. Alerta de spoiler: ¡no funciona!

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

<span class="caption">Listing 4-6: Intentando modificar un valor prestado</span>

Aquí está el error:

```text
error[E0596]: cannot borrow immutable borrowed content `*some_string` as mutable
 --> error.rs:8:5
  |
7 | fn change(some_string: &String) {
  |                        ------- use `&mut String` here to make mutable
8 |     some_string.push_str(", world");
  |     ^^^^^^^^^^^ cannot borrow as mutable
```

Así como las variables son inmutables por defecto, también lo son las referencias.
No podemos modificar algo a lo que tenemos referencia.

### Referencias mutables

Podemos corregir el error en el código del Listado 4-6 con solo un pequeño ajuste:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

Primero, tuvimos que cambiar `s` para que fuera `mut`. Luego tuvimos que crear
una referencia mutable con `&mut s` y aceptar una referencia mutable con
`some_string: &mut String`.

Pero las referencias mutables tienen una gran restricción: puede tener solo una
referencia mutable a una pieza particular de datos en un ámbito particular.
Este código fallará:

```rust,ignore
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;
```

Aquí está el error:

```text
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> borrow_twice.rs:5:19
  |
4 |     let r1 = &mut s;
  |                   - first mutable borrow occurs here
5 |     let r2 = &mut s;
  |                   ^ second mutable borrow occurs here
6 | }
  | - first borrow ends here
```

Esta restricción permite la mutación, pero de una manera muy controlada.
Es algo con lo que los nuevos Rustaceos luchan, porque la mayoría de los
lenguajes te permiten mutar cuando lo desees.

El beneficio de tener esta restricción es que Rust puede evitar carreras de
datos en tiempo de compilación. Una *data race* es similar a una condición de
carrera y ocurre cuando se producen estos tres comportamientos:

* Dos o más punteros acceden a los mismos datos al mismo tiempo.
* Al menos uno de los punteros se está utilizando para escribir en los datos.
* No se usa ningún mecanismo para sincronizar el acceso a los datos.

Las carreras de datos causan un comportamiento indefinido y pueden ser
difíciles de diagnosticar y corregir cuando intenta rastrearlos en tiempo
de ejecución; Rust evita que este problema ocurra porque ni siquiera compilará
código con *data race*.

Como siempre, podemos usar llaves para crear un nuevo alcance, permitiendo
múltiples referencias mutables, pero no *simultáneas*:

```rust
let mut s = String::from("hello");

{
    let r1 = &mut s;

} // r1 sale fuera del alcance aquí, por lo que podemos hacer una nueva referencia sin problemas.

let r2 = &mut s;
```

Existe una regla similar para combinar referencias mutables e inmutables. Este
código da como resultado un error:

```rust,ignore
let mut s = String::from("hello");

let r1 = &s; // no problem
let r2 = &s; // no problem
let r3 = &mut s; // BIG PROBLEM
```

Aquí está el error:

```text
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as
immutable
 --> borrow_thrice.rs:6:19
  |
4 |     let r1 = &s; // no problem
  |               - immutable borrow occurs here
5 |     let r2 = &s; // no problem
6 |     let r3 = &mut s; // BIG PROBLEM
  |                   ^ mutable borrow occurs here
7 | }
  | - immutable borrow ends here
```

¡Uf! Nosotros *tampoco* no podemos tener una referencia mutable mientras tengamos
una referencia inmutable. Los usuarios de una referencia inmutable no esperan que
los valores cambien repentinamente de debajo de ellos. Sin embargo, las referencias
inmutables múltiples están bien porque nadie que solo está leyendo los datos tiene
la capacidad de afectar la lectura de los datos de otra persona.

Aunque estos errores pueden ser frustrantes a veces, recuerde que es el compilador
Rust el que señala un posible error anticipadamente (en tiempo de compilación en
lugar de en tiempo de ejecución) y le muestra exactamente dónde está el problema.
Entonces no tienes que rastrear por qué tus datos no son lo que pensabas que eran.

### Referencias colgantes

En lenguajes con punteros, es fácil crear erróneamente un *puntero colgando*,
un puntero que hace referencia a una ubicación en la memoria que se le pudo
haber dado a alguien más, liberando algo de memoria mientras se conserva un
puntero a esa memoria. En Rust, por el contrario, el compilador garantiza que
las referencias nunca serán referencias colgantes: si tiene una referencia
a algunos datos, el compilador se asegurará de que los datos no salgan del
alcance antes que la referencia a los datos.

Intentemos crear una referencia colgante, que Rust evitará con un error en
tiempo de compilación:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from("hello");

    &s
}
```

Aquí está el error:

```text
error[E0106]: missing lifetime specifier
 --> main.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is
  no value for it to be borrowed from
  = help: consider giving it a 'static lifetime
```

Este mensaje de error se refiere a una función que aún no hemos cubierto: tiempos de vida.
Analizaremos los tiempos de vida en detalle en el Capítulo 10. Pero, si ignora las partes
acerca de las vidas útiles, el mensaje contiene la clave de por qué este código es un problema:

```text
en-this function's return type contains a borrowed value, but there is no value
for it to be borrowed from.

es-el tipo de devolución de esta función contiene un valor prestado, pero no hay
ningún valor para que se tome prestado.
```

Echemos un vistazo más de cerca a lo que está sucediendo exactamente en cada etapa de
nuestro código `dangle`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn dangle() -> &String { // dangle devuelve una referencia a un String

    let s = String::from("hello"); // s es una nuevo String

    &s // devolvemos una referencia a String, s
} // ¡Toma! s sale del alcance y se descarta. Su memoria desaparece.
  // ¡Peligro!
```

Debido a que `s` se crea dentro de `dangle`, cuando el código de `dangle`
se termina,`s` será desasignado. Pero tratamos de devolverle una referencia.
Eso significa que esta referencia estaría apuntando a un `String` inválido
¡Eso no es bueno! Rust no nos dejará hacer esto.

La solución aquí es devolver el `String` directamente:

```rust
fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```

Esto funciona sin ningún problema. La propiedad se trasladó y no se desasignó nada.

### Las reglas de las referencias

Repasemos lo que hemos visto sobre las referencias:

* En cualquier momento dado, puede tener *cualquiera de los dos* 
  una referencia mutable *o*
  cualquier cantidad de referencias inmutables.
* Las referencias siempre deben ser válidas.

A continuación, veremos un tipo diferente de referencia: *slices*.
