## To `panic!` or Not to `panic!`

Entonces, ¿cómo decides cuándo debes llamar `panic!` Y cuándo debes devolver
el `Result`?. Cuando el código entra en pánico, no hay forma de recuperarlo.
Puede llamar `panic!` por cualquier situación de error, ya sea que haya una
forma posible de recuperación o no, pero luego está tomando la decisión en
nombre del código llamando a su código de que una situación es irrecuperable. Cuando elige devolver un valor de `Result`, le da las opciones de código de
llamada en lugar de tomar una decisión al respecto. El código de llamada
podría optar por intentar la recuperación de una manera apropiada para su
situación, o podría decidir que un valor `Err` en este caso es irrecuperable,
por lo que puede llamar `¡pánico!` y convertir su error recuperable en uno
irrecuperable. Por lo tanto, devolver `Result` es una buena opción
predeterminada cuando se define una función que podría fallar.

En situaciones excepcionales, es más apropiado escribir un código que entra
en pánico en lugar de devolver un `Result`. Analicemos por qué es apropiado
entrar en pánico en ejemplos, códigos de prototipos y pruebas. Luego
discutiremos las situaciones en las que el compilador no puede decir que la
falla es imposible, pero usted, como ser humano, puede hacerlo. El capítulo
concluirá con algunas pautas generales sobre cómo decidir si entrar en pánico
en el código de la biblioteca.

### Ejemplos, código de prototipo y pruebas

Cuando está escribiendo un ejemplo para ilustrar algún concepto, tener un
código robusto de manejo de errores en el ejemplo también puede hacer que el
ejemplo sea menos claro. En los ejemplos, se entiende que una llamada a un
método como `unwrap` que podría entrar en pánico significa un marcador de
posición para la forma en que desearía que su aplicación manejara los errores,que pueden diferir en función de lo que haga el resto del código.

Del mismo modo, los métodos `unwrap` y `expect` son muy útiles al crear
prototipos, antes de que esté listo para decidir cómo manejar los errores.
Dejan marcas claras en su código para cuando esté listo para hacer que su
programa sea más robusto.

Si falla una llamada al método en una prueba, querría que fallara toda la
prueba, incluso si ese método no es la funcionalidad bajo prueba. Debido a
que `¡pánico!` Es la forma en que una prueba se marca como una falla, llamar
`unwrap` o `expect` es exactamente lo que debería suceder.

### Casos en los que tiene más información que el compilador

También sería apropiado llamar a `unwrap` cuando tenga alguna otra lógica que
asegure que `Result` tendrá un valor `Ok`, pero la lógica no es algo que el
compilador entienda. Todavía tendrá un valor de `Result` que debe manejar:
cualquier operación que llame todavía tiene la posibilidad de fallar en
general, aunque sea lógicamente imposible en su situación particular. Si
puede asegurarse al inspeccionar manualmente el código que nunca tendrá una
variante `Err`, es perfectamente aceptable llamar a `unwrap`. Aquí hay un
ejemplo:

```rust
use std::net::IpAddr;

let home: IpAddr = "127.0.0.1".parse().unwrap();
```

Estamos creando una instancia `IpAddr` al analizar una cadena codificada.
Podemos ver que `127.0.0.1` es una dirección IP válida, por lo que es
aceptable usar `unwrap` aquí. Sin embargo, tener un string codificado y
válido no cambia el tipo de retorno del método `parse`: todavía obtenemos un
valor `Result`, y el compilador todavía nos hará manejar el `Result` como si
fuera la variante `Err` es una posibilidad porque el compilador no es lo
suficientemente inteligente como para ver que esta cadena siempre es una
dirección IP válida. Si la cadena de dirección IP proviene de un usuario en
lugar de estar codificada en el programa y, por lo tanto, *sí* tiene una
posibilidad de falla, definitivamente queremos manejar el `Result` de una
manera más robusta.

### Pautas para el manejo de errores

Es aconsejable que su código entre en pánico cuando es posible que su código
termine en mal estado. En este contexto, un *mal estado* es cuando se han
roto algunas suposiciones, garantías, contratos o invariantes, como cuando se
pasan valores no válidos, valores contradictorios o valores perdidos a su
código, más uno o más de los siguientes:

* El mal estado no es algo *esperado* que ocurra ocasionalmente.
* Su código después de este punto necesita confiar en no estar en este mal
 estado.
