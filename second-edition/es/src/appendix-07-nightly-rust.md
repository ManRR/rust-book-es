# Apéndice G: cómo se fabrica el Rust y “Nightly Rust”

Este apéndice es sobre cómo se hace Rust y cómo eso te afecta como
desarrollador de Rust. Mencionamos que el resultado en este libro fue
generado por Rust estable 1.21.0, pero cualquier ejemplo que compile debería
continuar compilando en cualquier versión estable de Rust mayor que eso. Esta
sección es para explicar cómo aseguramos que esto sea verdad.

### Estabilidad sin estancamiento

Como lenguaje, a Rust le importa *mucho* la estabilidad de su código.
Queremos que Rust sea una base sólida sobre la que pueda construir, y si las
cosas cambiaran constantemente, sería imposible. Al mismo tiempo, si no
podemos experimentar con nuevas funciones, es posible que no descubramos
fallas importantes hasta después de su lanzamiento, cuando ya no podamos
cambiar las cosas.

Nuestra solución a este problema es lo que llamamos “estabilidad sin
estancamiento”, y nuestro principio rector es este: nunca debería temer
actualizarse a una nueva versión de Rust estable. Cada actualización debería
ser indolora, pero también debería traerle nuevas funciones, menos errores y
tiempos de compilación más rápidos.

### Choo, Choo! Release Channels and Riding the Trains

El desarrollo de Rust opera en un *train schedule*. Es decir, todo el
desarrollo se realiza en la rama `master` del repositorio Rust. Las versiones
siguen un modelo de tren de lanzamiento de software, que ha sido utilizado
por Cisco IOS y otros proyectos de software. Hay tres *release channels* para 
Rust:

* Nightly
* Beta
* Stable

La mayoría de los desarrolladores de Rust utilizan principalmente el canal
estable, pero aquellos que quieran probar nuevas características
experimentales pueden usarlas todas las nightly o beta.

Aquí hay un ejemplo de cómo funciona el proceso de desarrollo y liberación:
supongamos que el equipo de Rust está trabajando en el lanzamiento de Rust
1.5. Ese lanzamiento se realizó en diciembre de 2015, pero nos proporcionará
números de versión realistas. Se agrega una nueva función a Rust: un nuevo
commit aterriza en la rama `master`. Cada noche, se produce una nueva versión
nocturna de Rust. Todos los días es un día de lanzamiento, y nuestra versión
de lanzamiento crea estos lanzamientos automáticamente. A medida que pasa el
tiempo, nuestros lanzamientos se ven así, una vez por noche:

```text
nightly: * - - * - - *
```

¡Cada seis semanas, es hora de preparar una nueva versión! La rama `beta` del
repositorio Rust se bifurca desde la rama `master` utilizada por cada noche.
Ahora, hay dos lanzamientos:

```text
nightly: * - - * - - *
                     |
beta:                *
```

La mayoría de los usuarios de Rust no usan activamente las versiones beta,
sino que las prueban en versión beta en su sistema de CI para ayudar a Rust a
descubrir posibles regresiones. Mientras tanto, todavía hay un lanzamiento
nocturno todas las noches:

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

Digamos que se encuentra una regresión. ¡Qué bueno que hayamos tenido un poco
de tiempo para probar la versión beta antes de que la regresión escapara a
una versión estable!. La corrección se aplica a `master`, por lo que cada
noche se repara, y luego la solución se transfiere a la rama `beta`, y se
produce una nueva versión de beta:

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

¡Seis semanas después de que se creó la primera versión beta, es hora de una
versión estable!. La rama `estable` se produce a partir de la rama `beta`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

¡Hurra! Rust 1.5 está listo! Sin embargo, hemos olvidado una cosa: porque
han pasado las seis semanas, también necesitamos una nueva versión beta de la
*próxima* versión de Rust, 1.6. Así que después de las ramas `stable` de
`beta`, la siguiente versión de `beta` se separa de `nightly` nuevamente:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

Esto se llama el “modelo de tren” porque cada seis semanas, un lanzamiento
“abandona la estación”, pero aún tiene que realizar un viaje a través del
canal beta antes de que llegue como un lanzamiento estable.

Rust lanza cada seis semanas, como un reloj. Si conoce la fecha de una
publicación de Rust, puede saber la fecha de la siguiente: son seis semanas
después. Un buen aspecto de tener lanzamientos programados cada seis semanas
es que el próximo tren llegará pronto. Si una característica se pierde un
lanzamiento en particular, no hay necesidad de preocuparse: ¡otra está
sucediendo en poco tiempo! Esto ayuda a reducir la presión para introducir
funciones posiblemente sin pulir cerca de la fecha límite de lanzamiento.

