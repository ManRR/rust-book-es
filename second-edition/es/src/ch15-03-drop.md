## Ejecutando código de limpieza con el *trait* `Drop`

El segundo *trait* importante para el patrón de puntero inteligente es
`Drop`, que permite personalizar lo que sucede cuando un valor está a punto
de quedar fuera del alcance. Usted puede proporcionar una implementación para
el *trait* `Drop` en cualquier tipo, y el código que especifique se puede
usar para liberar recursos como archivos o conexiones de red. Estamos
introduciendo `Drop` en el contexto de punteros inteligentes porque la
funcionalidad del *trait* `Drop` casi siempre se usa cuando se implementa un
puntero inteligente. Por ejemplo, `Box<T>` personaliza `Drop` para desasignar
el espacio en el montículo al que apunta el *box*.

En algunos lenguajes, el programador debe llamar al código para liberar
memoria o recursos cada vez que terminan de usar una instancia de un puntero
inteligente. Si olvidan, el sistema puede sobrecargarse y bloquearse. En Rust puede especificar que un poco de código particular se ejecuta cada vez que un
valor sale del alcance, y el compilador insertará este código
automáticamente. Como resultado, no necesita tener cuidado con la colocación
del código de limpieza en todas partes de un programa que una instancia de un
tipo en particular haya finalizado, ¡de todas maneras no perderá recursos!.

Especifique el código que se ejecutará cuando un valor salga del alcance
implementando el *trait* `Drop`. El *trait* `Drop` requiere que implemente un
método llamado `drop` que toma una referencia mutable a `self`. Para ver
cuando Rust llama `drop`, implementemos `drop` con las sentencias
`println!` por ahora.

El listado 15-14 muestra una estructura `CustomSmartPointer` cuya única
funcionalidad personalizada es que imprimirá
`Dropping CustomSmartPointer!` cuando la instancia se salga del alcance. Este
ejemplo demuestra cuando Rust ejecuta la función `drop`.

<span class="filename">Filename: src/main.rs</span>

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer { data: String::from("my stuff") };
    let d = CustomSmartPointer { data: String::from("other stuff") };
    println!("CustomSmartPointers created.");
}
```

<span class="caption">Listado 15-14: Una estructura `CustomSmartPointer` que
implementa el *trait* `Drop` donde pondríamos nuestro código de
limpieza</span>

El *trait* `Drop` está incluido en el preludio, por lo que no es necesario
importarlo. Implementamos el *trait* `Drop` en `CustomSmartPointer` y
proporcionamos una implementación para el método `drop` que llama
`println!`. El cuerpo de la función `drop` es donde colocaría cualquier
lógica que quisiera ejecutar cuando una instancia de su tipo saliera del
alcance. Estamos imprimiendo algunos textos aquí para demostrar cuándo Rust
llamará `drop`.

En `main`, creamos dos instancias de `CustomSmartPointer` y luego imprimimos
`CustomSmartPointers created`. Al final de `main`, nuestras instancias de
`CustomSmartPointer` saldrán del alcance, y Rust llamará al código que
colocamos en el método `drop`, imprimiendo nuestro mensaje final. Tenga en
cuenta que no necesitamos llamar al método `drop` explícitamente.

Cuando ejecutamos este programa, veremos el siguiente resultado:

```text
CustomSmartPointers created.
Dropping CustomSmartPointer with data `other stuff`!
Dropping CustomSmartPointer with data `my stuff`!
```

Rust automáticamente nos llamó `drop` cuando nuestras instancias salieron del
alcance, llamando al código que especificamos. Las variables se eliminan en
el orden inverso de su creación, por lo que `d` se eliminó antes de `c`. Este
ejemplo le da una guía visual de cómo funciona el método `drop`; por lo
general, debe especificar el código de limpieza que debe ejecutar su tipo en
lugar de un mensaje de impresión.

### Dropping un valor temprano con `std::mem::drop`

Desafortunadamente, no es fácil deshabilitar la funcionalidad automática de `drop`. Desactivar `drop` no suele ser necesario; el objetivo del *trait* `Drop` es que se soluciona automáticamente. Ocasionalmente, sin embargo, es
posible que desee limpiar un valor temprano. Un ejemplo es cuando se usan
punteros inteligentes que administran *locks*: es posible que desee forzar el
método `drop` que libera el *locks* para que otro código en el mismo alcance
pueda adquirir el *locks*. Rust no te permite llamar al método `drop` del
*trait* `Drop` de forma manual; en su lugar, debe llamar a la función
`std::mem::drop` proporcionada por la biblioteca estándar si desea forzar la
eliminación de un valor antes de que termine su ámbito.

Si tratamos de llamar manualmente el método `drop` del *trait* `Drop`
modificando la función `main` del Listado 15-14, como se muestra en el
Listado 15-15, obtendremos un error del compilador:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let c = CustomSmartPointer { data: String::from("some data") };
    println!("CustomSmartPointer created.");
    c.drop();
    println!("CustomSmartPointer dropped before the end of main.");
}
```

