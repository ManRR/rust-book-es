# Introducción

> Nota: esta edición del libro es igual a
> [The Rust Programming Language][nsprust] disponible en formato de impresión
> y ebook desde [No Starch Press][nsp].

[nsprust]: https://nostarch.com/rust
[nsp]: https://nostarch.com/

Bienvenido a *El lenguaje de programación de Rust*
(*The Rust Programming Language*), un libro introductorio sobre Rust. El
lenguaje de programación Rust le ayuda a escribir software más rápido y
confiable. La ergonomía de alto nivel y el control de bajo nivel a menudo
están reñidos con el diseño del lenguaje de programación; Rust desafía ese
conflicto. Mediante el equilibrio entre una potente capacidad técnica y una
gran experiencia de desarrollador, Rust le ofrece la opción de controlar
detalles de bajo nivel (como el uso de memoria) sin todas las molestias
asociadas tradicionalmente con dicho control.

## Para quién es el Rust

Rust es ideal para muchas personas por una variedad de razones. Veamos
algunos de los grupos más importantes.

### Equipos de desarrolladores

Rust está demostrando ser una herramienta productiva para colaborar entre
grandes equipos de desarrolladores con diferentes niveles de conocimiento de
programación de sistemas. El código de bajo nivel es propenso a una variedad
de errores sutiles, que en la mayoría de los otros lenguajes solo se pueden
atrapar mediante pruebas exhaustivas y una cuidadosa revisión del código por
parte de desarrolladores experimentados. En Rust, el compilador desempeña un
rol de guardián al negarse a compilar código con estos esquivos errores,
incluidos los errores de concurrencia. Al trabajar junto con el compilador,
el equipo puede dedicar su tiempo a enfocarse en la lógica del programa en
lugar de buscar errores.

Rust también trae herramientas de desarrollo contemporáneas al mundo de
programación de sistemas:

* Cargo, el administrador de dependencias incluido y la herramienta de
 compilación, hace que agregar, compilar y gestionar dependencias sea
 sencillo y coherente en todo el ecosistema de Rust.
* Rustfmt garantiza un estilo de codificación consistente entre los
 desarrolladores.
* El Rust Language Server impulsa la integración del entorno de desarrollo
 integrado (IDE) para la finalización del código y mensajes de error en línea.

Al usar estas y otras herramientas en el ecosistema de Rust, los desarrolladores pueden ser productivos al escribir código a nivel de sistema.

### Estudiantes

Rust es para estudiantes y aquellos que están interesados en aprender sobre
conceptos de sistemas. Con Rust, muchas personas han aprendido sobre temas
como el desarrollo de sistemas operativos. La comunidad es muy acogedora y
está feliz de responder las preguntas de los estudiantes. A través de
esfuerzos como este libro, los equipos de Rust quieren que los conceptos de
sistemas sean más accesibles para más personas, especialmente aquellos que
son nuevos en la programación.

### Compañías

Cientos de empresas, grandes y pequeñas, usan Rust en producción para una
variedad de tareas. Esas tareas incluyen herramientas de línea de comandos,
servicios web, herramientas DevOps, dispositivos integrados, análisis y
transcodificación de audio y video, criptomonedas, bioinformática, motores de
búsqueda, aplicaciones de Internet de las cosas, aprendizaje automático e
incluso partes importantes del navegador web Firefox.

### Desarrolladores de código abierto

Rust es para las personas que desean construir el lenguaje de programación
Rust, la comunidad, las herramientas de desarrollo y las bibliotecas. Nos
encantaría que contribuyeras al lenguaje Rust.

### Personas que valoran la velocidad y la estabilidad

Rust es para las personas que anhelan velocidad y estabilidad en un lenguaje.
Por velocidad, nos referimos a la velocidad de los programas que puede crear
con Rust y la velocidad a la que Rust le permite escribirlos. Las
comprobaciones del compilador Rust garantizan la estabilidad mediante la
incorporación de funciones y la refactorización. Esto está en contraste con
el código frágil heredado en lenguaje sin estos controles, que los
desarrolladores a menudo tienen miedo de modificar. Al esforzarse por
abstracciones de costo cero, características de nivel superior que compilan
código de nivel inferior tan rápido como el código escrito manualmente, Rust
se esfuerza por hacer que código seguro sea también código rápido.

El lenguaje Rust espera poder apoyar a muchos otros usuarios también; los
mencionados aquí son simplemente algunos de los principales interesados. En
general, la mayor ambición de Rust es eliminar las concesiones que los
programadores han aceptado durante décadas al proporcionar seguridad
*y* productividad, velocidad *y* ergonomía. Prueba Rust y verifica si sus
opciones te funcionan bien.

## Para quien es este libro

Este libro asume que ha escrito código en otro lenguaje de programación pero
no hace suposiciones sobre cuál. Hemos intentado que el material sea
ampliamente accesible para personas de una amplia variedad de entornos de
programación. No pasamos mucho tiempo hablando acerca de qué *es* la
programación o cómo pensar sobre ella. Si eres completamente nuevo en la
programación, lo mejor sería leer un libro que específicamente ofrezca una
introducción a la programación.

## Como usar este libro

En general, este libro asume que estás leyéndolo en secuencia de principio a
fin. Los capítulos posteriores se basan en conceptos de capítulos anteriores
y anteriores los capítulos podrían no profundizar en detalles sobre un tema;
por lo general revisamos el tema en un capítulo posterior.

