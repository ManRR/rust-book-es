# Un proyecto de I/O: creación de un programa de línea de comando

Este capítulo es un resumen de las muchas habilidades que ha aprendido hasta
ahora y una exploración de algunas características más de la biblioteca.
Desarrollaremos una herramienta de línea de comando que interactúa con la
entrada/salida de archivos y línea de comando para practicar algunos de los
conceptos de Rust que ahora tiene bajo su cinturón.

La velocidad, la seguridad, la salida binaria simple y el soporte
multiplataforma de Rust lo convierten en un lenguaje ideal para crear
herramientas de línea de comandos, por lo que para nuestro proyecto,
crearemos nuestra propia versión de la herramienta clásica de línea de
comandos `grep` (**g**lobally search a **r**egular **e**xpression and **p**rint).En el caso de uso más simple, `grep` busca un archivo específico
para un texto especificado. Para hacerlo, `grep` toma como argumentos un
nombre de archivo y un texto. Luego lee el archivo, busca las líneas en ese
archivo que contiene el argumento de texto e imprime esas líneas.

En el camino, mostraremos cómo hacer que nuestra herramienta de línea de
comandos use las características del terminal que usan muchas herramientas de
línea de comando. Leeremos el valor de una variable de entorno para permitir
al usuario configurar el comportamiento de nuestra herramienta. También
imprimiremos en la secuencia de error estándar de la consola (`stderr`) en
lugar de la salida estándar (`stdout`), por lo que, por ejemplo, el usuario
puede redirigir la salida exitosa a un archivo mientras sigue apareciendo
mensajes de error en la pantalla.

El miembro de la comunidad de Rust, Andrew Gallant, ya ha creado una versión
completa y muy rápida de `grep`, llamada `ripgrep`. En comparación, nuestra
versión de `grep` será bastante simple, pero este capítulo le dará algunos de
los conocimientos básicos que necesita para comprender un proyecto del mundo
real como `ripgrep`.

Nuestro proyecto `grep` combinará una cantidad de conceptos que has aprendido
hasta ahora:

* Código de organización (usando lo que aprendiste en los módulos, Capítulo 7)
* Uso de vectores y *string* (colecciones, Capítulo 8)
* Manejo de errores (Capítulo 9)
* Usar *trait* y *lifetimes* cuando corresponda (Capítulo 10)
* Pruebas de escritura (Capítulo 11)

También presentaremos brevemente *closures*, *iterators*, y *trait objects*, que los Capítulos 13 y 17 cubrirán en detalle.
