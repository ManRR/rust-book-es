## Conciso control de flujo con `if let`

La sintaxis `if let` le permite combinar `if` y `let` de una manera menos detallada
para manejar valores que coinciden con un patrón sin tener en cuenta el resto.
Considere el programa en el listado 6-6 que coincide con un valor `Option <u8>`
pero solo quiere ejecutar el código si el valor es 3.

```rust
let some_u8_value = Some(0u8);
match some_u8_value {
    Some(3) => println!("three"),
    _ => (),
}
```

<span class="caption">Listing 6-6: Un `match` que solo se preocupa por ejecutar el
código cuando el valor es `Some(3)`</span>

Queremos hacer algo con la coincidencia `Some(3)` pero no hacemos nada con
ningún otro valor `Some <u8>` o el valor `None`. Para satisfacer la expresión `match`,
tenemos que agregar `_ => ()` después de procesar solo una variante, que es
una gran cantidad de código repetitivo para agregar.

En cambio, podríamos escribir esto de una manera más corta usando `if let`. El
siguiente código se comporta igual que el `match` en el Listado 6-6:

```rust
# let some_u8_value = Some(0u8);
if let Some(3) = some_u8_value {
    println!("three");
}
```

La sintaxis `if let` toma un patrón y una expresión separados por un signo igual.
Funciona de la misma manera que un `match`, donde la expresión se asigna al `match`
y el patrón es su primer brazo.

Usar `if let` significa menos tipeo, menos indentación y menos código repetitivo.
Sin embargo, pierdes la verificación exhaustiva de que `match` se aplica. Elegir
entre `match` y `if let` depende de lo que esté haciendo en su situación particular
y de si obtener concisión es una compensación adecuada para perder una verificación
exhaustiva.

En otras palabras, puede pensar en `if let` como azúcar sintáctico para una
`match` que ejecuta código cuando el valor coincide con un patrón y luego ignora
todos los demás valores.

Podemos incluir un `else` con un `if let`. El bloque de código que acompaña al
`else` es el mismo que el bloque de código que iría con el caso `_` en la expresión
`match` que es equivalente a `if let` y `else`. Recuerde la definición de la enumeración
`Coin` en el Listado 6-4, donde la variante `Quarter` también contiene un valor
`UsState`. Si quisiéramos contar todas las monedas que no son cuartos *quarters*
vemos al mismo tiempo que anunciamos el estado de los cuartos, podríamos hacer
eso con una expresión de `match` como esta:

```rust
# #[derive(Debug)]
# enum UsState {
#    Alabama,
#    Alaska,
# }
#
# enum Coin {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter(UsState),
# }
# let coin = Coin::Penny;
let mut count = 0;
match coin {
    Coin::Quarter(state) => println!("State quarter from {:?}!", state),
    _ => count += 1,
}
```

O podríamos usar una expresión `if let` y `else` como esta:

```rust
# #[derive(Debug)]
# enum UsState {
#    Alabama,
#    Alaska,
# }
#
# enum Coin {
#    Penny,
#    Nickel,
#    Dime,
#    Quarter(UsState),
# }
# let coin = Coin::Penny;
let mut count = 0;
if let Coin::Quarter(state) = coin {
    println!("State quarter from {:?}!", state);
} else {
    count += 1;
}
```

Si tiene una situación en la que su programa tiene una lógica que es demasiado
detallada para expresar usando un `match`, recuerde que`if let` está también
en su caja de herramientas de Rust.

## Resumen

Ahora hemos cubierto cómo usar las enumeraciones para crear tipos personalizados
que pueden ser uno de un conjunto de valores enumerados. Mostramos cómo el tipo
`Opción <T>` de la biblioteca estándar lo ayuda a usar el sistema de tipos para
evitar errores. Cuando los valores *enum* tienen datos dentro de ellos, puede usar
`match` o `if let` para extraer y usar esos valores, dependiendo de la cantidad
de casos que necesite manejar.

Sus programas de Rust ahora pueden expresar conceptos en su dominio usando estructuras
y enumeraciones. La creación de tipos personalizados para usar en su API garantiza
la seguridad del tipo: el compilador se asegurará de que sus funciones obtengan solo
valores del tipo que cada función espera.

Con el fin de proporcionar a sus usuarios una API bien organizada que sea fácil de
usar y solo exhiba exactamente lo que necesitarán sus usuarios, veamos ahora los
módulos de Rust.
