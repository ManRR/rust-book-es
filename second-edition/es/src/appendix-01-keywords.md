## Apéndice A: Palabras clave

La siguiente lista contiene palabras clave que están reservadas para uso
actual o futuro por el idioma Rust. Como tales, no se pueden usar como
identificadores, como nombres de funciones, variables, parámetros, campos de estructura, módulos, *crates*, constantes, macros, valores estáticos
atributos, tipos, *traits* o *lifetimes*.

### Palabras clave actualmente en uso

Las siguientes palabras clave actualmente tienen la funcionalidad descrita.

* `as` - realiza una conversión primitiva, elimina la ambigüedad del *trait*
  específico que contiene
  un elemento, o cambiar el nombre de los elementos en las declaraciones `use` y `extern crate`
* `break` - salir de un bucle inmediatamente
* `const` - define elementos constantes o *raw pointers* constantes
* `continue` - continúa a la siguiente iteración de bucle
* `cajón` - enlaza un *crate* externa o una variable macro que representa el
 *crate* en
  que la macro está definida
* `else` - repliegue para construcciones de flujo de control `if` y `if let`
* `enum` - define una enumeración
* `extern`: vincula un *crate*, función o variable externa
* `false` - Boolean false literal
* `fn` - define una función o el tipo de puntero a función
* `for` - itera sobre elementos de un iterador, implementa un *trait* o
 especifica un
  *higher-ranked lifetime*
* `if` - bifurcación basada en el resultado de una expresión condicional
* `impl` - implementar funcionalidad inherente o de *trait*
* `in` - parte de la sintaxis de bucle `for`
* `let` - enlazar una variable
* `loop` - loop incondicionalmente
* `match` - relaciona un valor con los patrones
* `mod` - define un módulo
* `move` - hacer un *closure* tomar posesión de todas sus capturas
* `mut` - denota mutabilidad en referencias, *raw pointers* o enlaces de
 patrones
* `pub` - denotan visibilidad pública en campos struct, bloques `impl` o
 módulos
* `ref` - enlazar por referencia
* `return` - regreso de la función
* `Self` - un alias tipo para el tipo que implementa un *trait*
* `self` - tema del método o módulo actual
* `static` - variable global o *lifetime* que dura toda la ejecución del
 programa
* `struct` - define una estructura
* `super` - módulo principal del módulo actual
* `trait` - define un *trait*
* `true` - Boolean verdadero literal
* `type` - define un tipo de alias o tipo asociado
* `unsafe` - denota código inseguro, funciones, *traits* o implementaciones
* `use` - importar símbolos en el alcance
* `where` - denote cláusulas que restringen un tipo
* `while` - loop condicionalmente basado en el resultado de una expresión

### Palabras clave reservadas para uso futuro

Las siguientes palabras clave no tienen ninguna funcionalidad, pero están
reservadas por Rust para un posible uso futuro.

* `abstract`
* `alignof`
* `become`
* `box`
* `do`
* `final`
* `macro`
* `offsetof`
* `override`
* `priv`
* `proc`
* `pure`
* `sizeof`
* `typeof`
* `unsized`
* `virtual`
* `yield`
