## Cómo escribir pruebas

Las pruebas son funciones de Rust que verifican que el código que no es
prueba está funcionando de la manera esperada. Los cuerpos de las funciones
de prueba suelen realizar estas tres acciones:

1. Configure cualquier información o estado necesario.
2. Ejecute el código que desea probar.
3. Afirma que los resultados son lo que esperas.

Veamos las características que Rust proporciona específicamente para escribir
pruebas que toman estas acciones, que incluyen el atributo `test`, algunas
macros y el atributo `should_panic`.

### La anatomía de una función de prueba

En su forma más simple, una prueba en Rust es una función anotada con el
atributo `test`. Los atributos son metadatos sobre piezas de código Rust; un
ejemplo es el atributo `derive` que usamos con las estructuras en el
Capítulo 5. Para cambiar una función a una función de prueba, agregue
`#[test]` en la línea antes de `fn`. Cuando ejecuta sus pruebas con el
comando `cargo test`, Rust construye un binario ejecutable de prueba que
ejecuta las funciones anotadas con el atributo `test` e informa si cada
función de prueba pasa o falla.

En el Capítulo 7, vimos que cuando realizamos un nuevo proyecto de biblioteca
con Cargo, automáticamente se genera para nosotros un módulo de prueba con
una función de prueba. Este módulo le ayuda a comenzar a escribir sus pruebas
para que no tenga que buscar la estructura exacta y la sintaxis de las
funciones de prueba cada vez que inicia un nuevo proyecto. ¡Puede agregar
tantas funciones de prueba adicionales como tantos módulos de prueba como
desee!

Exploraremos algunos aspectos de cómo funcionan las pruebas experimentando
con la prueba de plantilla generada por nosotros sin probar realmente ningún
código. Luego, escribiremos algunas pruebas del mundo real que llaman a algún
código que hemos escrito y afirman que su comportamiento es correcto.

Vamos a crear un nuevo proyecto de biblioteca llamado `adder`:

```text
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

El contenido del archivo *src/lib.rs* en su biblioteca `adder` debe parecerse
al Listado 11-1.

<span class="filename">Filename: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

<span class="caption">Listado 11-1: el módulo de prueba y la función generada
automáticamente por `cargo new`</span>

Por ahora, ignoremos las dos líneas superiores y nos centraremos en la
función para ver cómo funciona. Tenga en cuenta la anotación `#[test]` antes
de la línea `fn`: este atributo indica que se trata de una función de prueba,
por lo que el ejecutable de prueba sabe que debe tratar esta función como una
prueba. También podríamos tener funciones que no sean de prueba en el módulo
`tests` para ayudar a configurar escenarios comunes o realizar operaciones
comunes, por lo que debemos indicar qué funciones son pruebas utilizando el
atributo `#[test]`.

El cuerpo de la función utiliza la macro `assert_eq!` Para afirmar que 2 + 2
es igual a 4. Esta afirmación sirve como un ejemplo del formato para una
prueba típica. Vamos a ejecutarlo para ver que pasa esta prueba.

El comando `cargo test` ejecuta todas las pruebas en nuestro proyecto, como
se muestra en el Listado 11-2.

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.22 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

<span class="caption">Listado 11-2: El resultado de ejecutar la prueba
generada automáticamente</span>

Cargo compilado y ejecutado la prueba. Después de las líneas `Compiling`,
`Finished`, y `Running` está la línea `running 1 test`. La siguiente línea
muestra el nombre de la función de prueba generada, llamada `it_works`, y el
resultado de ejecutar esa prueba, `ok`. El resumen general de la ejecución de
las pruebas aparece a continuación. El texto `test result: ok` significa que
pasaron todas las pruebas y que pasó la porción que dice `1 passed; 0 failed`
totaliza el número de pruebas que pasaron o fallaron.

Como no tenemos ninguna prueba que hayamos marcado como ignorada, el resumen
muestra `0 ignorado`. Tampoco hemos filtrado las pruebas que se están
ejecutando, por lo que el final del resumen muestra `0 filtered out`.
Hablaremos sobre ignorar y filtrar las pruebas en la siguiente sección,
“Controlar cómo se ejecutan las pruebas”.

La estadística `0 measured` es para *benchmark tests* que miden el
rendimiento. *Benchmark tests*, al momento de escribir este
documento, solo disponibles en Rust *nightly*. Ver
[the documentation about benchmark tests][bench] para aprender más.

[bench]: ../../unstable-book/library-features/test.html