<span class="caption">Listado 15-15: Intentando llamar al método `drop` del
*trait* `Drop` manualmente para limpiar temprano</span>

Cuando intentemos compilar este código, obtendremos este error:

```text
error[E0040]: explicit use of destructor method
  --> src/main.rs:14:7
   |
14 |     c.drop();
   |       ^^^^ explicit destructor calls not allowed
```

Este mensaje de error indica que no podemos llamar `drop` explícitamente. El
mensaje de error utiliza el término *destructor*, que es el término de
programación general para una función que limpia una instancia. A
*destructor* es análogo a *constructor*, que crea una instancia. La función
`drop` en Rust es un destructor particular.

Rust no nos permite llamar `drop` explícitamente porque Rust todavía llamaría
automáticamente `drop` sobre el valor al final de `main`. Este sería un error
*double free* porque Rust estaría intentando limpiar el mismo valor dos veces.

No podemos deshabilitar la inserción automática de `drop` cuando un valor
está fuera del alcance, y no podemos llamar al método `drop` explícitamente.
Entonces, si necesitamos forzar que un valor se elimine temprano, podemos
usar la función `std::mem::drop`.

La función `std::mem::drop` es diferente del método `drop` en el *trait* `Drop`. Lo llamamos pasando el valor que queremos forzar para descartarlo
temprano como argumento. La función está en el preludio, por lo que podemos
modificar `main` en el Listado 15-15 para llamar a la función `drop`, como se
muestra en el Listado 15-16:

<span class="filename">Filename: src/main.rs</span>

```rust
# struct CustomSmartPointer {
#     data: String,
# }
#
# impl Drop for CustomSmartPointer {
#     fn drop(&mut self) {
#         println!("Dropping CustomSmartPointer!");
#     }
# }
#
fn main() {
    let c = CustomSmartPointer { data: String::from("some data") };
    println!("CustomSmartPointer created.");
    drop(c);
    println!("CustomSmartPointer dropped before the end of main.");
}
```

<span class="caption">Listado 15-16: Llamar a `std::mem::drop` para soltar
explícitamente un valor antes de que salga del alcance</span>

Al ejecutar este código, se imprimirá lo siguiente:

```text
CustomSmartPointer created.
Dropping CustomSmartPointer with data `some data`!
CustomSmartPointer dropped before the end of main.
```

El texto ```Quitando CustomSmartPointer con data` some data`! ``` Se imprime
entre el `CustomSmartPointer created` y `CustomSmartPointer drop before the end of main` texto, mostrando que se llama en al código del método `drop`
para soltar `c` en ese punto.

Puede usar el código especificado en una implementación del *trait* `Drop`
de muchas maneras para hacer la limpieza conveniente y segura: por ejemplo,
¡podría usarla para crear su propio asignador de memoria! Con el *trait*
`Drop` y el sistema de propiedad de Rust, no tienes que acordarte de limpiar
porque Rust lo hace automáticamente.

Tampoco tiene que preocuparse por los problemas que resultan de la limpieza
accidental de los valores que todavía están en uso: el sistema de propiedad
que asegura que las referencias sean siempre válidas también asegura que
`drop` se llame solo una vez cuando el valor ya no se use.

Ahora que hemos examinado `Box<T>` y algunas de las características de los
punteros inteligentes, veamos algunos otros punteros inteligentes definidos
en la biblioteca estándar.