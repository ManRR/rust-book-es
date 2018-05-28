## Mejorando nuestro proyecto de E/S

Con este nuevo conocimiento sobre los iteradores, podemos mejorar el proyecto
de E/S en el Capítulo 12 mediante el uso de iteradores para hacer que los
lugares en el código sean más claros y concisos. Veamos cómo los iteradores
pueden mejorar nuestra implementación de la función `Config::new` y la
función `search`.

### Eliminando un `clone` usando un iterador

En el listado 12-6, agregamos un código que tomó una porción de valores
`String` y creó una instancia de la estructura `Config` indexando en el
sector y clonando los valores, permitiendo que la estructura `Config` sea
propietaria de esos valores. En el listado 13-24, hemos reproducido la
implementación de la función `Config::new` tal como estaba en el listado
12-23:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
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

<span class="caption">Listado 13-24: Reproducción de la función
`Config::new` del Listado 12-23</span>

En ese momento, dijimos que no nos preocuparamos por las ineficientes
llamadas `clon` porque las eliminaríamos en el futuro. Bueno, ese momento es
ahora.

Necesitamos `clone` aquí porque tenemos un corte con elementos `String` en el
parámetro `args`, pero la función `new` no posee `args`. Para devolver la
propiedad de una instancia de `Config`, tuvimos que clonar los valores de los
campos `query` y `filename` de `Config` para que la instancia `Config` pueda
poseer sus valores.

Con nuestro nuevo conocimiento acerca de los iteradores, podemos cambiar la
función `new` para tomar posesión de un iterador como argumento en lugar de
tomar prestado un segmento. Utilizaremos la funcionalidad del iterador en
lugar del código que verifica la longitud del sector y los índices en
ubicaciones específicas. Esto aclarará lo que hace la función `Config::new`
porque el iterador tendrá acceso a los valores.

Una vez que `Config::new` toma posesión del iterador y deja de usar las
operaciones de indexación que toman prestado, podemos mover los valores
`String` del iterador a `Config` en lugar de llamar `clone` y hacer una nueva
asignación.

#### Uso del iterador devuelto directamente

Abra el archivo *src/main.rs* del proyecto de E/S, que debería verse
así:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::new(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    // --snip--
}
```

Cambiaremos el inicio de la función `main` que teníamos en el Listado 12-24
al código del Listado 13-25. Esto no se compilará hasta que actualicemos
`Config::new` también.

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
fn main() {
    let config = Config::new(env::args()).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    // --snip--
}
```

<span class="caption">Listado 13-25: Pasando el valor de retorno de
`env::args` a `Config::new`</span>

La función `env::args` devuelve un iterador. En lugar de recopilar los valores del iterador en un vector y luego pasar un *slice* a `Config::new`, ahora estamos pasando la propiedad del iterador devuelto `env::args` a
`Config::new` directamente.

A continuación, debemos actualizar la definición de `Config::new`. En el
archivo *src/lib.rs* de su proyecto de E/S, cambiemos la firma de
`Config::new` para que parezca el Listado 13-26. Esto aún no se compilará
porque necesitamos actualizar el cuerpo de la función.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
impl Config {
    pub fn new(mut args: std::env::Args) -> Result<Config, &'static str> {
        // --snip--
```

<span class="caption">Listado 13-26: Actualizando la firma de `Config::new`
para esperar un iterador</span>

La documentación de la biblioteca estándar para la función `env::args`
muestra que el tipo de iterador que devuelve es `std::env::Args`. Hemos
actualizado la firma de la función `Config::new` para que el parámetro
`args` tenga el tipo `std::env::Args` en lugar de `&[String]`. Como tomamos
posesión de `args` y vamos a mutar `args` al iterar sobre él, podemos agregar
la palabra clave `mut` en la especificación del parámetro `args` para hacerlo
mutable.

#### Usar métodos de *trait* `Iterator` en lugar de indexar

A continuación, corregiremos el cuerpo de `Config::new`. La documentación de
la biblioteca estándar también menciona que `std::env::Args` implementa el
*trait* `Iterator`, ¡así que sabemos que podemos llamar al método `next` en
él! El listado 13-27 actualiza el código del listado 12-23 para usar el
método `next`:

<span class="filename">Filename: src/lib.rs</span>

```rust
# fn main() {}
# use std::env;
#
# struct Config {
#     query: String,
#     filename: String,
#     case_sensitive: bool,
# }
#
impl Config {
    pub fn new(mut args: std::env::Args) -> Result<Config, &'static str> {
        args.next();