La siguiente parte de la salida de prueba, que comienza con `Doc-tests adder`,
es para los resultados de cualquier prueba de documentación. Todavía no
tenemos pruebas de documentación, pero Rust puede compilar cualquier ejemplo
de código que aparezca en nuestra documentación API. ¡Esta característica nos
ayuda a mantener nuestros documentos y nuestro código sincronizados!
Discutiremos cómo escribir pruebas de documentación en la sección
“Comentarios de la documentación como pruebas” del Capítulo 14. Por ahora,
ignoraremos el resultado de `Doc-tests`.

Cambiemos el nombre de nuestra prueba para ver cómo eso cambia la salida de
la prueba. Cambie la función `it_works` a un nombre diferente, como
`exploration`, como sigue:

<span class="filename">Filename: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }
}
```

Luego ejecute `cargo test` nuevamente. La salida ahora muestra `exploración` en lugar de `it_works`:

```text
running 1 test
test tests::exploration ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Agreguemos otra prueba, ¡pero esta vez haremos una prueba que falla! Las
pruebas fallan cuando algo en la función de prueba entra en pánico. Cada
prueba se ejecuta en un nuevo *hilo* (*thread*), y cuando el hilo principal
ve que un hilo de prueba ha muerto, la prueba se marca como fallida. Hablamos
sobre la forma más simple de causar pánico en el Capítulo 9, que es llamar a
la macro `panic!`. Ingrese la nueva prueba, `another`, para que su archivo
*src/lib.rs* se vea como el Listado 11-3.

<span class="filename">Filename: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    #[test]
    fn exploration() {
        assert_eq!(2 + 2, 4);
    }

    #[test]
    fn another() {
        panic!("Make this test fail");
    }
}
```

<span class="caption">Listado 11-3: Agregar una segunda prueba que fallará
porque llamamos a la macro `panic!`o</span>

Ejecute las pruebas nuevamente usando `cargo test`. La salida debería verse
como el Listado 11-4, que muestra que nuestra prueba `exploración` pasó y
`another` falló.

```text
running 2 tests
test tests::exploration ... ok
test tests::another ... FAILED

failures:

---- tests::another stdout ----
    thread 'tests::another' panicked at 'Make this test fail', src/lib.rs:10:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::another

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed
```

<span class="caption">Listado 11-4: resultados de la prueba cuando una prueba
pasa y una prueba falla</span>

En lugar de `ok`, la línea `test tests::another` muestra `FAILED`. Aparecen
dos nuevas secciones entre los resultados individuales y el resumen: la
primera sección muestra el motivo detallado de cada falla de prueba. En este
caso, `another` falló porque `panicked at 'Make this test fail'`, que sucedió
en la línea 10 en el archivo *src/lib.rs*. La siguiente sección enumera solo
los nombres de todas las pruebas que fallan, lo cual es útil cuando hay
muchas pruebas y muchos resultados de pruebas fallidas detalladas. Podemos
usar el nombre de una prueba que falla para ejecutar solo esa prueba para
depurarla más fácilmente; hablaremos más sobre las formas de ejecutar pruebas
en la sección “Controlar cómo se ejecutan las pruebas”.

La línea de resumen se muestra al final: en general, el resultado de nuestra
prueba es `FAILED`. Tuvimos un pase de prueba y una prueba falló.

Ahora que ha visto cómo son los resultados de las pruebas en diferentes
escenarios, veamos algunas macros además de `panic!` Que son útiles en las
pruebas.

### Comprobando los resultados con la macro `assert!`

La macro `assert!`, Provista por la biblioteca estándar, es útil cuando
quiere asegurarse de que alguna condición en una prueba se evalúe como
`true`. Le damos a la macro `assert!` Un argumento que evalúa a un booleano.
Si el valor es `true`,` assert! `No hace nada y la prueba pasa. Si el valor
es `false`, la macro `assert! `Llama a la macro `panic!`, Lo que hace que la
prueba falle. El uso de la macro `assert!` Nos ayuda a verificar que nuestro
código funcione de la manera que queremos.

En el Capítulo 5, Listado 5-15, usamos una estructura `Rectangle` y un método `can_hold`, que se repiten aquí en el Listado 11-5. Pongamos este código en el archivo *src/lib.rs* y escribamos algunas pruebas con la macro `assert!`.

<span class="filename">Filename: src/lib.rs</span>

```rust
# fn main() {}
#[derive(Debug)]
pub struct Rectangle {
    length: u32,
    width: u32,
}