* No hay una buena manera de codificar esta información en los tipos que usa.

Si alguien llama a su código y le envía valores que no tienen sentido, la
mejor opción podría ser invocar `panic!` Y alertar a la persona que utiliza
su biblioteca sobre el error en su código para que puedan solucionarlo
durante el desarrollo. De manera similar, `panic!` Es a menudo apropiado si
está llamando un código externo que está fuera de su control y devuelve un
estado no válido que no tiene forma de solucionar.

Cuando se alcanza un mal estado, pero se espera que suceda sin importar lo
bien que escribe tu código, aún es más apropiado devolver un `Result` en
lugar de hacer una llamada `pánico!` los ejemplos incluyen un analizador que
recibe datos malformados o una solicitud HTTP que devuelve un estado que
indica que ha alcanzado un límite de velocidad. En estos casos, debe indicar que la falla es una posibilidad esperada por devolver un `Result` para
propagar estos estados incorrectos hacia arriba para que el código de llamada
puede decidir cómo manejar el problema. ¡Llamar a `panic!`! No sería la mejor
manera para manejar estos casos.

Cuando su código realiza operaciones con valores, su código debe verificar que
los valores son válidos primero y el pánico si los valores no son válidos.
Esto es principalmente para razones de seguridad: intentar operar con datos
no válidos puede exponer su código a vulnerabilidades Esta es la razón
principal por la que la biblioteca estándar llamará `¡pánico!` si intenta
acceder a la memoria fuera de límites: tratando de acceder a la memoria
que no pertenece a la estructura de datos actual es un problema de seguridad
común. Las funciones a menudo tienen *contratos*: su comportamiento solo está
garantizado si las entradas cumplen requisitos particulares. Presa del pánico
cuando se viola el contrato tiene sentido porque una violación del contrato
siempre indica un error del lado de la persona que llama y no es un tipo de error que desea que el código de llamada tenga que explícitamente encargarse de. De hecho, no hay una forma razonable de invocar el código para recuperar;
los *programadores* que llaman necesitan arreglar el código. Contratos para una función, especialmente cuando una violación causará pánico, debe explicarse en la API de documentación para la función.

Sin embargo, tener muchos controles de errores en todas sus funciones sería
verbose y molesto. Afortunadamente, puede usar el sistema de tipos de Rust
(y, por lo tanto, el tipo de verificación que hace el compilador) para hacer
muchos de los controles por usted. Si su función tiene un tipo particular
como parámetro, puede continuar con la lógica de su código sabiendo que el
compilador ya se ha asegurado de que tiene un valor válido. Por ejemplo, si
tiene un tipo en lugar de una `Opción`, su programa espera tener *algo* en lugar de *nada*. Entonces, su código no tiene que manejar dos casos para las
variantes `Some` y `None`: solo tendrá un caso para definitivamente tener un
valor. El código que intenta no pasarle nada a tu función ni siquiera
compilará, por lo que tu función no tiene que verificar ese caso en el tiempo
de ejecución. Otro ejemplo es el uso de un tipo de entero sin signo como
`u32`, que asegura que el parámetro nunca sea negativo.

### Crear tipos personalizados para validación

Tomemos la idea de usar el sistema de tipos de Rust para garantizar que
tengamos un valor válido un paso más allá y busquemos la creación de un tipo
personalizado para la validación. Recuerde el juego de adivinanzas en el
Capítulo 2 en el cual nuestro código le pedía al usuario que adivinara un
número entre 1 y 100. Nunca validamos que la conjetura del usuario estuviera
entre esos números antes de verificarlo contra nuestro número secreto; solo
validamos que la suposición fue positiva. En este caso, las consecuencias no
fueron muy graves: nuestra salida de “Demasiado alto” o “Demasiado bajo”
seguiría siendo correcta. Pero sería una mejora útil para guiar al usuario
hacia conjeturas válidas y tener un comportamiento diferente cuando un
usuario adivina un número que está fuera de rango frente a cuando un usuario
escribe, por ejemplo, letras en su lugar.

Una forma de hacer esto sería analizar la conjetura como un `i32` en lugar de
solo un `u32` para permitir números potencialmente negativos, y luego agregar
un *check* para que el número esté dentro del rango, así:

