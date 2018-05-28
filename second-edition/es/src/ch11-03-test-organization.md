## Organización de prueba

Como se mencionó al comienzo del capítulo, las pruebas son una disciplina
compleja, y diferentes personas usan terminología y organización diferentes.
La comunidad de Rust piensa en las pruebas en términos de dos categorías
principales: *pruebas unitarias* y *pruebas de integración*
(*unit tests* and *integration tests*). Las pruebas unitarias son pequeñas y
más enfocadas, probando un módulo por separado a la vez, y pueden probar
interfaces privadas. Las pruebas de integración son completamente externas a
su biblioteca y usan su código de la misma manera que cualquier otro código
externo, utilizando solo la interfaz pública y posiblemente ejerciendo
múltiples módulos por prueba.

Escribir ambos tipos de pruebas es importante para garantizar que las piezas
de su biblioteca estén haciendo lo que espera, por separado y juntas.

### Pruebas unitarias

El propósito de las pruebas unitarias es probar cada unidad de código
aisladamente del resto del código para identificar rápidamente dónde está el
código y qué no funciona como se espera. Colocará pruebas unitarias en el
directorio *src* de cada archivo con el código que están probando. La
convención es crear un módulo llamado `tests` en cada archivo para contener
las funciones de prueba y anotar el módulo con `cfg(test)`.

#### El Módulo de Pruebas y `#[cfg(prueba)]`

La anotación `#[cfg(prueba)]` en el módulo de pruebas le dice a Rust que
compile y ejecute el código de prueba solo cuando ejecuta `carga test`, no
cuando ejecuta `cargo build`. Esto ahorra tiempo de compilación cuando solo
desea construir la biblioteca y ahorra espacio en el artefacto compilado
resultante porque las pruebas no están incluidas. Verá que debido a que las
pruebas de integración van en un directorio diferente, no necesitan la
anotación `#[cfg(prueba)]`. Sin embargo, como las pruebas unitarias van en
los mismos archivos que el código, usará `#[cfg(prueba)]` para especificar
que no se deben incluir en el resultado compilado.

Recuerde que cuando generamos el nuevo proyecto `adder` en la primera sección
de este capítulo, Cargo generó este código para nosotros:

<span class="filename">Filename: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

Este código es el módulo de prueba generado automáticamente. El atributo
`cfg` significa *configuración* y le dice a Rust que el siguiente ítem solo
debería incluirse dado una determinada opción de configuración. En este caso,
la opción de configuración es `test`, que es proporcionada por Rust para
compilar y ejecutar pruebas. Al utilizar el atributo `cfg`, Cargo compila
nuestro código de prueba solo si ejecutamos activamente las pruebas con
`cargo test`. Esto incluye cualquier función auxiliar que pueda estar dentro
de este módulo, además de las funciones anotadas con `#[test]`.

#### Prueba de funciones privadas

Existe un debate dentro de la comunidad de pruebas sobre si las funciones
privadas deben o no evaluarse directamente, y otros idiomas dificultan o
imposibilitan la prueba de funciones privadas. Independientemente de la
ideología de prueba que adhiera, las reglas de privacidad de Rust le permiten
probar funciones privadas. Considere el código en el listado 11-12 con la
función privada `internal_adder`.

<span class="filename">Filename: src/lib.rs</span>

```rust
pub fn add_two(a: i32) -> i32 {
    internal_adder(a, 2)
}

fn internal_adder(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn internal() {
        assert_eq!(4, internal_adder(2, 2));
    }
}
```

<span class="caption">Listado 11-12: Prueba de una función privada</span>

Tenga en cuenta que la función `internal_adder` no está marcada como `pub`,
pero como las pruebas son solo código Rust y el módulo `tests` es simplemente
otro módulo, puede importar y llamar `internal_adder` en una prueba bien.
Si no crees que se deben probar las funciones privadas, no hay nada en Rust
que te obligue a hacerlo.

### Pruebas de integración

En Rust, las pruebas de integración son completamente externas a tu
biblioteca. Utilizan su biblioteca de la misma manera que lo haría con
cualquier otro código, lo que significa que solo pueden llamar a funciones
que forman parte de la API pública de su biblioteca. Su propósito es probar
si muchas partes de su biblioteca funcionan juntas correctamente. Las
unidades de código que funcionan correctamente por sí mismas podrían tener
problemas cuando están integradas, por lo que la cobertura de prueba del
código integrado también es importante. Para crear pruebas de integración,
primero necesita un directorio *tests*.