impl Rectangle {
    pub fn can_hold(&self, other: &Rectangle) -> bool {
        self.length > other.length && self.width > other.width
    }
}
```

<span class="caption">Listado 11-5: Usando la estructura `Rectangle` y su
método `can_hold` del Capítulo 5</span>

El método `can_hold` devuelve un booleano, lo que significa que es un caso de
uso perfecto para la macro `assert! `. En el listado 11-6, escribimos una
prueba que ejerce el método `can_hold` creando una instancia `Rectangle` que
tiene una longitud de 8 y un ancho de 7 y afirma que puede contener otra
instancia `Rectangle` que tiene una longitud de 5 y un ancho de 1.

<span class="filename">Filename: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        let larger = Rectangle { length: 8, width: 7 };
        let smaller = Rectangle { length: 5, width: 1 };

        assert!(larger.can_hold(&smaller));
    }
}
```

<span class="caption">Listado 11-6: Una prueba para `can_hold` que verifica
si un rectángulo más grande puede contener un rectángulo más pequeño</span>

Tenga en cuenta que hemos agregado una nueva línea dentro del módulo
`tests`: `use super::*;`. El módulo `tests` es un módulo regular que sigue
las reglas de visibilidad habituales que cubrimos en el Capítulo 7 en la
sección “Reglas de privacidad” . Como el módulo `tests` es un módulo interno,
debemos poner el código bajo prueba en el módulo externo dentro del alcance
del módulo interno. Aquí usamos un glob para que todo lo que definimos en el
módulo externo esté disponible para este módulo `tests`.

Hemos nombrado a nuestra prueba `larger_can_hold_smaller`, y hemos creado las
dos instancias `Rectangle` que necesitamos. Luego llamamos a la macro
`assert!` Y la pasamos como resultado de llamar
`larger.can_hold(&smaller)`. Se supone que esta expresión devuelve `true`,
por lo que nuestra prueba debería pasar. ¡Vamos a averiguar!

```text
running 1 test
test tests::larger_can_hold_smaller ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

¡Pasa!, agreguemos otra prueba, esta vez afirmando que un rectángulo más
pequeño no puede contener un rectángulo más grande:

<span class="filename">Filename: src/lib.rs</span>

```rust
# fn main() {}
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn larger_can_hold_smaller() {
        // --snip--
    }

    #[test]
    fn smaller_cannot_hold_larger() {
        let larger = Rectangle { length: 8, width: 7 };
        let smaller = Rectangle { length: 5, width: 1 };

        assert!(!smaller.can_hold(&larger));
    }
}
```

Debido a que el resultado correcto de la función `can_hold` en este caso es
`false`, necesitamos negar ese resultado antes de pasarlo a la macro
`assert!`. Como resultado, nuestra prueba pasará si `can_hold` devuelve
`false`:

```text
running 2 tests
test tests::smaller_cannot_hold_larger ... ok
test tests::larger_can_hold_smaller ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

¡Dos pruebas que pasan! Ahora veamos qué sucede con los resultados de
nuestras pruebas cuando introducimos un error en nuestro código. Cambiemos la
implementación del método `can_hold` reemplazando el signo mayor que con un
signo menor que cuando se comparan las longitudes:

```rust
# fn main() {}
# #[derive(Debug)]
# pub struct Rectangle {
#     length: u32,
#     width: u32,
# }
// --snip--

impl Rectangle {
    pub fn can_hold(&self, other: &Rectangle) -> bool {
        self.length < other.length && self.width > other.width
    }
}
```

Ejecutar las pruebas ahora produce lo siguiente:

