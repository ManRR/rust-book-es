# Características del lenguaje funcional: *Iterators* y *Closures*

El diseño de Rust se ha inspirado en muchos lenguajes y técnicas existentes,
y una influencia significativa es la *programación funcional*. La
programación en un estilo funcional a menudo incluye el uso de funciones como
valores pasándolos en argumentos, regresándolos de otras funciones,
asignándolos a variables para su posterior ejecución, y así sucesivamente.

En este capítulo, no debatiremos el tema de qué programación funcional es o
no será, sino que discutiremos algunas características de Rust que son
similares a las características en muchos lenguajes que a menudo se denominan
funcionales.

Más específicamente, cubriremos:

* *Closures*, una construcción similar a una función que puedes almacenar en una variable
* *Iteradores*, una forma de procesar una serie de elementos
* Cómo utilizar estas dos características para mejorar el proyecto de E/S en
 el Capítulo 12
* El rendimiento de estas dos características (alerta de spoiler: ¡son más
 rápidos de lo que crees!)

Otras características de Rust, como la *coincidencia de patrones*
(*pattern matching*) y las enumeraciones, que hemos cubierto en otros
capítulos, también están influenciadas por el estilo funcional. Dominar los
*Closures* y los *Iterators* (*Iteradores*) es una parte importante de la
redacción del código Rust idiomático y rápido, por lo que les dedicaremos
todo este capítulo.