Gracias a este proceso, siempre puedes consultar la próxima compilación de
Rust y verificar por ti mismo que es fácil actualizar a: si una versión beta
no funciona como se esperaba, puedes informarla al equipo y arreglarla antes
de que se publique. el próximo lanzamiento estable sucede! La rotura en una
versión beta es relativamente rara, pero `rustc` sigue siendo una pieza de
software, y existen errores.

### Características inestables

Hay una captura más con este modelo de lanzamiento: características
inestables. Rust utiliza una técnica llamada “indicadores de características”
para determinar qué características están habilitadas en una versión
determinada. Si una nueva característica está en desarrollo activo, aterriza
en `master`, y por lo tanto, en nightly, pero detrás de *feature flag*. Si
usted, como usuario, desea probar la función de trabajo en progreso, puede
hacerlo, pero debe usar un lanzamiento nocturno de Rust y anotar su código
fuente con la bandera apropiada para habilitarla.

Si está utilizando una versión beta o estable de Rust, no puede usar ninguna
*feature flags*. Esta es la clave que nos permite obtener un uso práctico con
nuevas características antes de declararlas estables para siempre. Aquellos
que deseen optar por la vanguardia pueden hacerlo, y aquellos que quieran una
experiencia sólida como una roca pueden quedarse con la estabilidad y saber
que su código no se romperá. Estabilidad sin estancamiento.

Este libro solo contiene información sobre características estables, ya que
las características en progreso todavía están cambiando, y seguramente serán
diferentes entre cuándo se escribió este libro y cuándo se habilitan en
versiones estables. Puede encontrar documentación solo para *nightly features*
en línea.

### Rustup y el papel de Rust Nightly

Rustup facilita el cambio entre diferentes canales de lanzamiento de Rust, a
nivel global o por proyecto. De forma predeterminada, tendrá instalado Rust
estable. Para instalar todas las noches, por ejemplo:

```text
$ rustup install nightly
```

Puede ver todas las *toolchains* (versiones de Rust y componentes asociados)
que ha instalado con `rustup` también. Aquí hay un ejemplo en una de las
computadoras con Windows de sus autores:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

Como puede ver, la cadena de herramientas estable es la predeterminada. La
mayoría de los usuarios de Rust usan estable la mayor parte del tiempo. Es
posible que desee utilizar estable la mayor parte del tiempo, pero use todas
las noches en un proyecto específico, porque le importa una característica de
vanguardia. Para hacerlo, puede usar `rustup override` en el directorio de
ese proyecto para establecer la cadena de herramientas nocturna como la que
debe usar `rustup` cuando se encuentre en ese directorio:

```text
$ cd ~/projects/needs-nightly
$ rustup override add nightly
```

Ahora, cada vez que llamas `rustc` o `cargo` dentro de
*~/projects/needs-nightly*, `rustup` se asegurará de que está utilizando Rust
nightly, en lugar de su defecto de Rust estable. ¡Esto es útil cuando tienes muchos proyectos de Rust!

### El proceso y los equipos de RFC

Entonces, ¿cómo aprendes acerca de estas nuevas características?. El modelo
de desarrollo de Rust sigue un proceso de *Request For Comments (RFC)
process*. Si desea una mejora en Rust, puede escribir una propuesta, llamada
RFC.

Cualquiera puede escribir RFC para mejorar Rust, y las propuestas son
revisadas y discutidas por el equipo de Rust, que se compone de muchos
subtemas de temas. Hay una lista completa de los equipos
[on Rust’s website](https://www.rust-lang.org/en-US/team.html), que incluye
equipos para cada área del proyecto: diseño del lenguaje, implementación del
compilador, infraestructura, documentación y más. El equipo apropiado lee la
propuesta y los comentarios, escribe algunos comentarios propios y,
finalmente, hay consenso para aceptar o rechazar la función.

Si la característica es aceptada, se abre un problema en el repositorio de
Rust y alguien puede implementarlo. ¡La persona que lo implementa muy bien
puede no ser la persona que propuso la función en primer lugar!. Cuando la
implementación está lista, aterriza en la rama `master` detrás de una puerta
de característica, como discutimos en la sección de “Características
inestables”.

Después de un tiempo, una vez que los desarrolladores de Rust que usan
lanzamientos nocturnos hayan podido probar la nueva función, los miembros del
equipo discutirán la función, cómo se resuelve todas las *nightly* y deciden
si debe convertirse en Rust estable o no. Si la decisión es avanzar, la puerta
característica se elimina y la característica ahora se considera estable.
Monta los trenes en una nueva versión estable de Rust.