```text
running 2 tests
test tests::smaller_cannot_hold_larger ... ok
test tests::larger_can_hold_smaller ... FAILED

failures:

---- tests::larger_can_hold_smaller stdout ----
    thread 'tests::larger_can_hold_smaller' panicked at 'assertion failed:
    larger.can_hold(&smaller)', src/lib.rs:22:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::larger_can_hold_smaller

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

¡Nuestras pruebas captaron el error! Como `larger.length` es 8 y
`smaller.length` es 5, la comparación de las longitudes en `can_hold` ahora
devuelve `false`: 8 no es menor que 5.

### Probando la igualdad con las macros `assert_eq!` Y `assert_ne!`

Una forma común de probar la funcionalidad es comparar el resultado del
código bajo prueba con el valor que espera que devuelva el código para
asegurarse de que son iguales. Puede hacer esto usando la macro `assert!` Y
pasarle una expresión usando el operador `==`. Sin embargo, esta es una
prueba tan común que la biblioteca estándar proporciona un par de macros:
`assert_eq!` y `assert_ne!` para realizar esta prueba de manera más
conveniente. Estas macros comparan dos argumentos para igualdad o desigualdad,
respectivamente. También imprimirán los dos valores si la afirmación falla,
lo que hace que sea más fácil ver *por qué* la prueba falló; por el contrario
la macro `assert!` solo indica que obtuvo un valor `false` para la expresión
`==`, no los valores que conducen al valor `false`.

En el listado 11-7, escribimos una función llamada `add_two` que agrega `2` a
su parámetro y devuelve el resultado. Luego probamos esta función usando la
macro `assert_eq!`.

<span class="filename">Filename: src/lib.rs</span>

```rust
# fn main() {}
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_adds_two() {
        assert_eq!(4, add_two(2));
    }
}
```

<span class="caption">Listado 11-7: probando la función `add_two` usando la
macro `assert_eq!`</span>

¡Comprobemos que pase!

```text
running 1 test
test tests::it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

El primer argumento que le dimos a la macro `assert_eq!`, `4`, es igual al
resultado de invocar `add_two(2)`. La línea para esta prueba es
`test tests::it_adds_two ... ok`, y el texto `ok` indica que nuestra prueba
pasó.

Vamos a introducir un error en nuestro código para ver cómo funciona cuando
falla una prueba que usa `assert_eq!`. Cambie la implementación de la
función `add_two` para agregar `3` en su lugar:

```rust
# fn main() {}
pub fn add_two(a: i32) -> i32 {
    a + 3
}
```

Ejecute las pruebas nuevamente:

```text
running 1 test
test tests::it_adds_two ... FAILED

failures:

---- tests::it_adds_two stdout ----
        thread 'tests::it_adds_two' panicked at 'assertion failed: `(left == right)`
  left: `4`,
 right: `5`', src/lib.rs:11:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::it_adds_two

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

¡Nuestra prueba captó el error! La prueba `it_adds_two` falló, mostrando el
mensaje `` assertion failed: `(left == right)` `` y mostrando que `left` era
`4` y `right` era `5`. Este mensaje es útil y nos ayuda a comenzar la
depuración: significa el argumento `left` para `assert_eq!` era `4` pero el
argumento `right`, donde tenía `add_two (2)`, era `5`.

Tenga en cuenta que en algunos lenguajes y marcos de prueba, los parámetros
de las funciones que afirman que dos valores son iguales se llaman
`expected` y `real`, y el orden en que especificamos los argumentos es
importante. Sin embargo, en Rust, se llaman `left` y `right`, y el orden en
que especificamos el valor esperamos y el valor que produce el código bajo
prueba no importa. Nosotros podría escribir la afirmación en esta prueba
como `assert_eq! (add_two (2), 4)`, que daría como resultado un mensaje de
error que muestra `` assertion failed: `(left ==right) ``  y que `left` era
`5` y `right` era `4`.

La macro `assert_ne!` Pasará si los dos valores que le damos no son iguales y
fallar si son iguales Esta macro es más útil para casos en los que no
estamos seguros qué valor *será*, pero sabemos cuál es el valor
definitivamente *no* será si nuestro el código está funcionando como
queremos. Por ejemplo, si estamos probando una función que está garantizado
para cambiar su entrada de alguna manera, pero la forma en que la entrada
se cambia depende del día de la semana en que realizamos nuestras pruebas,
lo mejor afirmar podría ser que la salida de la función no es igual a la
entrada.

Debajo de la superficie, las macros `assert_eq!` y `assert_ne!` Usan los
operadores `==` y `! =`, respectivamente. Cuando las aserciones fallan,
estas macros imprimen su argumentos que utilizan el formato de depuración,
lo que significa que los valores que se comparan deben implementar los
*trait* `PartialEq` y `Debug`. Todos los tipos primitivos y la mayoría
de los tipos de biblioteca estándar implementan estos *trait*. Para
estructuras y enumeraciones que defina, necesitará implementar `PartialEq`
para afirmar que los valores de esos tipos son iguales o no iguales.
Necesitarás implementar `Debug` para imprimir los valores cuando la
afirmación falla. Debido a que ambos *trait* son *trait* derivables,
como se menciona en el Listado 5-12 en el Capítulo 5, esto suele ser tan
sencillo como agregar la anotación `#[derive (PartialEq, Debug)]` a su
definición *struct* o *enum*. Ver el Apéndice C para más detalles sobre estos y otros *trait* derivables.

