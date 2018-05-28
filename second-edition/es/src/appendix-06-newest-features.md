# Apéndice F - Características más nuevas

Este apéndice documenta características que se han agregado a Rust estable
desde que se completó la parte principal del libro.

## Field init shorthand

Podemos inicializar una estructura de datos (struct, enum, union) con campos
con nombre, escribiendo `fieldname` como una abreviatura de `fieldname:
fieldname`. Esto permite una sintaxis compacta para la inicialización, con menos duplicación:

```rust
#[derive(Debug)]
struct Person {
    name: String,
    age: u8,
}

fn main() {
    let name = String::from("Peter");
    let age = 27;

    // Using full syntax:
    let peter = Person { name: name, age: age };

    let name = String::from("Portia");
    let age = 27;

    // Using field init shorthand:
    let portia = Person { name, age };

    println!("{:?}", portia);
}
```

## Regresando de los bucles

Uno de los usos de un `loop` es volver a intentar una operación que sabe que
puede fallar, como comprobar si un subproceso completó su trabajo. Sin
embargo, es posible que deba pasar el resultado de esa operación al resto de
su código. Si lo agrega a la expresión `break` que usa para detener el bucle,
lo devolverá el bucle roto:

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    assert_eq!(result, 20);
}
```

## Grupos anidados en declaraciones 'use'

Si tiene un árbol de módulos complejo con muchos submódulos diferentes y
necesita importar algunos elementos de cada uno, podría ser útil agrupar
todas las importaciones en la misma declaración para mantener su código
limpio y evitar repetir el nombre de los módulos base.

La declaración `use` admite la anidación para ayudarlo en esos casos, tanto
con importaciones simples como globales. Por ejemplo, estos fragmentos
importan `bar`, `Foo`, todos los elementos en `baz` y `Bar`:

```rust
# #![allow(unused_imports, dead_code)]
#
# mod foo {
#     pub mod bar {
#         pub type Foo = ();
#     }
#     pub mod baz {
#         pub mod quux {
#             pub type Bar = ();
#         }
#     }
# }
#
use foo::{
    bar::{self, Foo},
    baz::{*, quux::Bar},
};
#
# fn main() {}
```

## Rangos inclusivos

Anteriormente, cuando se usaba un rango (`..` o `...`) como expresión, tenía
que ser `..`, que es exclusivo del límite superior, mientras que los patrones
tenían que usar `...` , que incluye el límite superior. Ahora, `.. =` se
acepta como sintaxis para los rangos inclusivos tanto en el contexto de
expresión como de rango:

```rust
fn main() {
    for i in 0 ..= 10 {
        match i {
            0 ..= 5 => println!("{}: low", i),
            6 ..= 10 => println!("{}: high", i),
            _ => println!("{}: out of range", i),
        }
    }
}
```

La sintaxis `...` todavía se acepta en las coincidencias, pero no se acepta
en las expresiones. `.. =` debería ser preferido.

## Enteros de 128 bits

Rust 1.26.0 agregó primitivos enteros de 128 bits:

- `u128`: Un entero sin signo de 128 bits con rango [0, 2^128 - 1]
- `i128`: Un entero con signo de 128 bits con rango [-(2^127), 2^127 - 1]

Estas primitivas se implementan de manera eficiente a través del soporte
LLVM. Están disponibles incluso en plataformas que no admiten nativamente
enteros de 128 bits y se pueden usar como los otros tipos de enteros.

Estas primitivas pueden ser muy útiles para algoritmos que necesitan usar
enteros muy grandes de manera eficiente, como ciertos algoritmos
criptográficos.