#### El directorio *tests*

Creamos un directorio *tests* en el nivel superior de nuestro directorio de
proyectos, junto a *src*. Cargo sabe buscar archivos de prueba de integración
en este directorio. Entonces podemos hacer tantos archivos de prueba como
queramos en este directorio, y Cargo compilará cada uno de los archivos como
una caja individual.

Vamos a crear una prueba de integración. Con el código en el listado 11-12
todavía en el archivo *src/lib.rs*, cree un directorio *tests*, cree un nuevo
archivo llamado *tests/integration_test.rs*, e ingrese el código en el
listado 11-13.

<span class="filename">Filename: tests/integration_test.rs</span>

```rust,ignore
extern crate adder;

#[test]
fn it_adds_two() {
    assert_eq!(4, adder::add_two(2));
}
```

<span class="caption">Una prueba de integración de una función
en el crate `adder`</span>

Hemos agregado `extern crate adder` en la parte superior del código, que no
necesitamos en las pruebas unitarias. La razón es que cada prueba en el
directorio `tests` es una caja separada, por lo que debemos importar nuestra
biblioteca a cada una de ellas.

No es necesario anotar ningún código en *tests/integration_test.rs* con
`#[cfg(prueba)]`. Cargo trata el directorio `tests` especialmente y compila
los archivos en este directorio solo cuando ejecutamos `cargo test`. Ejecute
`cargo test` ahora:

```text
$ cargo test
   Compiling adder v0.1.0 (file:///projects/adder)
    Finished dev [unoptimized + debuginfo] target(s) in 0.31 secs
     Running target/debug/deps/adder-abcabcabc

running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/integration_test-ce99bcc2479f4607

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Las tres secciones de salida incluyen las pruebas unitarias, la prueba de
integración y las pruebas de doc. La primera sección para las pruebas
unitarias es la misma que hemos visto: una línea para cada prueba unitaria
(una llamada `internal` que agregamos en el Listado 11-12) y luego una línea
de resumen para las pruebas unitarias.

La sección de pruebas de integración comienza con la línea
`Running target/debug/deps/ integration test-ce99bcc2479f4607`
(el hash al final de su salida será diferente). A continuación, hay una línea
para cada función de prueba en esa prueba de integración y una línea de
resumen para los resultados de la prueba de integración justo antes de que
comience la sección `Doc-tests adder`.

De forma similar a cómo agregar más funciones de prueba unitarias agrega más
líneas de resultado a la sección de pruebas de la unidad, agregar más
funciones de prueba al archivo de prueba de integración agrega más líneas de
resultados a esta sección del archivo de prueba de integración. Cada archivo
de prueba de integración tiene su propia sección, por lo que si agregamos más
archivos en el directorio *tests*, habrá más secciones de prueba de integración.

Todavía podemos ejecutar una función de prueba de integración particular
especificando el nombre de la función de prueba como un argumento para
`cargo test`. Para ejecutar todas las pruebas en un archivo de prueba de
integración en particular, use el argumento `--test` de `cargo test` seguido
del nombre del archivo:

```text
$ cargo test --test integration_test
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/integration_test-952a27e0126bb565

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

Este comando ejecuta solo las pruebas en el archivo
*tests/integration_test.rs *.

#### Submódulos en pruebas de integración

A medida que agrega más pruebas de integración, es posible que desee crear
más de un archivo en el directorio *tests* para ayudar a organizarlos; por
ejemplo, puede agrupar las funciones de prueba por la funcionalidad que están
probando. Como se mencionó anteriormente, cada archivo en el directorio
*tests* se compila como su propio *crate* separado.

El tratamiento de cada archivo de prueba de integración como su propio
*crate* es útil para crear ámbitos separados que se parecen más a la forma en
que los usuarios finales usarán su *crate*. Sin embargo, esto significa que
los archivos en el directorio *tests* no comparten el mismo comportamiento
que los archivos en *src* hacen, como aprendió en el Capítulo 7 con respecto
a cómo separar el código en módulos y archivos.

El comportamiento diferente de los archivos en el directorio *tests* es más
notable cuando tiene un conjunto de funciones auxiliares que serían útiles en
múltiples archivos de prueba de integración e intenta seguir los pasos de la
sección “Mover módulos a otros archivos” del capítulo 7 para extraerlos en un módulo común. Por ejemplo, si creamos *tests/common.rs* y colocamos una
función llamada `setup`, podemos agregar un código a `setup` que queremos
llamar desde múltiples funciones de prueba en múltiples archivos de prueba:

<span class="filename">Filename: tests/common.rs</span>

```rust
pub fn setup() {
    // setup code specific to your library's tests would go here
}
```

Cuando ejecutemos las pruebas nuevamente, veremos una nueva sección en la
salida de prueba para el archivo *common.rs*, aunque este archivo no contenga
ninguna función de prueba ni llamemos a la función `setup` desde ningún lugar:

```text
running 1 test
test tests::internal ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/common-b8b07b6f1be2db70

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

     Running target/debug/deps/integration_test-d993c68b431d39df

running 1 test
test it_adds_two ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

`common` aparece en los resultados de la prueba con `running 0 tests` para
que no sea lo que queríamos. Solo queríamos compartir un código con los otros
archivos de prueba de integración.

Para evitar que aparezca `common` en la salida de prueba, en lugar de crear
*tests/common.rs*, crearemos *tests/common/mod.rs*. En la sección “Reglas de
los sistemas de archivos del módulo” del Capítulo 7, usamos la convención de
nomenclatura *nombre_módulo/mod.rs* para los archivos de los módulos que
tienen submódulos. No tenemos submódulos para 'comunes' aquí, pero nombrar el
archivo de esta manera le dice a Rust que no trate el módulo `common` como un
archivo de prueba de integración. Cuando movemos el código de la función
`setup` en *tests/common/mod.rs* y eliminamos el archivo *tests/common.rs*,
la sección en la salida de prueba ya no aparecerá. Los archivos en
subdirectorios del directorio *tests* no se compilan como *cretes* separadas
o tienen secciones en la salida de prueba.

Después de que hemos creado *tests/common/mod.rs*, podemos usarlo desde
cualquiera de los archivos de prueba de integración como un módulo. Aquí hay
un ejemplo de llamar a la función `setup` de la prueba `it_adds_two` en
*tests/integration_test.rs*:

<span class="filename">Filename: tests/integration_test.rs</span>

```rust,ignore
extern crate adder;

mod common;

#[test]
fn it_adds_two() {
    common::setup();
    assert_eq!(4, adder::add_two(2));
}
```

Tenga en cuenta que la declaración `mod common;` es la misma que las
declaraciones de módulo que demostramos en el listado 7-4. Luego, en la
función de prueba, podemos llamar a la función `common::setup ()`.

#### Pruebas de integración para *Binary Crates*

Si nuestro proyecto es una *binary crates* que solo contiene un archivo
*src/main.rs* y no tiene un archivo *src/lib.rs*, no podemos crear pruebas de
integración en el directorio *tests* y usar `extern crate` para importar
funciones definidas en el archivo *src/main.rs *. Solo los *crates* de la
biblioteca exponen funciones que otros *crates* pueden llamar y usar; los
*binary crates* se deben ejecutar por sí mismas.

Esta es una de las razones por las que los proyectos de Rust que proporcionan
un binario tienen un archivo *src/main.rs* sencillo que llama a la lógica que
vive en el archivo *src/lib.rs*. Usando esa estructura, las pruebas de
integración *pueden* probar el *crates* de la biblioteca usando
`extern crate` para ejercer la funcionalidad importante. Si la funcionalidad
importante funciona, la pequeña cantidad de código en el archivo
*src/main.rs* también funcionará, y esa pequeña cantidad de código no
necesita ser probada.

## Resumen

Las funciones de prueba de Rust proporcionan una manera de especificar cómo
debe funcionar el código para garantizar que continúe funcionando como espera
incluso cuando realice cambios. Las pruebas unitarias ejercitan diferentes
partes de una biblioteca por separado y pueden probar detalles privados de
implementación. Las pruebas de integración comprueban que muchas partes de la
biblioteca funcionen juntas correctamente, y usan la API pública de la
biblioteca para probar el código de la misma forma que el código externo lo
usará. A pesar de que el sistema de tipos y las reglas de propiedad de Rust
ayudan a prevenir algunos tipos de errores, las pruebas siguen siendo
importantes para reducir los errores de lógica que tienen que ver con la
forma en que se espera que su código se comporte.

¡Combinaremos el conocimiento que aprendió en este capítulo y en capítulos
anteriores para trabajar en un proyecto!.