### Agregar mensajes personalizados de fallo

También puede agregar un mensaje personalizado para imprimir con el mensaje
de fallo como argumentos opcionales para las macros `assert!`, `Assert_eq!`,
y `assert_ne!`. Cualquier argumento especificado después de un argumento
requerido para `assert!` o los dos argumentos requeridos para `assert_eq!` y
`assert_ne!` se pasan a la macro `format!` (Estudiado en el Capítulo 8 en la
sección “Concatenación con el operador `+` o la Macro `format!`”), por lo
que puede pasar una cadena de formato que contenga `{}` marcadores de
posición y valores para ir en esos marcadores de posición. Los mensajes
personalizados son útiles para documentar lo que significa una afirmación;
cuando falla una prueba, tendrá una mejor idea de cuál es el problema con el
código.

Por ejemplo, supongamos que tenemos una función que saluda a las personas
por su nombre y queremos probar que el nombre que pasamos a la función
aparezca en el resultado:

<span class="filename">Filename: src/lib.rs</span>

```rust
# fn main() {}
pub fn greeting(name: &str) -> String {
    format!("Hello {}!", name)
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn greeting_contains_name() {
        let result = greeting("Carol");
        assert!(result.contains("Carol"));
    }
}
```

Los requisitos para este programa aún no se han acordado, y estamos bastante
seguros de que el texto `Hello` al comienzo del saludo cambiará. Decidimos
que no queremos tener que actualizar la prueba cuando cambian los requisitos
así que en lugar de verificar la igualdad exacta del valor devuelto por la
función `greeting`, simplemente afirmaremos que la salida contiene el texto
de la entrada parámetro.

Vamos a introducir un error en este código cambiando `greeting` para no
incluir `name` para ver cómo se ve esta falla en la prueba:

```rust
# fn main() {}
pub fn greeting(name: &str) -> String {
    String::from("Hello!")
}
```

Ejecutar esta prueba produce lo siguiente:

```text
running 1 test
test tests::greeting_contains_name ... FAILED

failures:

---- tests::greeting_contains_name stdout ----
        thread 'tests::greeting_contains_name' panicked at 'assertion failed:
result.contains("Carol")', src/lib.rs:12:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::greeting_contains_name
```

Este resultado simplemente indica que la aserción falló y en qué línea está
la aserción. Un mensaje de falla más útil en este caso imprimiría el valor
que obtenemos de la función `greeting`. Cambiemos la función de prueba,
dándole un mensaje de falla personalizado hecho de una cadena de formato con
un marcador de posición rellenado con el valor real que obtuvimos de la
función `greeting`:

```rust,ignore
#[test]
fn greeting_contains_name() {
    let result = greeting("Carol");
    assert!(
        result.contains("Carol"),
        "Greeting did not contain name, value was `{}`", result
    );
}
```

Ahora cuando ejecutamos la prueba, obtendremos un mensaje de error más informativo:

```text
---- tests::greeting_contains_name stdout ----
        thread 'tests::greeting_contains_name' panicked at 'Greeting did not
contain name, value was `Hello!`', src/lib.rs:12:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

Podemos ver el valor que realmente obtuvimos en el resultado de la prueba,
lo que nos ayudaría a depurar lo que sucedió en lugar de lo que esperábamos
que sucediera.

### Comprobación de *pánicos* (*Panics*) con `should_panic`

Además de verificar que nuestro código arroje los valores correctos que
esperamos, también es importante verificar que nuestro código maneje las
condiciones de error como esperamos. Por ejemplo, considere el tipo `Guess`
que creamos en el Capítulo 9, Listado 9-9. Otro código que usa `Guess`
depende de la garantía de que las instancias `Guess` contendrán solo valores
entre 1 y 100. Podemos escribir una prueba que asegure que intentar crear
una instancia `Guess` con un valor fuera de ese rango entra en pánico.

Hacemos esto agregando otro atributo, `should_panic`, a nuestra función de
prueba. Este atributo hace pasar una prueba si el código dentro de la
función entra en pánico; la prueba fallará si el código dentro de la función
no entra en pánico.

El listado 11-8 muestra una prueba que verifica que las condiciones de error
de `Guess::new` ocurran cuando esperamos que lo hagan.

<span class="filename">Filename: src/lib.rs</span>

```rust
# fn main() {}
pub struct Guess {
    value: u32,
}

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 || value > 100 {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

<span class="caption">Listado 11-8: ¡Prueba que una condición causará un
`panic!`</span>

Colocamos el atributo `#[should_panic]` después del atributo `#[test]` y
antes de la función de prueba a la que se aplica. Miremos el resultado
cuando pase esta prueba:

```text
running 1 test
test tests::greater_than_100 ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

¡Se ve bien! Ahora introduzcamos un error en nuestro código eliminando la
condición de que la función `new` entrará en pánico si el valor es mayor que
100:

```rust
# fn main() {}
# pub struct Guess {
#     value: u32,
# }
#
// --snip--

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1  {
            panic!("Guess value must be between 1 and 100, got {}.", value);
        }

        Guess {
            value
        }
    }
}
```

Cuando ejecutamos la prueba en el listado 11-8, fallará:

```text
running 1 test
test tests::greater_than_100 ... FAILED

