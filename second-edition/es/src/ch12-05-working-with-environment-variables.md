## Trabajando con variables de entorno

Mejoraremos `minigrep` agregando una característica adicional: una opción
para la búsqueda de mayúsculas y minúsculas que el usuario puede activar a
través de una variable de entorno. Podríamos hacer que esta característica
sea una opción de línea de comando y requerir que los usuarios la ingresen
cada vez que quieran que se aplique, pero en su lugar usaremos una variable
de entorno. Hacerlo permite a nuestros usuarios establecer la variable de
entorno una vez y hacer que todas sus búsquedas sean insensibles a las
mayúsculas y minúsculas en esa sesión de terminal.

### Escribir una prueba de falla para la función `search` sensible a mayúsculas y minúsculas

Queremos agregar una nueva función `search_case_insensitive` a la que
llamaremos cuando la variable de entorno esté activada. Seguiremos siguiendo
el proceso TDD, por lo que el primer paso es volver a escribir una prueba
fallida. Añadiremos una nueva prueba para la nueva función
`search_case_insensitive` y cambiaremos el nombre de nuestra antigua prueba
de `one_result` a `case_sensitive` para aclarar las diferencias entre las dos
pruebas, como se muestra en el listado 12-20.

<span class="filename">Filename: src/lib.rs</span>

```rust
#[cfg(test)]
mod test {
    use super::*;

    #[test]
    fn case_sensitive() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Duct tape.";

        assert_eq!(
            vec!["safe, fast, productive."],
            search(query, contents)
        );
    }

    #[test]
    fn case_insensitive() {
        let query = "rUsT";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.
Trust me.";

        assert_eq!(
            vec!["Rust:", "Trust me."],
            search_case_insensitive(query, contents)
        );
    }
}
```

<span class="caption">Listado 12-20: Agregar una nueva prueba de falla para
la función sensible a mayúsculas y minúsculas que estamos a punto de
agregar</span>

Tenga en cuenta que también hemos editado el `contents` de la prueba
anterior. Hemos agregado una nueva línea con el texto `"Duct tape."` Usando
una D mayúscula que no debe coincidir con la consulta `"duct"` cuando estamos
buscando casos sensibles. Cambiar la prueba anterior de esta manera ayuda a
garantizar que no rompamos accidentalmente la funcionalidad de búsqueda
sensible a mayúsculas y minúsculas que ya hemos implementado. Esta prueba
debe pasar ahora y continuar pasando a medida que trabajamos en la búsqueda
sensible a mayúsculas y minúsculas.

La nueva prueba para la búsqueda *sensible* usa `"rUsT"` como consulta. En la
función `search_case_insensitive` que estamos a punto de agregar, la
consulta `"rUsT"` debe coincidir con la línea que contiene `"Rust:"`con una R
mayúscula y hacer coincidir la línea `"Trust me."` aunque ambos tengan
carcasa diferente a la consulta. Esta es nuestra prueba de falla, y no
compilará porque aún no hemos definido la función `search_case_insensitive`.
Siéntase libre de agregar una implementación de esqueleto que siempre
devuelve un vector vacío, similar a la forma en que lo hicimos para la
función `search` en el listado 12-16 para ver la compilación de prueba y
fallar.

### Implementando la función `search_case_insensitive`

La función `search_case_insensitive`, que se muestra en el listado 12-21,
será casi la misma que la función `search`. La única diferencia es que vamos
a escribir en minúscula la `query` y cada `líne`, así que cualquiera sea el
caso de los argumentos de entrada, serán el mismo caso cuando verifiquemos si
la línea contiene la consulta.

<span class="filename">Filename: src/lib.rs</span>

```rust
pub fn search_case_insensitive<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let query = query.to_lowercase();
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.to_lowercase().contains(&query) {
            results.push(line);
        }
    }

    results
}
```

<span class="caption">Listado 12-21: Definir la función
`search_case_insensitive` para minúsculas de la consulta y la línea antes de
compararlas</span>

Primero, minamos la cadena `query` y la almacenamos en una variable sombreada
con el mismo nombre. Llamar `to_lowercase` en la consulta es necesario así
que no importa si la consulta del usuario es `"rust"`,`"RUST"`,`"Rust:"`,
o `"rUsT"`, trataremos la consulta como si fuera `"rust"` y ser insensible al
asunto.

Tenga en cuenta que `query` ahora es un `String` en lugar de un
*string slice*, porque la invocación `to_lowercase` crea nuevos datos en
lugar de hacer referencia a los datos existentes. Digamos que la consulta es
`"rUsT"`, como un ejemplo: ese *string slice* no contiene `u` o `t`
minúsculas para que lo usemos, así que tenemos que asignar un nuevo `String`
que contenga `"rust"`. Cuando pasamos `query` como un argumento para el
método `contains` ahora, necesitamos agregar un *ampersand* ya que la firma
de `contains` está definida para tomar un *string slice*.

A continuación, agregamos una llamada a `to_lowercase` en cada `line` antes
de verificar si contiene `query` para minúsculas de todos los caracteres.
Ahora que convertimos `line` y `query` a minúsculas, encontraremos
coincidencias sin importar el caso de la consulta.

Veamos si esta implementación pasa las pruebas:

