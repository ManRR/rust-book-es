# Proyecto final: creación de un servidor web multiproceso

Ha sido un largo viaje, pero hemos llegado al final del libro. En este capítulo, construiremos un proyecto más juntos para demostrar algunos de los conceptos que cubrimos en los capítulos finales, así como recapitular algunas lecciones anteriores.

Para nuestro proyecto final, haremos un servidor web que diga "hola" y se parezca a la figura 20-1 en un navegador web.

![hello from rust](img/trpl20-01.png)

<span class="caption">Figure 20-1: Nuestro último proyecto compartido</span>

Aquí está el plan para construir el servidor web:

1. Aprende un poco sobre TCP y HTTP.
2. Escuche las conexiones TCP en un socket.
3. Analice una pequeña cantidad de solicitudes HTTP.
4. Crea una respuesta HTTP adecuada.
5. Mejore el rendimiento de nuestro servidor con un grupo de subprocesos
 (*thread pool*).

Pero antes de comenzar, debemos mencionar un detalle: el método que usaremos
no será la mejor manera de construir un servidor web con Rust. En *https:
//crates.io/* se encuentran disponibles varias *crates* listas para
producción que proporcionan implementaciones más completas de servidor web y
conjunto de subprocesos que las que crearemos.

Sin embargo, nuestra intención en este capítulo es ayudarlo a aprender, no a
tomar la ruta fácil. Debido a que Rust es un lenguaje de programación de
sistemas, podemos elegir el nivel de abstracción con el que queremos trabajar
y podemos ir a un nivel más bajo de lo que es posible o práctico en otros
lenguajes. Escribiremos el servidor HTTP básico y el grupo de subprocesos
manualmente para que pueda aprender las ideas generales y técnicas detrás de
los *crates* que puede usar en el futuro.
