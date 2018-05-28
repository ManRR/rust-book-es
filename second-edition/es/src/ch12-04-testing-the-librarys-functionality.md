## Desarrollar la funcionalidad de la biblioteca con desarrollo basado en pruebas (*Test-driven development (TDD)*)

Ahora que hemos extraído la lógica en *src/lib.rs* y dejamos la recolección
de los argumentos y el manejo de errores en *src/main.rs*, es mucho más fácil
escribir pruebas para la funcionalidad central de nuestro código. Podemos
llamar funciones directamente con varios argumentos y verificar valores
devueltos sin tener que llamar a nuestro binario desde la línea de comando.
Siéntete libre de escribir algunas pruebas para la funcionalidad en las
funciones `Config::new` y `run` por su cuenta.

En esta sección, agregaremos la lógica de búsqueda al programa `minigrep` utilizando el proceso de *Test-driven development (TDD)*. Esta técnica de desarrollo de software sigue estos pasos:

1. Escriba una prueba que falla y ejecútala para asegurarte de que falla por
 el motivo que esperas.
2. Escriba o modifique el código justo para hacer pasar la nueva prueba.
3. Refactorice el código que acaba de agregar o cambiar y asegúrese de que
 las pruebas continúen pasando.
4. ¡Repite desde el paso 1!

Este proceso es solo una de las muchas maneras de escribir software, pero TDD
también puede ayudar a impulsar el diseño del código. Escribir la prueba
antes de escribir el código que hace que pase la prueba ayuda a mantener una
cobertura de prueba alta durante todo el proceso.

Probará la implementación de la funcionalidad que realmente buscará el *string* de consulta en el contenido del archivo y generará una lista de
líneas que coincidan con la consulta. Agregaremos esta funcionalidad en una
función llamada `search`.

### Escribir una prueba fallida

Como ya no los necesitamos, eliminemos las sentencias `println!` de
*src/lib.rs* y *src/main.rs* que usamos para verificar el comportamiento del
programa. Luego, en *src/lib.rs*, agregaremos un módulo `test` con una
función de prueba, como hicimos en el Capítulo 11. La función de prueba
especifica el comportamiento que queremos que tenga la función `search`: lo
hará tome una consulta y el texto para buscar la consulta, y devolverá solo
las líneas del texto que contiene la consulta. El listado 12-15 muestra esta
prueba, que aún no se compilará.

<span class="filename">Filename: src/lib.rs</span>

```rust
# fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
#      vec![]
# }
#
#[cfg(test)]
mod test {
    use super::*;

    #[test]
    fn one_result() {
        let query = "duct";
        let contents = "\
Rust:
safe, fast, productive.
Pick three.";

        assert_eq!(
            vec!["safe, fast, productive."],
            search(query, contents)
        );
    }
}
```

<span class="caption">Listado 12-15: Creando una prueba de falla para la
función `search` que deseamos tener</span>

Esta prueba busca la cadena `"duct"`. El texto que estamos buscando tiene
tres líneas, de las cuales solo una contiene `"duct"`. Afirmamos que el valor
devuelto por la función `search` contiene solo la línea que esperamos.

No podemos ejecutar esta prueba y verla fallar porque la prueba ni siquiera
se compila: ¡la función `search` aún no existe! Así que ahora agregaremos
solo el código suficiente para hacer que la prueba se compile y ejecute al
agregar una definición de la función `search` que siempre devuelve un vector
vacío, como se muestra en el Listado 12-16. Entonces la prueba debería
compilarse y fallar porque un vector vacío no coincide con un vector que
contiene la línea `"safe, fast, productive."`.