```text
running 2 tests
test test::case_insensitive ... ok
test test::case_sensitive ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

¡Estupendo! Ellas pasaron. Ahora, llamemos a la nueva función
`search_case_insensitive` de la función `run`. Primero, agregaremos una
opción de configuración a la estructura `Config` para cambiar entre búsqueda
sensible a mayúsculas y minúsculas. Agregar este campo provocará errores en
el compilador porque aún no estamos inicializando este campo en ninguna parte:

<span class="filename">Filename: src/lib.rs</span>

```rust
pub struct Config {
    pub query: String,
    pub filename: String,
    pub case_sensitive: bool,
}
```

Tenga en cuenta que agregamos el campo `case_sensitive` que contiene un
booleano. A continuación, necesitamos la función `run` para verificar el
valor del campo `case_sensitive` y usar eso para decidir si se llama a la
función `search` o a la función `search_case_insensitive`, como se muestra en
el Listado 12-22. Tenga en cuenta que esto todavía no se compilará.

<span class="filename">Filename: src/lib.rs</span>

```rust
# use std::error::Error;
# use std::fs::File;
# use std::io::prelude::*;
#
# fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
#      vec![]
# }
#
# pub fn search_case_insensitive<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
#      vec![]
# }
#
# struct Config {
#     query: String,
#     filename: String,
#     case_sensitive: bool,
# }
#
pub fn run(config: Config) -> Result<(), Box<Error>> {
    let mut f = File::open(config.filename)?;

    let mut contents = String::new();
    f.read_to_string(&mut contents)?;

    let results = if config.case_sensitive {
        search(&config.query, &contents)
    } else {
        search_case_insensitive(&config.query, &contents)
    };

    for line in results {
        println!("{}", line);
    }

    Ok(())
}
```

<span class="caption">Listado 12-22: Llamar a `search` o
`search_case_insensitive` en función del valor en `config.case_sensitive`</span>

Finalmente, necesitamos verificar la variable de entorno. Las funciones para
trabajar con variables de entorno se encuentran en el módulo `env` de la
biblioteca estándar, por lo que queremos poner ese módulo dentro del alcance
con una línea `use std::env;` en la parte superior de *src/lib.rs* . Luego
usaremos la función `var` del módulo `env` para buscar una variable de
entorno llamada `CASE_INSENSITIVE`, como se muestra en el listado 12-23.

<span class="filename">Filename: src/lib.rs</span>

```rust
use std::env;
# struct Config {
#     query: String,
#     filename: String,
#     case_sensitive: bool,
# }

// --snip--

impl Config {
    pub fn new(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let filename = args[2].clone();

        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

        Ok(Config { query, filename, case_sensitive })
    }
}
```

<span class="caption">Listado 12-23: Comprobación de una variable de entorno
llamada `CASE_INSENSITIVE`</span>

Aquí, creamos una nueva variable `case_sensitive`. Para establecer su valor,
llamamos a la función `env::var` y le pasamos el nombre de la variable de
entorno` CASE_INSENSITIVE`. La función `env::var` devuelve un `Result` que
será la variante exitosa `Ok` que contiene el valor de la variable de entorno
si se establece la variable de entorno. Devolverá la variante `Err` si la
variable de entorno no está configurada.

Estamos utilizando el método `is_err` en el `Result` para comprobar si se
trata de un error y, por lo tanto, no está configurado, lo que significa que
*debería* hacer una búsqueda sensible a mayúsculas y minúsculas. Si la
variable de entorno `CASE_INSENSITIVE` se configura en cualquier cosa,
`is_err` devolverá false y el programa realizará una búsqueda que no
distingue entre mayúsculas y minúsculas. No nos importa el *valor* de la
variable de entorno, solo si está configurada o no, así que estamos
verificando `is_err` en lugar de usar `unwrap`, `expect`, o cualquiera de los
otros métodos que tenemos visto en `Result`.

Pasamos el valor en la variable `case_sensitive` a la instancia `Config` para
que la función `run` pueda leer ese valor y decidir si se llama `search` o
`search_case_insensitive`, como implementamos en el listado 12-22.

¡Hagamos un intento! Primero, ejecutaremos nuestro programa sin la variable
de entorno establecida y con la consulta `to`, que debe coincidir con
cualquier línea que contenga la palabra “to” en minúsculas:

```text
$ cargo run to poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep to poem.txt`
Are you nobody, too?
How dreary to be somebody!
```

¡Parece que todavía funciona! Ahora, ejecutemos el programa con
`CASE_INSENSITIVE` establecido en `1` pero con la misma consulta `to`.

Si está usando PowerShell, necesitará establecer la variable de entorno y
ejecutar el programa en dos comandos en lugar de uno:

```text
$ $env:CASE_INSENSITIVE=1
$ cargo run to poem.txt
```

Deberíamos obtener líneas que contengan “to” que podrían tener letras
mayúsculas:

```text
$ CASE_INSENSITIVE=1 cargo run to poem.txt
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep to poem.txt`
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Excelente, ¡también tenemos líneas que contienen “To”! Nuestro programa
`minigrep` ahora puede hacer búsquedas sensibles a mayúsculas y minúsculas
controladas por una variable de entorno. Ahora sabe cómo administrar las
opciones establecidas utilizando argumentos de línea de comando o variables
de entorno.

Algunos programas permiten argumentos *y* variables de entorno para la misma
configuración. En esos casos, los programas deciden que uno u otro tiene
prioridad. Para otro ejercicio por su cuenta, intente controlar la
sensibilidad de mayúsculas y minúsculas mediante un argumento de línea de
comando o una variable de entorno. Decida si el argumento de la línea de
comando o la variable de entorno debe tener prioridad si el programa se
ejecuta con un conjunto sensible a mayúsculas y minúsculas y un conjunto
de insensible a mayúsculas y minúsculas.

El módulo `std::env` contiene muchas más funciones útiles para tratar las
variables de entorno: consulte su documentación para ver qué hay disponible.
