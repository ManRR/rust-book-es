# *Smart Pointers* (*Punteros inteligentes*)

Un *puntero* es un concepto general para una variable que contiene una
dirección en la memoria. Esta dirección se refiere a, o “apunta a,” algunos
otros datos. El tipo de puntero más común en Rust es una referencia, que
aprendió en el Capítulo 4. Las referencias se indican con el símbolo `&` y
toman prestado el valor al que apuntan. No tienen ninguna capacidad especial
que no sea referirse a los datos. Además, no tienen sobrecarga y son el tipo
de puntero que usamos con más frecuencia.

Los *punteros inteligentes* (*Smart Pointers*), por otro lado, son estructuras de
datos que no
solo actúan como un puntero sino que también tienen metadatos y capacidades
adicionales. El concepto de *punteros inteligentes* no es exclusivo de Rust:
los punteros inteligentes se originaron en C ++ y existen en otros lenguajes
también. En Rust, los diferentes *punteros inteligentes* definidos en la
biblioteca estándar proporcionan una funcionalidad más allá de la
proporcionada por las referencias. Un ejemplo que exploraremos en este
capítulo es el *conteo de referencias* un tipo de puntero inteligente. Este
puntero le permite tener múltiples propietarios de datos al hacer un
seguimiento del número de propietarios y, cuando no quedan propietarios,
limpiar los datos.

En Rust, que usa el concepto de propiedad y endeudamiento, una diferencia
adicional entre referencias y punteros inteligentes es que las referencias
son punteros que solo toman datos prestados; en cambio, en muchos casos, los
punteros inteligentes *poseen* los datos que señalan.

Ya hemos encontrado algunos punteros inteligentes en este libro, como
`String` y `Vec <T>` en el Capítulo 8, aunque no los llamamos punteros
inteligentes en ese momento. Ambos tipos cuentan como punteros inteligentes
porque poseen algo de memoria y te permiten manipularla. También tienen
metadatos (como su capacidad) y capacidades o garantías adicionales (como con
`String` que garantiza que sus datos siempre serán válidos para UTF-8).

Los punteros inteligentes generalmente se implementan usando estructuras. La
característica que distingue a un puntero inteligente de una estructura
ordinaria es que los punteros inteligentes implementan los *trait* `Deref`
y `Drop`. El trait `Deref` permite que una instancia de la estructura del
puntero inteligente se comporte como una referencia para que pueda escribir
código que funcione con referencias o punteros inteligentes. El *trait*
`Drop` le permite personalizar el código que se ejecuta cuando una instancia
del puntero inteligente sale del alcance. En este capítulo, discutiremos
ambos *trait* y demostraremos por qué son importantes para los indicadores
inteligentes.

Dado que el patrón de puntero inteligente es un patrón de diseño general que
se utiliza con frecuencia en Rust, este capítulo no cubrirá todos los
punteros inteligentes existentes. Muchas bibliotecas tienen sus propios
indicadores inteligentes, e incluso puede escribir el suyo. Cubriremos los
punteros inteligentes más comunes en la biblioteca estándar:

* `Box <T>` para asignar valores en el montículo
* `Rc <T>`, un tipo de conteo de referencia que permite la propiedad múltiple
* `Ref <T>` y `RefMut <T>`, a los que se accede a través de `RefCell <T>`, un
 tipo que impone las reglas de préstamo en tiempo de ejecución en lugar de
 tiempo de compilación

Además, cubriremos el patrón de *mutabilidad interior*
(*interior mutability*) donde un tipo
inmutable expone una API para mutar un valor interior. También discutiremos
*ciclos de referencia* (*reference cycles*): cómo pueden perder memoria y cómo prevenirlos.

¡Vamos a sumergirnos!
