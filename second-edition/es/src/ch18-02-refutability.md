## Refutability: Whether a Pattern Might Fail to Match

Los patrones vienen en dos formas: refutable e irrefutable. Patrones que
coincidirán para cualquier valor posible pasado son *irrefutables*. Un
ejemplo sería `x` en la declaración `let x = 5;` porque `x` coincide con
cualquier cosa y por lo tanto no puede fallar. Los patrones que pueden no
coincidir con algún posible valor son *refutable*. Un ejemplo sería
`Some(x)` en la expresión `if let Some (x) = a_value` porque si el valor en
la variable `a_value` es `None` en lugar de `Some`, el patrón` Some (x)` no coincidirá.

Los parámetros de función, declaraciones `let` y `for` *loops* solo pueden
aceptar patrones irrefutables, porque el programa no puede hacer nada
significativo cuando los valores no coinciden Las expresiones `if let` y
`while let` solo aceptan patrones refutables, porque por definición están
destinados a manejar posibles falla: la funcionalidad de un condicional está
en su capacidad para realizar diferente dependiendo del éxito o el fracaso.

En general, no debería preocuparse por la distinción entre refutable
y patrones irrefutables; sin embargo, necesitas estar familiarizado con el
concepto de refutabilidad para que pueda responder cuando lo ve en un mensaje
de error. En esos casos, deberá cambiar el patrón o la construcción con la
que está utilizando el patrón, dependiendo del comportamiento previsto del
código.

Veamos un ejemplo de lo que sucede cuando tratamos de usar un patrón refutable
donde Rust requiere un patrón irrefutable y viceversa. El listado 18-8
muestra una declaración `let`, pero para el patrón hemos especificado
`Some(x)`, un patrón refutable. Como era de esperar, este código no se
compilará.

```rust,ignore
let Some(x) = some_option_value;
```

<span class="caption">Listado 18-8: Intentando usar un patrón refutable con
`let`</span>

Si `some_option_value` era un valor `None`, no coincidiría con el patrón
`Some(x)`, lo que significa que el patrón es refutable. Sin embargo, la
instrucción `let` solo puede aceptar un patrón irrefutable porque no hay nada
válido que el código pueda hacer con un valor `None`. En tiempo de
compilación, Rust se quejará de que hemos intentado utilizar un patrón
refutable cuando se requiere un patrón irrefutable:

```text
error[E0005]: refutable pattern in local binding: `None` not covered
 -->
  |
3 | let Some(x) = some_option_value;
  |     ^^^^^^^ pattern `None` not covered
```

Como no cubrimos (y no pudimos cubrir) todos los valores válidos con el
patrón `Some(x)`, Rust produce correctamente un error de compilación.

Para solucionar el problema donde tenemos un patrón refutable donde se
necesita un patrón irrefutable, podemos cambiar el código que usa el patrón:
en lugar de usar `let`, podemos usar `if let`. Entonces, si el patrón no
coincide, el código omitirá el código en las llaves, dándole una forma de
continuar válidamente. El Listado 18-9 muestra cómo arreglar el código en el
Listado 18-8.

```rust
# let some_option_value: Option<i32> = None;
if let Some(x) = some_option_value {
    println!("{}", x);
}
```

<span class="caption">Listado 18-9: Usando `if let` y un bloque con patrones
refutables en vez de `let`</span>

¡Le hemos dado un código! Este código es perfectamente válido, aunque
significa que no podemos usar un patrón irrefutable sin recibir un error. Si
le damos a `if let` un patrón que siempre coincidirá, como `x`, como se
muestra en el Listado 18-10, no se compilará.

```rust,ignore
if let x = 5 {
    println!("{}", x);
};
```

<span class="caption">Listado 18-10: Intentando usar un patrón irrefutable
con `if let`</span>

Rust se queja de que no tiene sentido usar `if let` con un patrón irrefutable:

```text
error[E0162]: irrefutable if-let pattern
 --> <anon>:2:8
  |
2 | if let x = 5 {
  |        ^ irrefutable pattern
```

Por esta razón, los brazos de *match* deben usar patrones refutables, excepto
el último brazo, que debe coincidir con cualquier valor restante con un
patrón irrefutable. Rust nos permite usar un patrón irrefutable en un
`match` con un solo brazo, pero esta sintaxis no es particularmente útil y
podría ser reemplazada por una declaración `let` más simple.

Ahora que sabe dónde usar patrones y la diferencia entre patrones refutables
e irrefutables, cubramos toda la sintaxis que podemos usar para crear
patrones.
