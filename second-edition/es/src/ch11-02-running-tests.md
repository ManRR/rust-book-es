## Controlar cómo se ejecutan las pruebas

Así como `cargo run` compila su código y luego ejecuta el binario resultante,`cargo test` compila su código en modo de prueba y ejecuta el binario de
prueba resultante. Puede especificar opciones de línea de comando para
cambiar el comportamiento predeterminado de `cargo test`. Por ejemplo, el
comportamiento predeterminado del binario producido por `cargo test` es
ejecutar todas las pruebas en paralelo y capturar la salida generada durante
las pruebas, evitando que se muestre la salida y facilitando la lectura de
los resultados de las pruebas.

Algunas opciones de línea de comando van a `cargo test`, y algunas van al
binario de prueba resultante. Para separar estos dos tipos de argumentos,
enumere los argumentos que van a `cargo test` seguido por el separador`--` y
luego a los que van al examen binario. Ejecutando `cargo test --help` muestra
las opciones que puede usar con `cargo test`, y ejecutar
`cargo test - --help` muestra las opciones que puede usar después del
separador `--`.

### Ejecutar Pruebas en Paralelo o Consecutivamente

Cuando ejecuta pruebas múltiples, de forma predeterminada se ejecutan en
paralelo utilizando subprocesos. Esto significa que las pruebas terminarán de
ejecutarse más rápido para que pueda obtener retroalimentación más rápido
sobre si su código está funcionando o no. Debido a que las pruebas se están
ejecutando al mismo tiempo, asegúrese de que sus pruebas no dependan entre sí
ni en ningún estado compartido, incluido un entorno compartido, como el
directorio de trabajo actual o las variables de entorno.

Por ejemplo, supongamos que cada una de sus pruebas ejecuta algún código que
crea un archivo en el disco llamado *test-output.txt* y escribe algunos datos
en ese archivo. Luego, cada prueba lee los datos en ese archivo y afirma que
el archivo contiene un valor particular, que es diferente en cada prueba.
Debido a que las pruebas se ejecutan al mismo tiempo, una prueba puede
sobrescribir el archivo cuando otra prueba escribe y lee el archivo. La
segunda prueba fallará, no porque el código sea incorrecto, sino porque las
pruebas se han interferido entre sí mientras se ejecutaban en paralelo. Una
solución es asegurarse de que cada prueba escriba en un archivo diferente;
otra solución es ejecutar las pruebas de a una por vez.

Si no desea ejecutar las pruebas en paralelo o si desea un control más
preciso sobre el número de subprocesos utilizados, puede enviar el indicador
`--test-threads` y el número de subprocesos que desea utilizar para el
binario de prueba. Eche un vistazo al siguiente ejemplo:

```text
$ cargo test -- --test-threads=1
```

Establecemos el número de hilos de prueba en `1`, diciéndole al programa que
no use ningún paralelismo. Ejecutar las pruebas con un hilo llevará más
tiempo que ejecutarlas en paralelo, pero las pruebas no interferirán entre sí
si comparten el estado.

### Mostrando salida de la función

De forma predeterminada, si pasa una prueba, la biblioteca de pruebas de Rust
captura todo lo impreso en la salida estándar. Por ejemplo, si llamamos
`println!`. En una prueba y la prueba pasa, no veremos la salida `println!` En
la terminal; solo veremos la línea que indica que pasó la prueba. Si falla
una prueba, veremos todo lo que se imprimió en la salida estándar con el
resto del mensaje de falla.

Como ejemplo, el Listado 11-10 tiene una función tonta que imprime el valor
de su parámetro y devuelve 10, así como una prueba que pasa y una prueba que
falla.

<span class="filename">Filename: src/lib.rs</span>

```rust
fn prints_and_returns_10(a: i32) -> i32 {
    println!("I got the value {}", a);
    10
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn this_test_will_pass() {
        let value = prints_and_returns_10(4);
        assert_eq!(10, value);
    }

    #[test]
    fn this_test_will_fail() {
        let value = prints_and_returns_10(8);
        assert_eq!(5, value);
    }
}
```

<span class="caption">Listado 11-10: Pruebas para una función que llama a
`println!`</span>

Cuando ejecutamos estas pruebas con `cargo test`, veremos el siguiente
resultado:

```text
running 2 tests
test tests::this_test_will_pass ... ok
test tests::this_test_will_fail ... FAILED

failures:

---- tests::this_test_will_fail stdout ----
        I got the value 8
thread 'tests::this_test_will_fail' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.

failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

Tenga en cuenta que en ninguna parte de esta salida vemos `I got the value 4`
que es lo que se imprime cuando se ejecuta la prueba que se ejecuta. Esa
salida ha sido capturada. La salida de la prueba que falló,
`I got the value 8`, aparece en la sección del resultado del resumen de la
prueba, que también muestra la causa de la falla de la prueba.

Si también queremos ver valores impresos para pasar pruebas, podemos
inhabilitar el comportamiento de captura de salida utilizando el indicador
`--nocapture`:

```text
$ cargo test -- --nocapture
```

Cuando ejecutamos nuevamente las pruebas en el Listado 11-10 con el indicador
`--nocapture`, vemos el siguiente resultado:

```text
running 2 tests
I got the value 4
I got the value 8
test tests::this_test_will_pass ... ok
thread 'tests::this_test_will_fail' panicked at 'assertion failed: `(left == right)`
  left: `5`,
 right: `10`', src/lib.rs:19:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.