```rust,ignore
loop {
    // --snip--

    let guess: i32 = match guess.trim().parse() {
        Ok(num) => num,
        Err(_) => continue,
    };

    if guess < 1 || guess > 100 {
        println!("The secret number will be between 1 and 100.");
        continue;
    }

    match guess.cmp(&secret_number) {
    // --snip--
}
```

La expresión `if` verifica si nuestro valor está fuera de rango, le dice al
usuario sobre el problema y llama a `continue` para comenzar la siguiente
iteración del ciclo y pedir otra aproximación. Después de la expresión `if`, podemos proceder con las comparaciones entre `guess` y el número secreto
sabiendo que `guess` está entre 1 y 100.

Sin embargo, esta no es una solución ideal: si era absolutamente crítico que
el programa solo operara con valores entre 1 y 100, y tenía muchas funciones
con este requisito, tener un check como este en cada función sería tedioso
(y podría afectar actuación).

En cambio, podemos hacer un nuevo tipo y poner las validaciones en una
función para crear una instancia del tipo en lugar de repetir las
validaciones en todas partes. De esta forma, es seguro para las funciones
usar el nuevo tipo en sus firmas y usar con confianza los valores que
reciben. El Listado 9-9 muestra una forma de definir un tipo `Guess` que solo
creará una instancia de `Guess` si la función `new` recibe un valor entre 1 y
100.

```rust
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

    pub fn value(&self) -> u32 {
        self.value
    }
}
```

<span class="caption">Listing 9-9: Un tipo `Guess` que solo continuará con
valores entre 1 y 100</span>

Primero, definimos una estructura llamada `Guess` que tiene un campo llamado
`value` que contiene un `u32`. Aquí es donde se almacenará el número.

Luego implementamos una función asociada llamada `new` en `Guess` que crea
instancias de valores `Guess`. La función `new` se define para tener un
parámetro llamado `value` de tipo `u32` y para devolver un `Guess`. El código
en el cuerpo de la función `new` prueba `value` para asegurarse de que esté
entre 1 y 100. Si `value` no pasa esta prueba, hacemos una llamada `panic!`,
Que alertará al programador quién está escribiendo el código de llamada que
tienen un error que necesitan corregir, porque la creación de un `Guess` con
un `value` fuera de este rango violaría el contrato en el que `Guess::new`
se basa. Las condiciones en las cuales `Guess::new` podría entrar en pánico
deberían discutirse en su documentación pública de API; cubriremos las
convenciones de documentación que indican la posibilidad de un `panic!` en la documentación de la API que usted crea en el Capítulo 14. Si `value` pasa la prueba, creamos un nuevo `Guess` con su campo `value` establecido en el parámetro `value` y devuelve el `Guess`.

A continuación, implementamos un método llamado `value` que toma prestado
`self`, no tiene ningún otro parámetro y devuelve un `u32`. Este tipo de
método a veces se llama *getter*, porque su propósito es obtener algunos
datos de sus campos y devolverlos. Este método público es necesario porque el
campo `value` de la estructura `Guess` es privado. Es importante que el campo
`value` sea privado para que el código que utiliza la estructura `Guess` no
permita establecer `value` directamente: el código fuera del módulo *debe*
usar la función `Guess::new` para crear una instancia de `Guess`, lo que
garantiza que no haya forma de que `Guess` tenga un `value` que no haya sido
verificado por las condiciones en la función `Guess::new`.

Una función que tiene un parámetro o devuelve solo números entre 1 y 100
podría entonces declarar en su firma que toma o devuelve un `Guess` en lugar
de un `u32` y no necesitaría hacer ninguna comprobación adicional en su
cuerpo.

## Resumen

Las funciones de manejo de errores de Rust están diseñadas para ayudarte a
escribir códigos más sólidos. La macro `panic!` indica que su programa está
en un estado que no puede manejar y le permite decirle al proceso que se
detenga en lugar de tratar de proceder con valores inválidos o incorrectos.
La enumeración `Result` usa el sistema de tipos de Rust para indicar que las
operaciones pueden fallar de una manera que su código podría recuperarse.
Puede usar `Result` para indicarle al código que llama a su código que
necesita manejar el posible éxito o falla también. El uso de `panic!` Y
`Result` en las situaciones apropiadas hará que su código sea más confiable
frente a problemas inevitables.

Ahora que ha visto formas útiles de que la biblioteca estándar use genéricos
con las enumeraciones `Option` y `Result`, hablaremos sobre cómo funcionan
los genéricos y cómo puede usarlos en su código.