Encontrará dos tipos de capítulos en este libro: capítulos conceptuales y
capítulos proyecto. En los capítulos conceptuales, aprenderás sobre un
aspecto de Rust. En los capítulos del proyecto, crearemos pequeños programas
juntos, aplicando lo que has aprendido hasta ahora. Los capítulos 2, 12 y 20
son capítulos de proyectos; el resto son capítulos conceptuales.

El Capítulo 1 explica cómo instalar Rust, cómo escribir un programa
*Hello, world!*, y cómo usar Cargo, el administrador de paquetes de Rust y la
herramienta de compilación. El Capítulo 2 es una introducción práctica al
lenguaje Rust. Aquí cubrimos conceptos a un alto nivel, y capítulos
posteriores proporcionarán detalles adicionales. Si quieres ensuciarte las
manos de inmediato, el Capítulo 2 es el lugar para eso. Al principio, usted
podría incluso omitir el Capítulo 3, que cubre las características de Rust
similares a las de otros lenguajes de programación, y dirígete directamente
al Capítulo 4 para aprender sobre el Sistema de propiedad de Rust. Sin
embargo, si eres un aprendiz particularmente meticuloso
que prefiere aprender cada detalle antes de pasar al siguiente, es posible
que desee saltar el Capítulo 2 e ir directamente al Capítulo 3, volviendo al
Capítulo 2 cuando le gustaría trabajar en un proyecto aplicando los detalles
que has aprendido.

El Capítulo 5 discute las estructuras y los métodos, y el Capítulo 6 cubre
las enumeraciones, las expresiones `match` y la construcción de flujo de
control `if let`. Utilizará structs y enumeraciones para hacer tipos
personalizados en Rust.

En el Capítulo 7, aprenderá sobre el sistema de módulos de Rust y sobre las
reglas de privacidad para organizar su código y su Interfaz de Programación
de Aplicaciones (API) pública. El Capítulo 8 analiza algunas estructuras de
datos de recopilación comunes que proporciona la biblioteca estándar, como
vectores, *string* y mapas hash. El Capítulo 9 explora la filosofía y las
técnicas de manejo de errores de Rust.

El Capítulo 10 explora los genéricos, los *traits* y los *lifetimes*, que le
dan la capacidad de definir el código que se aplica a múltiples tipos. El
Capítulo 11 trata sobre pruebas, que incluso con las garantías de seguridad
de Rust es necesario para garantizar que la lógica de su programa sea
correcta. En el Capítulo 12, construiremos nuestra propia implementación de
un subconjunto de funcionalidades desde la herramienta de línea de comandos
`grep` que busca texto dentro de los archivos. Para esto, usaremos muchos de
los conceptos que discutimos en los capítulos anteriores.

El Capítulo 13 explora *closures* e iteradores: características de Rust que
provienen de lenguajes de programación funcionales. En el Capítulo 14,
examinaremos Cargo con más profundidad y hablaremos sobre las mejores
prácticas para compartir sus bibliotecas con otras personas. El Capítulo 15
analiza los *smart pointers* que proporciona la biblioteca estándar y los
*traits* que permiten su funcionalidad.

En el Capítulo 16, analizaremos diferentes modelos de programación
concurrente y hablaremos sobre cómo Rust te ayuda a programar en múltiples
*hilos* (*threads*) sin miedo. El Capítulo 17 analiza cómo los modismos de
Rust se comparan con los principios de programación orientada a objetos con
los que puede estar familiarizado.

El Capítulo 18 es una referencia sobre patrones y concordancia de patrones,
que son formas poderosas de expresar ideas a través de los programas de Rust.
El Capítulo 19 contiene una mezcla heterogénea de temas de interés avanzados,
incluido Rust *unsafe* y más *lifetimes*, *traits*, tipos, funciones y
*closures*.

En el Capítulo 20, completaremos un proyecto en el que implementaremos un
servidor web *multithreaded* de bajo nivel.

Finalmente, algunos apéndices contienen información útil sobre el lenguaje en
un formato más similar a una referencia. El Apéndice A cubre las palabras
clave de Rust, el Apéndice B cubre los operadores y símbolos de Rust, el
Apéndice C cubre los *traits* derivables proporcionados por la biblioteca
estándar, y el Apéndice D cubre las macros.

No hay una manera incorrecta de leer este libro: si quieres saltarte,
¡adelante! es posible que deba volver a los capítulos anteriores si
experimenta alguna confusión. Pero haz lo que sea que funcione para ti.

Una parte importante del proceso de aprendizaje de Rust es aprender a leer
los mensajes de error que muestra el compilador: estos lo guiarán hacia el
*working code*. Como tal, proporcionaremos muchos ejemplos de código que
no se compila junto con el mensaje de error que el compilador le mostrará en
cada situación. ¡Sepa que si ingresa y ejecuta un ejemplo al azar, es posible
que no compile! asegúrese de leer el texto que lo rodea para ver si el
ejemplo que intenta ejecutar está destinado a error. En la mayoría de las
situaciones, lo guiaremos a la versión correcta de cualquier código que no
compile.

## Código fuente

*Los archivos de origen de los que se genera este libro se pueden encontrar en (versión original ingles)*
[GitHub][book].

[book]: https://github.com/rust-lang/book/tree/master/second-edition/src

Los archivos de origen de los que se genera este libro se pueden encontrar en
[GitHub][book-es].

[book-es]:
