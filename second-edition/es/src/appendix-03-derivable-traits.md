## Apéndice C: *Trait* derivables

En varios lugares del libro, hemos discutido el atributo `derive`, que
puedes aplicar a una definición *struct* o *enum*. El atributo `derive` genera
código que implementará un *trait* con su propia implementación en el tipo
que ha anotado con la sintaxis `derive`.

En este apéndice, proporcionamos una referencia de todos los *traits* en la
biblioteca estándar que puede usar con `derivar`. Cada sección cubre:

* Qué operadores y métodos que derivan este *trait* permitirán
* Lo que hace la implementación del *trait* proporcionado por `derive`
* Qué significa implementar el *trait* sobre el tipo
* Las condiciones en las que se le permite o no se le permite implementar el
 *trait*
* Ejemplos de operaciones que requieren el *trait*

Si desea un comportamiento diferente al proporcionado por el atributo
`derive`, consulte la documentación de la biblioteca estándar para cada
*trait* para obtener detalles sobre cómo implementarlos manualmente.

El resto de los *traits* definidos en la biblioteca estándar no se pueden
implementar en tus tipos usando `derivar`. Estos *traits* no tienen un
comportamiento predeterminado sensible, por lo tanto, depende de usted
implementarlos de la manera que tenga sentido para lo que está tratando de
lograr.

Un ejemplo de un *trait* que no se puede derivar es `Display`, que maneja
formateo para usuarios finales. Siempre debe considerar la forma adecuada de
mostrar un tipo a un usuario final. ¿Qué partes del tipo debería permitir que
un usuario final vea? ¿Qué partes encontrarían relevantes?. ¿Qué formato de
los datos sería más relevante para ellos? El compilador Rust no tiene esta
información, por lo que no puede proporcionar el comportamiento
predeterminado adecuado para usted.

La lista de *traits* derivables proporcionada en este apéndice no es
exhaustiva:
las bibliotecas pueden implementar `derivar` para sus propios *traits*,
haciendo que la lista de *traits* que puede utilizar `derivar` con
verdaderamente abiertos. Implementando `derivar` implica el uso de una macro
de procedimiento, que se trata en el Apéndice D.

### `Debug` para la salida del programador

El *trait* `Debug` habilita el formateo de depuración en los *strings* de
formato, que usted indica al agregar `:?` dentro de los placeholders con
`{}`.

El *trait* `Debug` le permite imprimir instancias de un tipo con fines de
depuración, para que usted y otros programadores que usen su tipo puedan
inspeccionar una instancia en un punto particular de la ejecución de un
programa.

El *trait* `Debug` es requerido, por ejemplo, en el uso de la macro
`assert_eq!`. Esta macro imprime los valores de las instancias dadas como
argumentos si falla la aserción de igualdad para que los programadores puedan
ver por qué las dos instancias no fueron iguales.

### `PartialEq` y `Eq` para comparaciones de igualdad

El *trait* `PartialEq` le permite comparar instancias de un tipo para
verificar la igualdad y permite el uso de los operadores `==` y `! = `.

Derivar `PartialEq` implementa el método `eq`. Cuando `PartialEq` se deriva
en structs, dos instancias son iguales solo si los campos *all* son iguales,
y las instancias no son iguales si los campos no son iguales. Cuando se
deriva en *enums*, cada variante es igual a sí misma y no igual a las otras
variantes.

El *trait* `PartialEq` es requerido, por ejemplo, con el uso de la macro
`assert_eq!`, que necesita poder comparar dos instancias de un tipo para la
igualdad.

El *trait* `Eq` no tiene métodos. Su propósito es señalar que para cada valor
del tipo anotado, el valor es igual a sí mismo. El *trait* `Eq` solo se puede
aplicar a los tipos que también implementan `PartialEq`, aunque no todos los
tipos que implementan `PartialEq` pueden implementar `Eq`. Un ejemplo de esto
son los tipos de números de coma flotante: la implementación de números de
coma flotante establece que dos instancias del valor *not-a-number* (`NaN`) no
son iguales entre sí.

Un ejemplo de cuando se requiere `Eq` es para las teclas en `HashMap<K, V>`,
por lo que `HashMap<K, V>` puede indicar si dos teclas son iguales.

### `PartialOrd` y `Ord` para hacer comparaciones de pedidos

El *trait* `PartialOrd` le permite comparar instancias de un tipo para
clasificar propósitos. Un tipo que implementa `PartialOrd` se puede usar con
`<`, `>`, `<=`, y `>=` operadores. Solo puede aplicar el *trait* `PartialOrd`
a los tipos que también implementan `PartialEq`.

Derivar `PartialOrd` implementa el método `partial_cmp`, que devuelve un
`Option<Ordering>` que será `None` cuando los valores dados no producen un
orden. Un ejemplo de un valor que no produce un orden, aunque la mayoría de
los valores de ese tipo se pueden comparar, es el valor de punto flotante
*not-a-number* (`NaN`). Llamar `partial_cmp` con cualquier número de punto
flotante y el valor de punto flotante `NaN` devolverá `None`.