test tests::this_test_will_fail ... FAILED

failures:

failures:
    tests::this_test_will_fail

test result: FAILED. 1 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out
```

Tenga en cuenta que la salida de las pruebas y los resultados de la prueba
están intercalados; la razón es que las pruebas se ejecutan en paralelo, como
hablamos en la sección anterior. Intenta usar la opción
`--test-threads = 1` y el indicador `--nocapture`, ¡y mira a qué se parece la
salida.

### Ejecutar un subconjunto de pruebas por nombre

A veces, ejecutar un conjunto de prueba completo puede llevar mucho tiempo.
Si está trabajando en un código en un área particular, es posible que desee
ejecutar solo las pruebas correspondientes a ese código. Puede elegir qué
pruebas ejecutar pasando 'prueba de carga' el nombre o los nombres de la
prueba(s) que desea ejecutar como argumento.

Para demostrar cómo ejecutar un subconjunto de pruebas, crearemos tres
pruebas para nuestra función `add_two`, como se muestra en el Listado 11-11,
y elegiremos cuáles ejecutar.

<span class="filename">Filename: src/lib.rs</span>

```rust
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn add_two_and_two() {
        assert_eq!(4, add_two(2));
    }

    #[test]
    fn add_three_and_two() {
        assert_eq!(5, add_two(3));
    }

    #[test]
    fn one_hundred() {
        assert_eq!(102, add_two(100));
    }
}
```

<span class="caption">Listado 11-11: Tres pruebas con tres nombres
diferentes</span>

Si ejecutamos las pruebas sin pasar ningún argumento, como vimos anteriormente, todas las pruebas se ejecutarán en paralelo:

```text
running 3 tests
test tests::add_two_and_two ... ok
test tests::add_three_and_two ... ok
test tests::one_hundred ... ok

test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

#### Ejecución de pruebas individuales

Podemos pasar el nombre de cualquier función de prueba a `cargo test` para ejecutar solo esa prueba:

```text
$ cargo test one_hundred
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-06a75b4a1f2515e9

running 1 test
test tests::one_hundred ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 2 filtered out
```

Solo se ejecutó la prueba con el nombre `one_hundred`; las otras dos pruebas
no coinciden con ese nombre. El resultado de la prueba nos permite saber que
tuvimos más pruebas de las que ejecutaba este comando al mostrar
`2 filtradas` al final de la línea de resumen.

No podemos especificar los nombres de múltiples pruebas de esta manera; solo
se usará el primer valor dado a `cargo test`. Pero hay una forma de ejecutar múltiples pruebas.

#### Filtrado para ejecutar múltiples pruebas

Podemos especificar parte del nombre de una prueba y se ejecutará cualquier
prueba cuyo nombre coincida con ese valor. Por ejemplo, debido a que dos de
los nombres de nuestras pruebas contienen `add`, podemos ejecutar esos dos
ejecutando `cargo test add`:

```text
$ cargo test add
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-06a75b4a1f2515e9

running 2 tests
test tests::add_two_and_two ... ok
test tests::add_three_and_two ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out
```

Este comando ejecutó todas las pruebas con `add` en el nombre y filtró la
prueba llamada `one_hundred`. También tenga en cuenta que el módulo en el que
aparecen las pruebas se convierte en parte del nombre de la prueba, por lo
que podemos ejecutar todas las pruebas en un módulo filtrando el nombre del
módulo.

### Ignorar algunas pruebas a menos que se solicite específicamente

Algunas veces, ejecutar algunas pruebas específicas puede llevar mucho tiempo
por lo que es posible que desee excluirlas durante la mayoría de las
ejecuciones de `cargo test`. En lugar de enumerar como argumentos todas las
pruebas que desea ejecutar, puede anotar las pruebas que consumen tiempo
utilizando el atributo `ignore` para excluirlas, como se muestra aquí:

<span class="filename">Filename: src/lib.rs</span>

```rust
#[test]
fn it_works() {
    assert_eq!(2 + 2, 4);
}

#[test]
#[ignore]
fn expensive_test() {
    // code that takes an hour to run
}
```

Después de `#[prueba]`, agregamos la línea `#[ignore]` a la prueba que
queremos excluir. Ahora cuando ejecutamos nuestras pruebas, `it_works` se
ejecuta, pero `expensive_test` no:

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.24 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 2 tests
test expensive_test ... ignored
test it_works ... ok

test result: ok. 1 passed; 0 failed; 1 ignored; 0 measured; 0 filtered out
```

La función `expensive_test` se muestra como `ignored`. Si queremos ejecutar
solo las pruebas ignoradas, podemos usar `cargo test -- --ignored`:

```text
$ cargo test -- --ignored
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-ce99bcc2479f4607

running 1 test
test expensive_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 1 filtered out
```

Al controlar qué pruebas se ejecutan, puede asegurarse de que los resultados
de su `cargo test` sean rápidos. Cuando estás en un punto en el que tiene
sentido comprobar los resultados de las pruebas `ignored` y tienes tiempo
para esperar los resultados, puedes ejecutar `cargo test -- --ignored`
en su lugar.
