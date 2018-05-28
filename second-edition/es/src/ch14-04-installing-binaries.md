## Instalación de binarios de Crates.io con `cargo install`

El comando `cargo install` le permite instalar y usar *binary crates*
localmente. Esto no pretende reemplazar los paquetes del sistema; está
destinado a ser una forma conveniente para que los desarrolladores de Rust
instalen herramientas que otros compartieron en
[crates.io](https://crates.io) <!-- ignore -->. Tenga en cuenta que solo
puede instalar paquetes que tienen *binary target*. Un *binary target*
es el programa ejecutable que se crea si el *crate* tiene un archivo
*src/main.rs* u otro archivo especificado como binario, a diferencia de un
*library target* que no se puede ejecutar por sí mismo, pero es adecuado para
incluir dentro de otros programas. Por lo general, los *crates* tienen
información en el archivo *README* sobre si un *crate* es una *library*, si
tiene un *binary target* o ambos.

Todos los binarios instalados con `cargo install` se almacenan en la carpeta
*bin* de la raíz de la instalación. Si instaló Rust usando `rustup` y no
tiene ninguna configuración personalizada, este directorio será
*$HOME/.cargo/bin*. Asegúrese de que el directorio esté en su `$PATH` para
poder ejecutar programas que haya instalado con `cargo install`.

Por ejemplo, en el Capítulo 12 mencionamos que hay una implementación de Rust
de la herramienta `grep` llamada `ripgrep` para buscar archivos. Si queremos
instalar `ripgrep`, podemos ejecutar lo siguiente:

```text
$ cargo install ripgrep
Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading ripgrep v0.3.2
 --snip--
   Compiling ripgrep v0.3.2
    Finished release [optimized + debuginfo] target(s) in 97.91 secs
  Installing ~/.cargo/bin/rg
```

La última línea del resultado muestra la ubicación y el nombre del binario
instalado, que en el caso de `ripgrep` es `rg`. Siempre que el directorio de
instalación esté en su `$PATH`, como se mencionó anteriormente, puede
ejecutar `rg --help` y ¡comenzar a utilizar una herramienta más rápida y
segura para buscar archivos!