Cuando se deriva en estructuras, `PartialOrd` compara dos instancias al
comparar el valor en cada campo en el orden en que los campos aparecen en la
estructura definición. Cuando se deriva en *enums*, variantes de la
enumeración declarada anteriormente en la definición de *enum* se considera
menor que las variantes enumeradas más adelante.

El *trait* `PartialOrd` es obligatorio, por ejemplo, para el método
`gen_range` desde el *crate* `rand` que genera un valor aleatorio en el rango
especificado por un valor bajo y un valor alto .

El *trait* `Ord` le permite saber que para cualquier dos valores del tipo
anotado, existirá un pedido válido. El *trait* `Ord` implementa el método
`cmp`, que devuelve `Ordering` en lugar de `Option<Ordering>` porque siempre
será posible realizar un pedido válido. Solo puede aplicar el *trait* `Ord` a
los tipos que también implementan `PartialOrd` y `Eq` (y `Eq` requiere
`PartialEq`). Cuando se deriva en estructuras y *enums*, `cmp` se comporta de
la misma manera que la implementación derivada para `partial_cmp` con
`PartialOrd`.

Un ejemplo de cuando se requiere `Ord` es cuando se almacenan valores en un
`BTreeSet<T>`, una estructura de datos que almacena datos según el orden de
clasificación de los valores.

### `Clone` y `Copy` para duplicar valores

El *trait* `Clonar` le permite crear explícitamente una copia profunda de un
valor, y el proceso de duplicación puede implicar ejecutar código arbitrario
y copiar montículo de datos. Consulte la sección “Ways Variables and Data
Interact: Clone” en el Capítulo 4 para más información sobre `Clone`.

Derivar `Clone` implementa el método `clon`, que cuando se implementa para el
tipo completo, llama `clon` a cada una de las partes del tipo. Esto significa
que todos los campos o valores en el tipo también deben implementar `Clone`
para derivar `Clone`.

Un ejemplo de cuando se requiere `Clone` es cuando se llama al método
`to_vec` en un *slice*. El sector no posee las instancias de tipo que
contiene, pero el vector devuelto desde `to_vec` necesitará poseer sus
instancias, por lo que las llamadas `to_vec` `clon` en cada elemento. Por lo
tanto, el tipo almacenado en la división debe implementar `Clone`.

El *trait* `Copy` le permite duplicar un valor copiando solo los bits
almacenados en la pila; no es necesario ningún código arbitrario. Consulte la
sección “Stack-Only Data: Copy” en el Capítulo 4 para obtener más
información sobre `Copy`.

El *trait* `Copy` no define ningún método para evitar que los programadores
sobrecarguen esos métodos y violando la suposición de que no se está
ejecutando ningún código arbitrario. De esta forma, todos los programadores
pueden asumir que copiar un valor será muy rápido.

Puede derivar `Copy` en cualquier tipo cuyas partes implementen `Copy`. Solo
puede aplicar el *trait* `Copy` a los tipos que también implementan `Clon`,
porque un tipo que implementa `Copy` tiene una implementación trivial de
`Clon` que realiza la misma tarea que `Copy`.

El *trait* `Copy` rara vez se requiere; los tipos que implementan `Copy`
tienen optimizaciones disponibles, lo que significa que no tiene que llamar
`clone`, lo que hace que el código sea más conciso.

Todo lo que se puede hacer con `Copy` también se puede lograr con `Clone`,
pero el código puede ser más lento o tener que usar `clone` en algunos
lugares.

### `Hash` para asignar un valor a un valor de tamaño fijo

El *trait* `Hash` le permite tomar una instancia de un tipo de tamaño
arbitrario y asignar esa instancia a un valor de tamaño fijo usando una
función hash. Derivar `Hash` implementa el método `hash`. La implementación
derivada del método `hash` combina el resultado de invocar `hash` en cada una
de las partes del tipo, lo que significa que todos los campos o valores
también deben implementar `Hash` para derivar `Hash`.

Un ejemplo de cuándo se requiere `Hash` es almacenar las claves en
`HashMap<K, V>`para almacenar datos de manera eficiente.

### `Default` para valores predeterminados

El *trait* `Default` le permite crear un valor predeterminado para un tipo.
Derivar `Default` implementa la función `default`. La implementación derivada
de la función `default` llama a la función `default` en cada parte del tipo,
lo que significa que todos los campos o valores del tipo también deben
implementar `Default` para derivar `Default`.

La función `Default::default` se usa comúnmente en combinación con la
sintaxis de actualización struct discutida en la sección “Crear instancias de
otras instancias con *Struct Update Syntax*” en el Capítulo 5. Puede
personalizar algunos campos de una estructura y luego configurar y use un
valor predeterminado para el resto de los campos usando
`..Default::default()`.

El *trait* `Default` es obligatorio cuando utiliza el método
`unwrap_or_default` en instancias `Option<T>`, por ejemplo. Si la
`Opción <T>` es `None`, el método `unwrap_or_default` devolverá el resultado
de `Default::default` para el tipo `T` almacenado en `Option<T>`.