failures:

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

No obtenemos un mensaje muy útil en este caso, pero cuando miramos la
función de prueba, vemos que está anotada con `#[should_panic]`. El error
que obtuvimos significa que el código en la función de prueba no causó
pánico.

Las pruebas que usan `should_panic` pueden ser imprecisas porque solo
indican que el código ha causado cierto pánico. Una prueba `should_panic`
pasaría incluso si la prueba entra en pánico por una razón diferente a la
que esperábamos que sucediera. Para hacer las pruebas `should_panic` más
precisas, podemos agregar un parámetro `expected` opcional al atributo
`should_panic`. El arnés de prueba se asegurará de que el mensaje de falla
contenga el texto provisto. Por ejemplo, considere el código modificado para
`Guess` en el Listado 11-9 donde la función `new` entra en pánico con
diferentes mensajes dependiendo de si el valor es demasiado pequeño o
demasiado grande.

<span class="filename">Filename: src/lib.rs</span>

```rust
# fn main() {}
# pub struct Guess {
#     value: u32,
# }
#
// --snip--

impl Guess {
    pub fn new(value: u32) -> Guess {
        if value < 1 {
            panic!("Guess value must be greater than or equal to 1, got {}.",
                   value);
        } else if value > 100 {
            panic!("Guess value must be less than or equal to 100, got {}.",
                   value);
        }

        Guess {
            value
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
}
```

<span class="caption">Listado 11-9: ¡Prueba que una condición causará un
`panic!` Con un mensaje de pánico particular</span>

Esta prueba pasará porque el valor que ponemos en el parámetro `expected`
del atributo `should_panic` es una subcadena del mensaje con el que entra en
juego la función `Guess::new`. Podríamos haber especificado todo el mensaje
de pánico que esperamos, que en este caso sería
`Guess value must be less than or equal to 100, got 200.`. Lo que elija especificar en el parámetro esperado para `should_panic` depende de cómo
gran parte del mensaje de pánico es único o dinámico y qué tan preciso
quiere que sea su prueba. En este caso, una subcadena del mensaje de pánico
es suficiente para garantizar que el código en la función de prueba ejecuta
el caso `else if value> 100`.

Para ver qué sucede cuando falla una prueba `should_panic` con un mensaje
`expected`, volvamos a introducir un error en nuestro código intercambiando
los cuerpos de los bloques `if value <1` y `else if value> 100`:

```rust,ignore
if value < 1 {
    panic!("Guess value must be less than or equal to 100, got {}.", value);
} else if value > 100 {
    panic!("Guess value must be greater than or equal to 1, got {}.", value);
}
```

Esta vez, cuando ejecutamos la prueba `should_panic`, fallará:

```text
running 1 test
test tests::greater_than_100 ... FAILED

failures:

---- tests::greater_than_100 stdout ----
        thread 'tests::greater_than_100' panicked at 'Guess value must be
greater than or equal to 1, got 200.', src/lib.rs:11:12
note: Run with `RUST_BACKTRACE=1` for a backtrace.
note: Panic did not include expected string 'Guess value must be less than or
equal to 100'

failures:
    tests::greater_than_100

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

El mensaje de falla indica que esta prueba entró en pánico como esperábamos,
pero el mensaje de pánico no incluyó la cadena de texto esperada
`'Guess value must be less than or equal to 100'`. El mensaje de pánico que
obtuvimos en este caso fue
`Guess value must be greater than or equal to 1,got 200.`. ¡Ahora podemos
comenzar a descubrir dónde está nuestro error!.

Ahora que conoce varias formas de escribir pruebas, veamos qué está
sucediendo cuando ejecutemos nuestras pruebas y exploremos las diferentes
opciones que podemos usar con `cargo test`.
