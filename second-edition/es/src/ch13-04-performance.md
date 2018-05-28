## Comparación del rendimiento: bucles contra iteradores

Para determinar si se deben usar bucles o iteradores, se necesita saber qué
versión de nuestras funciones de `search` es más rápida: la versión con un
bucle `for` explícito o la versión con iteradores.

Ejecutamos un punto de referencia cargando todo el contenido de
*Las aventuras de Sherlock Holmes* de Sir Arthur Conan Doyle en un `String` y
buscando la palabra *the* en los contenidos. Aquí están los resultados del
*benchmark* en la versión de `search` usando el bucle `for` y la versión que
usa iteradores:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

¡La versión del iterador fue ligeramente más rápida! Aquí no explicaremos el
código de referencia, porque el objetivo no es probar que las dos versiones
son equivalentes, sino obtener una idea general de cómo estas dos
implementaciones se comparan en términos de rendimiento.

Para obtener un punto de referencia más completo, debe verificar el uso de
varios textos de varios tamaños como los `contents`, diferentes palabras y
palabras de diferentes longitudes como la `query` y todo tipo de otras
variaciones. El punto es este: los iteradores, a pesar de ser una abstracción
de alto nivel, se compilan aproximadamente en el mismo código como si
hubieras escrito tú mismo el código de nivel inferior. Los iteradores son una
de las *abstracciones de costo cero* de Rust, con lo cual nos referimos a que
el uso de la abstracción no impone una sobrecarga de tiempo de ejecución
adicional. Esto es análogo a cómo Bjarne Stroustrup, el diseñador original e
implementador de C ++, define *zero-overhead* en “Foundations of C++” (2012):

> En general, las implementaciones de C ++ obedecen al principio de cero
> gastos: lo que no se usa, no se paga. Y más: lo que sí utilizas, no podrías
> codificar mejor.

Como otro ejemplo, el siguiente código se toma de un decodificador de audio.
El algoritmo de decodificación utiliza la operación matemática de predicción
lineal para estimar los valores futuros en función de una función lineal de
las muestras anteriores. Este código usa una cadena de iteradores para hacer
algunas operaciones matemáticas en tres variables en el alcance: una porción
de datos `buffer`, una matriz de 12 `coefficients`, y una cantidad por la
cual se cambian los datos en `qlp_shift`. Hemos declarado las variables
dentro de este ejemplo pero no les hemos dado ningún valor; aunque este
código no tiene mucho significado fuera de su contexto, sigue siendo un
ejemplo conciso y real de cómo Rust traduce las ideas de alto nivel al código
de bajo nivel.

```rust,ignore
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

Para calcular el valor de `prediction`, este código recorre cada uno de los
12 valores en `coefficients` y utiliza el método `zip` para emparejar los
valores de los coeficientes con los 12 valores anteriores en `buffer`. Luego,
para cada par, multiplicamos los valores juntos, sumamos todos los resultados
y cambiamos los bits en la suma de los bits `qlp_shift` a la derecha.

Los cálculos en aplicaciones como los decodificadores de audio a menudo
priorizan el rendimiento más altamente. Aquí, estamos creando un iterador,
usando dos adaptadores y luego consumiendo el valor. ¿Qué código de
ensamblado compilará este código Rust? Bueno, a partir de este escrito, se
compila en la misma asamblea que escribirías a mano. No hay ningún bucle
correspondiente a la iteración sobre los valores en `coefficients`: Rust sabe
que hay 12 iteraciones, por lo que “desenrolla” el bucle. *Desenrollar*
(*Unrolling*) es una optimización que elimina la sobrecarga del código de
control de bucle y en su lugar genera código repetitivo para cada iteración
del bucle.

Todos los coeficientes se almacenan en registros, lo que significa que el
acceso a los valores es muy rápido. No hay límites de verificación en el
acceso a la matriz en tiempo de ejecución. Todas estas optimizaciones que
Rust puede aplicar hacen que el código resultante sea extremadamente
eficiente. ¡Ahora que sabes esto, puedes usar iteradores y *closures* sin
miedo! Hacen que el código parezca tener un nivel más alto pero no imponen
una penalización en el rendimiento en tiempo de ejecución por hacerlo.

## Resumen

Los *closures* e iteradores son funciones de Rust inspiradas en ideas de
lenguaje de programación funcional. Contribuyen a la capacidad de Rust para
expresar claramente ideas de alto nivel en el desempeño de bajo nivel. Las
implementaciones de *closures* e iteradores son tales que el rendimiento del
tiempo de ejecución no se ve afectado. Esto es parte del objetivo de Rust de
esforzarse por proporcionar *abstracciones de costo cero*.

Ahora que hemos mejorado la expresividad de nuestro proyecto de E/S, veamos
algunas características más de `cargo` que nos ayudarán a compartir el
proyecto con el mundo.
