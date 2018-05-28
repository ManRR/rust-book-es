## Extender *Cargo* con comandos personalizados

Cargo está diseñado para que pueda ampliarlo con nuevos subcomandos sin tener
que modificar Cargo. Si un binario en su `$PATH` se llama `cargo-something`,
puede ejecutarlo como si fuera un subcomando Cargo ejecutando
`cargo something`. Los comandos personalizados como este también se enumeran
cuando ejecuta `cargo-list`. ¡Poder usar `cargo install` para instalar
extensiones y luego ejecutarlas al igual que las herramientas de Cargo
incorporadas es un beneficio súper conveniente del diseño de Cargo.

## Resumen

Compartir código con Cargo y [crates.io](https://crates.io) <!-- ignore -->
es parte de lo que hace que el ecosistema de Rust sea útil para muchas tareas
diferentes. La biblioteca estándar de Rust es pequeña y estable, pero los
*crates* son fáciles de compartir, usar y mejorar en una línea de tiempo
diferente a la del lenguaje. No tenga miedo de compartir el código que le sea
útil en [crates.io](https://crates.io) <!-- ignore -->; ¡es probable que
también sea útil para otra persona!
