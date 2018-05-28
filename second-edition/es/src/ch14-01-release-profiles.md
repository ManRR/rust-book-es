## Personalizar compilaciones con *Release Profiles*

En Rust, *release profiles* son perfiles predefinidos y personalizables con
diferentes configuraciones que permiten a un programador tener más control
sobre varias opciones para compilar el código. Cada perfil está configurado
independientemente de los demás.

Cargo tiene dos perfiles principales: el perfil `dev` que Cargo utiliza cuando ejecuta `cargo build` y el perfil`release` que Cargo utiliza cuando
ejecuta `cargo build --release`. El perfil `dev` se define con buenos valores
por defecto para el desarrollo, y el perfil `release` tiene buenos valores
por defecto para las compilaciones de versiones.

Estos nombres de perfil pueden ser conocidos a partir de la salida de sus
compilaciones:

```text
$ cargo build
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
$ cargo build --release
    Finished release [optimized] target(s) in 0.0 secs
```

El `dev` y `release` que se muestran en este resultado de compilación indican
que el compilador usa diferentes perfiles.

Cargo tiene configuraciones predeterminadas para cada uno de los perfiles que
se aplican cuando no hay secciones `[profile.*]` en el archivo *Cargo.toml*
del proyecto. Al agregar secciones `[profile.*]` para cualquier perfil que
desee personalizar, puede anular cualquier subconjunto de la configuración
predeterminada. Por ejemplo, aquí están los valores predeterminados para la
configuración `opt-level` para los perfiles `dev` y `release`:

<span class="filename">Filename: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

La configuración `opt-level` controla el número de optimizaciones que Rust
aplicará a su código, con un rango de 0 a 3. Aplicar más optimizaciones
extiende el tiempo de compilación, por lo que si está en desarrollo y
compilando su código con frecuencia, desea una compilación más rápida incluso
si el código resultante se ejecuta más despacio. Esa es la razón por la cual
el `opt-level` predeterminado para `dev` es `0`. Cuando esté listo para
lanzar su código, es mejor dedicar más tiempo a la compilación. Solo
compilará en modo de lanzamiento una vez, pero ejecutará el programa
compilado muchas veces, por lo que el modo de lanzamiento intercambia tiempo
de compilación más largo para el código que se ejecuta más rápido. Es por eso
que el `opt-level` predeterminado para el perfil `release` es `3`.

Puede anular cualquier configuración predeterminada agregando un valor
diferente en *Cargo.toml*. Por ejemplo, si queremos usar el nivel de
optimización 1 en el perfil de desarrollo, podemos agregar estas dos líneas
al archivo *Cargo.toml* de nuestro proyecto:

<span class="filename">Filename: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

Este código anula la configuración predeterminada de `0`. Ahora cuando
ejecutamos `cargo build`, Cargo utilizará los valores predeterminados para el
perfil `dev` más nuestra personalización para `opt-level`. Debido a que
establecemos `opt-level` en `1`, Cargo aplicará más optimizaciones que las
predeterminadas, pero no tantas como en una versión de lanzamiento.

Para ver la lista completa de opciones de configuración y valores
predeterminados para cada perfil, consulte
[Cargo’s documentation](https://doc.rust-lang.org/cargo/).