<span class="filename">Filename: src/lib.rs</span>

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    vec![]
}
```

<span class="caption">Listado 12-16: Definir lo suficiente de la función
`search` para que nuestra prueba compile</span>

Tenga en cuenta que necesitamos un *lifetime* explícito `a'` definido en la
firma de `search` y utilizado con el argumento `contents` y el valor de
retorno. Recuerde en el Capítulo 10 que los parámetros de *lifetime*
especifican qué *lifetime* del argumento está conectada a el *lifetime* del
valor de retorno. En este caso, indicamos que el vector devuelto debe
contener *string slices* que hacen referencia a *slices* del argumento
`contents` (en lugar del argumento `query`).

En otras palabras, le decimos a Rust que los datos devueltos por la función
`search` vivirán mientras los datos pasen a la función `search` en el
argumento `contents`. ¡Esto es importante! Los datos a los que se hace
referencia *por* un *slice* deben ser válidos para que la referencia sea
válida; si el compilador asume que estamos haciendo *string slices* de
`query` en lugar de `contents`, hará su comprobación de seguridad de forma
incorrecta.

Si olvidamos las anotaciones de *lifetime* y tratamos de compilar esta
función, obtendremos este error:

```text
error[E0106]: missing lifetime specifier
 --> src/lib.rs:5:51
  |
5 | pub fn search(query: &str, contents: &str) -> Vec<&str> {
  |                                                   ^ expected lifetime
parameter
  |
  = help: this function's return type contains a borrowed value, but the
  signature does not say whether it is borrowed from `query` or `contents`
```

Rust no puede saber cuál de los dos argumentos necesitamos, así que tenemos
que contarlo. Debido a que `contents` es el argumento que contiene todo
nuestro texto y queremos devolver las partes de ese texto que coinciden,
sabemos `contents` es el argumento que debe conectarse al valor de retorno
utilizando la sintaxis de duración.

Otros lenguajes de programación no requieren que conecte argumentos para
devolver valores en la firma. Aunque esto pueda parecer extraño, será más
fácil con el tiempo. Es posible que desee comparar este ejemplo con la
sección “Validación de referencias con *Lifetimes*” en el Capítulo
10.

Ahora ejecutemos la prueba:

```text
$ cargo test
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
--warnings--
    Finished dev [unoptimized + debuginfo] target(s) in 0.43 secs
     Running target/debug/deps/minigrep-abcabcabc

running 1 test
test test::one_result ... FAILED

failures:

---- test::one_result stdout ----
        thread 'test::one_result' panicked at 'assertion failed: `(left ==
right)`
left: `["safe, fast, productive."]`,
right: `[]`)', src/lib.rs:48:8
note: Run with `RUST_BACKTRACE=1` for a backtrace.


failures:
    test::one_result

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured; 0 filtered out

error: test failed, to rerun pass '--lib'
```

Genial, la prueba falla, exactamente como esperábamos. ¡Hagamos pasar la
prueba!

### Escribir código para pasar la prueba

Actualmente, nuestra prueba está fallando porque siempre devolvemos un vector
vacío. Para solucionar eso e implementar `search`, nuestro programa debe
seguir estos pasos:

* Iterar a través de cada línea de los contenidos.
* Verifique si la línea contiene nuestro texto de consulta.
* Si lo hace, agréguelo a la lista de valores que estamos devolviendo.
* Si no es así, no hagas nada.
* Devuelve la lista de resultados que coinciden.

Vamos a trabajar en cada paso, comenzando con iterar a través de las líneas.

#### Iterar a través de las líneas con el método `lines`

Rust tiene un método útil para manejar la iteración línea por línea de
*string*, convenientemente llamadas `lines`, que funciona como se muestra en
el listado 12-17. Tenga en cuenta que esto aún no se compilará.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
        // do something with line
    }
}
```

<span class="caption">Listado 12-17: iteración a través de cada línea en
`contents`</span>

El método `lines` devuelve un iterador. Hablaremos sobre iteradores en
profundidad en el Capítulo 13, pero recuerde que vio esta forma de usar un
iterador en el Listado 3-5, donde usamos un bucle `for` con un iterador para
ejecutar un código en cada elemento de una colección.

#### Buscando cada línea para la consulta

A continuación, comprobaremos si la línea actual contiene nuestro *sting* de
consulta. Afortunadamente, los *string* tienen un método útil llamado
`contains` que hace esto por nosotros. Agregue una llamada al método
`contains` en la función `search`, como se muestra en el Listado 12-18. Tenga
en cuenta que esto todavía no se compilará.

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    for line in contents.lines() {
        if line.contains(query) {
            // do something with line
        }
    }
}
```

