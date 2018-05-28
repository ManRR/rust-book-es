# Enums y patrones de *concordancias *

En este capítulo veremos *enumerations* (*enumeraciones*), también conocidas como *enums*.
Los enumeraciones le permiten definir un tipo enumerando sus posibles valores.
Primero, definiremos y usaremos una enumeración para mostrar cómo una enumeración puede
codificar el significado junto con los datos. A continuación, exploraremos una enumeración
particularmente útil, llamada `Opción`, que expresa que un valor puede ser algo o nada.
Luego veremos cómo la coincidencia de patrones en la expresión `match` hace que sea fácil
ejecutar código diferente para diferentes valores de una enumeración. Finalmente, cubriremos
cómo el constructo `if let` es otro idioma conveniente y conciso disponible para manejar
enumeraciones en su código.

Los *enums* son una característica en muchos lenguajes, pero sus capacidades difieren
en algunos de ellos. Las enumeraciones de Rust son más similares a *tipos de datos algebraicos*
(*algebraic data types*) en lenguajes funcionales, como F#, OCaml y Haskell.