        let query = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a query string"),
        };

        let filename = match args.next() {
            Some(arg) => arg,
            None => return Err("Didn't get a file name"),
        };

        let case_sensitive = env::var("CASE_INSENSITIVE").is_err();

        Ok(Config { query, filename, case_sensitive })
    }
}
```

<span class="caption">Listado 13-27: Cambiar el cuerpo de `Config::new`
para usar los métodos de iterador</span>

Recuerde que el primer valor en el valor de retorno de `env::args` es el
nombre del programa. Queremos ignorar eso y llegar al siguiente valor, así
que primero llamamos `next` y no hacemos nada con el valor de retorno.
Segundo, llamamos a `next` para obtener el valor que queremos poner en el
campo `query` de `Config`. Si `next` devuelve `Some`, usamos `match` para
extraer el valor. Si devuelve `None`, significa que no se dieron suficientes
argumentos y que regresamos temprano con un valor `Err`. Hacemos lo mismo
para el valor `filename`.

### Hacer código más claro con adaptadores de iterador

También podemos aprovechar los iteradores en la función `search` en nuestro
proyecto de E/S, que se reproduce aquí en el listado 13-28 como en el listado
12-19:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            results.push(line);
        }
    }

    results
}
```

<span class="caption">Listado 13-28: La implementación de la función
`search` del Listado 12-19</span>

Podemos escribir este código de una manera más concisa utilizando los métodos
del adaptador de iterador. Hacerlo también nos permite evitar tener un vector
intermedio `results` mutable. El estilo de programación funcional prefiere
minimizar la cantidad de estado mutable para hacer que el código sea más
claro. La eliminación del estado mutable podría permitir una mejora futura
para hacer que la búsqueda se realice en paralelo, ya que no tendríamos que
gestionar el acceso simultáneo al vector `results`. El listado 13-29 muestra
este cambio:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    contents.lines()
        .filter(|line| line.contains(query))
        .collect()
}
```

<span class="caption">Listado 13-29: Usar métodos de adaptador de iterador en
la implementación de la función `search`</span>

Recuerde que el propósito de la función `search` es devolver todas las líneas
en `contents` que contienen `query`. Similar al ejemplo `filter` en el
listado 13-19, este código usa el adaptador `filter` para mantener solo las
líneas que `line.contains(query)` devuelve `true`. Luego recogemos las líneas
correspondientes en otro vector con `collect`. ¡Mucho más simple! Siéntase
libre de hacer el mismo cambio para usar los métodos del iterador en la
función `search_case_insensitive` también.

La siguiente pregunta lógica es qué estilo debe elegir en su propio código y
por qué: la implementación original en el listado 13-28 o la versión que usa
iteradores en el listado 13-29. La mayoría de los programadores de Rust
prefieren usar el estilo de iterador. Es un poco más difícil controlarlo al
principio, pero una vez que te haces una idea de los diversos adaptadores de
iterador y lo que hacen, los iteradores pueden ser más fáciles de entender.
En lugar de jugar con los diversos bits de bucle y construir nuevos vectores,
el código se centra en el objetivo de alto nivel del bucle. Esto abstrae
parte del código común para que sea más fácil ver los conceptos que son
exclusivos de este código, como la condición de filtrado que debe pasar cada
elemento en el iterador.

Pero, ¿son las dos implementaciones verdaderamente equivalentes? La
suposición intuitiva podría ser que cuanto más bajo sea el ciclo será más
rápido. Hablemos de rendimiento.