<span class="caption">Listado 12-18: Agregar funcionalidad para ver si la
línea contiene el *string* en `query`</span>

#### Almacenamiento de líneas coincidentes

También necesitamos una forma de almacenar las líneas que contienen nuestra
*string* de consulta. Para eso, podemos hacer un vector mutable antes del
bucle `for` y llamar al método `push` para almacenar una `line` en el vector.
Después del ciclo `for`, devolvemos el vector, como se muestra en el listado
12-19.

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

<span class="caption">Listado 12-19: Almacenando las líneas que coinciden
para que podamos devolverlas</span>

Ahora la función `search` debería devolver solo las líneas que contienen
`query`, y nuestra prueba debería pasar. Vamos a ejecutar la prueba:

```text
$ cargo test
--snip--
running 1 test
test test::one_result ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out
```

¡Nuestra prueba pasó, entonces sabemos que funciona!

En este punto, podríamos considerar oportunidades para refactorizar la
implementación de la función de búsqueda mientras se mantienen las pruebas
para mantener la misma funcionalidad. El código en la función de búsqueda no
es tan malo, pero no aprovecha algunas características útiles de los
iteradores. Volveremos a este ejemplo en el Capítulo 13, donde exploraremos
los iteradores en detalle y veremos cómo mejorarlo.

#### Usando la función `search` en la función `run`

Ahora que la función `search` está trabajando y probada, necesitamos llamar
`search` desde nuestra función `run`. Necesitamos pasar el valor
`config.query` y el `contents` que `run` lee desde el archivo a la función
`search`. Luego `run` imprimirá cada línea devuelta desde `search`:

<span class="filename">Filename: src/lib.rs</span>

```rust,ignore
pub fn run(config: Config) -> Result<(), Box<Error>> {
    let mut f = File::open(config.filename)?;

    let mut contents = String::new();
    f.read_to_string(&mut contents)?;

    for line in search(&config.query, &contents) {
        println!("{}", line);
    }

    Ok(())
}
```

Todavía estamos usando un bucle `for` para devolver cada línea de `search` e
imprimirla.

¡Ahora todo el programa debería funcionar! Probémoslo, primero con una
palabra que debería devolver exactamente una línea del poema de Emily Dickinson, “frog”:

```text
$ cargo run frog poem.txt
   Compiling minigrep v0.1.0 (file:///projects/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.38 secs
     Running `target/debug/minigrep frog poem.txt`
How public, like a frog
```

¡Guay!. Ahora intentemos una palabra que coincida con varias líneas, como
“body”:

```text
$ cargo run body poem.txt
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep body poem.txt`
I’m nobody! Who are you?
Are you nobody, too?
How dreary to be somebody!
```

Y finalmente, asegurémonos de no obtener ninguna línea cuando buscamos una
palabra que no esté en ningún lugar del poema, como “monomorphization”:

```text
$ cargo run monomorphization poem.txt
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/minigrep monomorphization poem.txt`
```

¡Excelente! Creamos nuestra propia versión mini de una herramienta clásica y
aprendimos mucho sobre cómo estructurar las aplicaciones. También aprendimos
un poco sobre la entrada y salida de archivos, el *lifetimes*, las pruebas y
el análisis de línea de comando.

Para completar este proyecto, mostraremos brevemente cómo trabajar con
variables de entorno y cómo imprimir a error estándar, las cuales son útiles
cuando se escriben programas de línea de comando.
