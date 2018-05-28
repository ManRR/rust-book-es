## Instalación

El primer paso es instalar Rust. Descargaremos Rust a través de `rustup`, una
herramienta de línea de comando para administrar las versiones de Rust y las
herramientas asociadas. Necesitará una conexión a Internet para la descarga.

> Nota: si prefiere no usar `rustup` por algún motivo, consulte
> [la página de instalación de Rust](https://www.rust-lang.org/install.html) > para ver otras opciones.

Los siguientes pasos instalan la última versión estable del compilador Rust.
Todos los ejemplos y resultados de este libro usan Rust 1.21.0 estable. Las
garantías de estabilidad de Rust aseguran que todos los ejemplos en el libro
que se compilan continúen compilando con versiones más nuevas de Rust. La
salida puede variar ligeramente entre las versiones, porque Rust a menudo
mejora los mensajes de error y las advertencias. En otras palabras, cualquier
versión más nueva y estable de Rust que instale utilizando estos pasos
debería funcionar como se espera con el contenido de este libro.

> ### Notación de línea de comando
>
> En este capítulo y en todo el libro, mostraremos algunos comandos
> utilizados en la terminal. Las líneas que debe ingresar en un terminal
> comienzan todas con `$`. No necesita escribir el caracter `$`; indica el
> inicio de cada comando. Las líneas que no comienzan con `$` típicamente
> muestran el resultado del comando anterior. Además, los ejemplos específicos
> de PowerShell usarán `>` en lugar de `$`.

### Instalando `rustup` en Linux o macOS

Si está utilizando Linux o macOS, abra una terminal e ingrese el siguiente
comando:

```text
$ curl https://sh.rustup.rs -sSf | sh
```

El comando descarga un script e inicia la instalación de la herramienta
`rustup`, que instala la última versión estable de Rust. Es posible que se le
pida su contraseña. Si la instalación es exitosa, aparecerá la siguiente
línea:

```text
Rust is installed now. Great!
```

Si lo prefiere, puede descargar el script e inspeccionarlo antes de
ejecutarlo.

La secuencia de comandos de instalación agrega automáticamente Rust al
PATH del sistema después de su próximo inicio de sesión. Si desea comenzar a
usar Rust de inmediato en lugar de reiniciar su terminal, ejecute el
siguiente comando en su caparazón para agregar Rust al PATH del sistema
manualmente:

```text
$ source $HOME/.cargo/env
```

Alternativamente, puede agregar la siguiente línea a su *~/.bash_profile*:

```text
$ export PATH="$HOME/.cargo/bin:$PATH"
```

Además, necesitarás un *enlazador* (*linker*) de algún tipo. Es probable que
ya esté instalado, pero cuando intenta compilar un programa de Rust y obtiene
errores que indican que un enlazador no se pudo ejecutar, eso significa que
un enlazador no está instalado en su sistema y tendrá que instalar uno
manualmente. Los compiladores C normalmente vienen con el enlazador correcto.
Consulte la documentación de su plataforma para saber cómo instalar un
compilador de C. Además, algunos paquetes comunes de Rust dependen del código
C y necesitarán un compilador de C. Por lo tanto, podría valer la pena
instalar uno ahora.

### Instalando `rustup` en Windows

En Windows, vaya a [https://www.rust-lang.org/install.html][install] y siga
las instrucciones para instalar Rust. En algún momento de la instalación,
recibirá un mensaje explicando que también necesitará las herramientas de
compilación de C ++ para Visual Studio 2013 o posterior. La forma más fácil
de adquirir las herramientas de compilación es instalar
[Build Tools for Visual Studio 2017][visualstudio]. Las herramientas se
encuentran en la sección Otras herramientas y marcos.

[instalar]: https://www.rust-lang.org/install.html
[visualstudio]: https://www.visualstudio.com/downloads/

El resto de este libro usa comandos que funcionan tanto en *cmd.exe* como en
PowerShell. Si hay diferencias específicas, explicaremos cuál usar.

### Actualización y desinstalación

Después de haber instalado Rust mediante `rustup`, es fácil actualizar a la
última versión. Desde su shell, ejecute el siguiente script de actualización:

```text
$ rustup update
```

Para desinstalar Rust y `rustup`, ejecute el siguiente script de
desinstalación desde su shell:

```text
$ rustup self uninstall
```

### Solución de problemas

Para verificar si tiene instalado Rust correctamente, abra un shell e ingrese
esta línea:

```text
$ rustc --version
```

Debería ver el número de versión, el hash de confirmación y la fecha de
confirmación para la última versión estable que se ha publicado en el
siguiente formato:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Si ve esta información, ¡ha instalado Rust con éxito! Si no ve esta
información y está en Windows, verifique que Rust esté en su variable de
sistema `%PATH%`. Si eso es todo correcto y Rust todavía no funciona, hay
varios lugares donde puede obtener ayuda. El más fácil es
[the #rust IRC channel on irc.mozilla.org][irc]<!-- ignore -->, al que se
puede acceder a través de [Mibbit][mibbit]. En esa dirección puedes chatear
con otros Rustaceos (un sobrenombre ridículo que nos llamamos a nosotros
mismos) que pueden ayudarte. Otros recursos geniales incluyen [el foro de usuarios][users] y [Stack Overflow][stackoverflow].

[irc]: irc://irc.mozilla.org/#rust
[mibbit]: http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust
[users]: https://users.rust-lang.org/
[stackoverflow]: http://stackoverflow.com/questions/tagged/rust

### Documentación local

El instalador también incluye una copia de la documentación localmente, para
que pueda leerla sin conexión. Ejecute `rustup doc` para abrir la
documentación local en su navegador.

Cada vez que la biblioteca estándar proporciona un tipo o función y no está
seguro de qué hace ni cómo usarla, ¡utilice la documentación de la interfaz
de programación de aplicaciones (API) para averiguarlo!
