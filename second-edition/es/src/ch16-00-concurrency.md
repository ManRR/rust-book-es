# Fearless Concurrency

Manejar la programación concurrente de manera segura y eficiente es otro de
los principales objetivos de Rust. *La programación simultánea* (*Concurrent
programming*), donde las diferentes partes de un programa se ejecutan de
manera independiente, y la *programación paralela* (*parallel programming*),donde diferentes partes de un programa se ejecutan al mismo tiempo, son cada
vez más importantes a medida que más computadoras aprovechan sus múltiples
procesadores. Históricamente, la programación en estos contextos ha sido
difícil y propensa a errores: Rust espera cambiar eso.

Inicialmente, el equipo de Rust pensó que garantizar la seguridad de la
memoria y prevenir los problemas de concurrencia eran dos desafíos separados
que debían resolverse con diferentes métodos. Con el tiempo, el equipo
descubrió que los sistemas de propiedad y tipo son un poderoso conjunto de
herramientas para ayudar a administrar los problemas de simultaneidad de
seguridad *y* de la memoria. Al aprovechar la propiedad y la verificación de
tipos, muchos errores de concurrencia son errores de tiempo de compilación en
Rust en lugar de errores de tiempo de ejecución. Por lo tanto, en lugar de
hacerle perder mucho tiempo tratando de reproducir las circunstancias exactas
en las que se produce un error de simultaneidad de tiempo de ejecución, el
código incorrecto se negará a compilar y presentará un error explicando el
problema. Como resultado, puede corregir su código mientras está trabajando
en él, en lugar de hacerlo potencialmente después de que se haya enviado a
producción. Hemos apodado este aspecto de Rust *fearless* *concurrency*. La
concurrencia imprudente le permite escribir código libre de errores sutiles y
es fácil de refactorizar sin introducir nuevos errores.

> Nota: por simplicidad, nos referiremos a muchos de los problemas como
> *concurrente* en lugar de ser más precisos diciendo
> *concurrente y/o paralelo*. Si este libro tratase sobre concurrencia y/o
> paralelismo, seríamos más específicos. Para este capítulo, sustituya
> mentalmente *concurrente y/o paralelo* siempre que usemos *concurrente*.

Muchos lenguajes son dogmáticos sobre las soluciones que ofrecen para manejar
problemas concurrentes. Por ejemplo, Erlang tiene una funcionalidad elegante
para la simultaneidad de paso de mensajes pero solo tiene formas oscuras de
compartir estado entre hilos. Respaldar solo un subconjunto de posibles
soluciones es una estrategia razonable para lenguajes de nivel superior,porque un lenguaje de nivel superior promete beneficios al ceder algún
control para obtener abstracciones. Sin embargo, se espera que los lenguajes
de nivel inferior proporcionen la solución con el mejor rendimiento en
cualquier situación dada y tengan menos abstracciones sobre el hardware. Por
lo tanto, Rust ofrece una variedad de herramientas para modelar problemas de
la manera que sea apropiada para su situación y sus requisitos.

Estos son los temas que trataremos en este capítulo:

* Cómo crear *hilos* (*threads*) para ejecutar múltiples piezas de código al mismo tiempo
* *Concurrencia de mensaje*, donde los canales envían mensajes entre *hilos*   (*threads*)
* *Concurrencia de estado compartido*, donde varios subprocesos tienen acceso
 a algún dato
* Los *trait* `Sync` y `Send`, que extienden las garantías de concurrencia de
 Rust a los tipos definidos por el usuario, así como a los tipos proporcionados por la biblioteca estándar