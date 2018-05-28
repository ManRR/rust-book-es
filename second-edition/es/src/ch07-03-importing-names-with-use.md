## Referencia a los nombres en diferentes módulos

Hemos cubierto cómo llamar a las funciones definidas dentro de un módulo
usando el nombre del módulo como parte de la llamada, como en la llamada a la
función `nested_modules` que se muestra aquí en el Listado 7-7.

<span class="filename">Filename: src/main.rs</span>

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

fn main() {
    a::series::of::nested_modules();
}
```

<span class="caption">Listing 7-7: Llamar a una función especificando
completamente la ruta de su módulo envolvente</span>

Como puede ver, hacer referencia al nombre completo puede ser bastante largo.
Afortunadamente, Rust tiene una palabra clave para hacer que estas llamadas
sean más concisas.

### Bringing los nombres al alcance con la palabra clave `use`

La palabra clave `use` de Rust acorta las llamadas de función prolongadas al
poner en el alcance los módulos de la función que desea llamar. Aquí hay un
ejemplo de cómo llevar el módulo `a::series::of` al ámbito raíz de un *binary
crate*:

<span class="filename">Filename: src/main.rs</span>

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of;

fn main() {
    of::nested_modules();
}
```

La línea `use a::series::of;` significa que en lugar de usar la ruta completa
`a::series::of` donde queremos referirnos al módulo `of`, podemos usar `of`.

La palabra clave `use` solo trae lo que hemos especificado en el alcance: no
pone a los hijos de los módulos en el alcance. Es por eso que todavía tenemos
que usar `of :: nested_modules` cuando queremos llamar a la función
`nested_modules`.

Podríamos haber elegido traer la función al ámbito especificando la función
en el `uso` de la siguiente manera:

```rust
pub mod a {
    pub mod series {
        pub mod of {
            pub fn nested_modules() {}
        }
    }
}

use a::series::of::nested_modules;

fn main() {
    nested_modules();
}
```

Hacerlo nos permite excluir todos los módulos y hacer referencia directamente
a la función.

Debido a que las enumeraciones también forman una especie de espacio de
nombres como módulos, podemos poner las variantes de una enumeración en el
alcance con `use` también. Para cualquier tipo de declaración `use`, si traes
varios elementos de un espacio de nombres al ámbito, puedes enumerarlos
usando llaver y comas en la última posición, como sigue:

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::{Red, Yellow};

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = TrafficLight::Green;
}
```

Aún estamos especificando el espacio de nombres `TrafficLight` para la variante `Green` porque no incluimos `Green` en la declaración `use`.

### Bringing Todos los nombres al alcance con un Glob

Para poner todos los elementos en un espacio de nombres en el alcance a la vez, podemos usar la sintaxis `*`, que se llama operador *glob*. Este ejemplo trae todas las variantes de una enumeración al alcance sin tener que enumerarlas específicamente:

```rust
enum TrafficLight {
    Red,
    Yellow,
    Green,
}

use TrafficLight::*;

fn main() {
    let red = Red;
    let yellow = Yellow;
    let green = Green;
}
```

El operador `*` traerá dentro del alcance todos los elementos visibles en el
espacio de nombres `TrafficLight`. Deberías usar globs con moderación: son
convenientes, pero un glob también podría atraer más elementos de los que
esperabas y causar conflictos de nombres.

### Usando `super` para acceder a un módulo primario

Como viste al principio de este capítulo, cuando creas un *library crate*,Cargo crea un módulo de `tests` para usted. Vamos a entrar en más detallessobre eso ahora. En su proyecto `communicator`, abra *src/lib.rs*:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub mod client;

pub mod network;

#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
}
```

El Capítulo 11 explica más sobre las pruebas, pero algunas partes de este
ejemplo deberían tener sentido ahora: tenemos un módulo llamado `tests` que
vive al lado de nuestros otros módulos y contiene una función llamada
`it_works`. Aunque hay anotaciones especiales, ¡el módulo `tests` es solo
otro módulo! Entonces nuestra jerarquía de módulos se ve así:

```text
communicator
 ├── client
 ├── network
 |   └── client
 └── tests
```

Las pruebas son para ejercitar el código dentro de nuestra biblioteca, así
que intentemos llamar a nuestra función `client::connect` desde esta función
`it_works`, aunque no vamos a verificar ninguna funcionalidad en este
momento. Esto no funcionará aún:

<span class="filename">Filename: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        client::connect();
    }
}
```

Ejecute las pruebas invocando el comando `cargo test`:

```text
$ cargo test
   Compiling communicator v0.1.0 (file:///projects/communicator)
error[E0433]: failed to resolve. Use of undeclared type or module `client`
 --> src/lib.rs:9:9
  |
9 |         client::connect();
  |         ^^^^^^ Use of undeclared type or module `client`
```

La compilación falló, pero ¿por qué? No necesitamos colocar `communicator::`
delante de la función, como hicimos en *src/main.rs*, porque definitivamente
estamos dentro del `communicator` *library crate* aquí. La razón es
que las rutas son siempre relativas al módulo actual, que aquí es `tests`. La
única excepción es en una declaración `use`, donde las rutas son relativas a
la raíz del cajón de forma predeterminada. ¡Nuestro módulo `tests` necesita
el módulo `client` en su alcance!

Entonces, ¿cómo recuperamos un módulo en la jerarquía del módulo para llamar
a la función `client::connect` en el módulo `tests`? En el módulo `tests`,
podemos usar dos puntos para que Rust sepa que queremos comenzar desde la
raíz y enumerar toda la ruta, como esta:

```rust,ignore
::client::connect();
```

O bien, podemos usar `super` para subir un módulo en la jerarquía de nuestro módulo actual, así:

```rust,ignore
super::client::connect();
```

Estas dos opciones no se ven tan diferentes en este ejemplo, pero si está más
profundo en una jerarquía de módulos, comenzar desde la raíz cada vez haría
que el código sea extenso. En esos casos, usar `super` para pasar del módulo
actual a módulos -sibling- es un buen atajo. Además, si ha especificado la
ruta desde la raíz en muchos lugares en su código y luego reorganiza sus
módulos moviendo un subárbol a otro lugar, terminará necesitando actualizar
la ruta en varios lugares, lo que sería tedioso.

También sería molesto tener que escribir `super::` en cada prueba, pero ya
has visto la herramienta para esa solución: `use`! La funcionalidad
`super::` cambia la ruta que le das a `use` para que sea relativa al módulo
padre en lugar de al módulo raíz.

Por estas razones, en el módulo `tests` especialmente,`use super::something`
suele ser la mejor solución. Entonces ahora nuestra prueba se ve así:

<span class="filename">Filename: src/lib.rs</span>

```rust
#[cfg(test)]
mod tests {
    use super::client;

    #[test]
    fn it_works() {
        client::connect();
    }
}
```

Cuando ejecutemos `cargo test` nuevamente, la prueba pasará, y la primeraparte de la salida del resultado de la prueba será la siguiente:

```text
$ cargo test
   Compiling communicator v0.1.0 (file:///projects/communicator)
     Running target/debug/communicator-92007ddb5330fa5a

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

## Resumen

¡Ahora conoces algunas técnicas nuevas para organizar tu código! Utilice
estas técnicas para agrupar la funcionalidad relacionada, evitar que los
archivos sean demasiado largos y presentar una API pública ordenada a los
usuarios de su biblioteca.

A continuación, veremos algunas estructuras de datos de colección en la
biblioteca estándar que puede usar en su código bonito y ordenado.
