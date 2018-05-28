## Comentarios

Todos los programadores se esfuerzan por hacer que su código sea fácil de
entender, pero a veces una explicación adicional está justificada. En estos casos,
los programadores dejan notas, o *comentarios*, en su código fuente que el compilador
ignorará pero para la gente que lea el código fuente le puede ser útil.

Aquí hay un comentario simple:

```rust
// hello, world
```

En Rust, los comentarios deben comenzar con dos barras y continuar hasta el fin de
línea. Para comentarios que se extienden más allá de una sola línea, deberá incluir
`//` en cada línea, así:

```rust
// So we’re doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what’s going on.
```

Los comentarios también pueden colocarse al final de las líneas que contienen el código:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    let lucky_number = 7; // I’m feeling lucky today
}
```

Pero con mayor frecuencia los verá usar en el siguiente formato, con el comentario
en un línea separada sobre el código que está anotando:

<span class="filename">Filename: src/main.rs</span>

```rust
fn main() {
    // I’m feeling lucky today
    let lucky_number = 7;
}
```

Rust también tiene otro tipo de comentario, comentarios de documentación, que
trataremos en el Capítulo 14.
