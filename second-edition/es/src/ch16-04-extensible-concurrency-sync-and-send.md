## Concurrencia extensible con los *trait* `Sync` y `Send`

Curiosamente, el lenguaje Rust tiene *muy* pocas características de
*concurrencia* (*concurrency*). Casi todas las características de
concurrencia de las que hemos hablado hasta ahora en este capítulo han sido
parte de la biblioteca estándar, no del lenguaje. Sus opciones para controlar
la concurrencia no están limitadas al lenguaje o la biblioteca estándar;
puede escribir sus propias características de simultaneidad o usar las escritas por otros.

Sin embargo, dos conceptos de concurrencia están incrustados en el lenguaje: los *traits* `std::marker`, `Sync` y `Send`.

### Permitir la transferencia de propiedad entre hilos con `Send`

El *trait* del marcador `Send` indica que la propiedad del tipo que
implementa `Send` se puede transferir entre hilos. Casi todos los tipos de
Rust son `Send`, pero hay algunas excepciones, como `Rc<T> `: no puede ser
`Send` porque si clonó un valor `Rc<T>` e intentó transferir la propiedad del
clon a otro subproceso, ambos subprocesos pueden actualizar el recuento de
referencia al mismo tiempo. Por este motivo, `Rc<T>` se implementa para usar
en situaciones de subproceso único en las que no desea pagar la penalización
de rendimiento de subprocesos.

Por lo tanto, el sistema de tipos y los límites de rasgos de Rust aseguran
que nunca se pueda enviar accidentalmente un valor `Rc<T>` a través de los
hilos de forma insegura. Cuando tratamos de hacer esto en el listado 16-14,
obtuvimos el error `the trait Send is not implemented for Rc<Mutex<i32>>`.
Cuando cambiamos a `Arc<T>`, que es `Send`, el código compilado.

Cualquier tipo compuesto enteramente de tipos `Send` también se marca
automáticamente como `Send`. Casi todos los tipos primitivos son `Send`,
además de los *raw pointers*, que veremos en el Capítulo 19.

### Permitir acceso desde múltiples hilos con `Sync`

El *trait* del marcador `Sync` indica que es seguro para el tipo que
implementa `Sync` que se haga referencia desde múltiples hilos. En otras
palabras, cualquier tipo `T` es `Sync` si `&T` (una referencia a `T`) es
`Send`, lo que significa que la referencia se puede enviar de forma segura a
otro hilo. Similar a `Send`, los tipos primitivos son `Sync`, y los tipos
compuestos completamente por tipos que son `Sync` también son `Sync`.

El puntero inteligente `Rc<T>` tampoco es `Sync` por las mismas razones por
las que no es `Send`. El tipo `RefCell<T>` (del que hablamos en el Capítulo
15) y la familia de tipos relacionados `Cell<T>` no son `Sync`. La implementación de la comprobación de préstamos que `RefCell<T>` hace en tiempo de ejecución no es seguro *thread-safe*. El puntero inteligente
`Mutex<T>` es `Sync` y se puede usar para compartir el acceso con múltiples
hilos como se vio en la sección “Compartir un `Mutex<T>` entre varios hilos”.

### Implementar `Send` y `Sync` Manualmente es inseguro

Como los tipos que están compuestos por los *traits* `Enviar` y `Sincronizar`
también son automáticamente `Enviar` y `Sincronizar`, no tenemos que
implementar esos *traits* manualmente. Como *traits* de marcador, ni siquiera
tienen ningún método para implementar. Solo son útiles para aplicar
invariantes relacionados con la concurrencia.

La implementación manual de estos *traits* implica implementar código de Rust
inseguro. Hablaremos sobre el uso de un código de Rust inseguro en el
Capítulo 19; por ahora, la información importante es que construir nuevos
tipos simultáneos que no estén compuestos por las partes `Enviar` y
`Sincronizar` requiere una cuidadosa reflexión para mantener las garantías de
seguridad. [The Rustonomicon] tiene más información sobre estas garantías y
cómo mantenerlas.

[The Rustonomicon]: https://doc.rust-lang.org/stable/nomicon/

## Resumen

Esta no es la última vez que verá concurrencia en este libro: el proyecto del
Capítulo 20 utilizará los conceptos de este capítulo en una situación más
realista que los ejemplos más pequeños que se tratan aquí.

Como se mencionó anteriormente, debido a que muy poco de cómo Rust maneja la
concurrencia es parte del lenguaje, muchas soluciones de concurrencia se
implementan como cajas. Éstos evolucionan más rápido que la biblioteca
estándar, así que asegúrese de buscar en línea las cajas actuales y de última
generación para usar en situaciones multiproceso.

La biblioteca estándar de Rust proporciona canales para el paso de mensajes y
tipos de punteros inteligentes, como `Mutex<T>` y `Arc<T>`, que son seguros
de usar en contextos concurrentes. El sistema de tipos y el comprobador de
préstamos garantizan que el código que utiliza estas soluciones no termine
con carreras de datos o referencias no válidas. Una vez que obtenga su código
para compilar, puede estar seguro de que se ejecutará felizmente en varios
subprocesos sin los tipos de errores difíciles de localizar comunes en otros
lenguajes. La programación concurrente ya no es un concepto al que temer:
¡avance y convierta sus programas en concurrentes, sin miedo!.

A continuación, hablaremos sobre formas idiomáticas de modelar problemas y
estructurar soluciones a medida que sus programas de Rust crecen. Además,
discutiremos cómo los modismos de Rust se relacionan con aquellos con los que
podrías estar familiarizado con la programación orientada a objetos.
