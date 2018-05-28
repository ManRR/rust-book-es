## Características de los lenguajes orientados a objetos

No hay consenso en la comunidad de programación sobre qué características
debe tener un lenguaje para ser considerado orientado a objetos. Rust está
influenciado por muchos paradigmas de programación, incluido OOP; por ejemplo
exploramos las características que provienen de la programación funcional en
el Capítulo 13. Podría decirse que los lenguajes OOP comparten ciertas
características comunes, a saber, objetos, encapsulación y herencia. Veamos
qué significa cada una de esas características y si Rust lo admite.

### Los objetos contienen datos y comportamiento

El libro *Design Patterns: Elements of Reusable Object-Oriented Software*
(*Patrones de diseño: Elementos del software reutilizable orientado a
objetos*) por Enoch Gamma, Richard Helm, Ralph Johnson y John Vlissides
(Addison-Wesley Professional, 1994) conocido coloquialmente como el libro
*The Gang of Four*
(*Banda de los Cuatro*), es un catálogo de patrones de diseño orientados a
objetos. Define OOP de esta manera:

> Los programas orientados a objetos están formados por objetos. Un *objeto* > empaqueta ambos
> los datos y los procedimientos que operan en esos datos. Los procedimientos
> son
> típicamente llamados *métodos* o *operaciones*.

Usando esta definición, Rust está orientado a objetos: las estructuras y las
enumeraciones tienen datos, y los bloques `impl` proporcionan métodos en las
estructuras y las enumeraciones. A pesar de que las estructuras y las
enumeraciones con métodos no se *llaman* objetos, proporcionan la misma
funcionalidad, de acuerdo con la definición de objetos de la Banda de los
Cuatro.

### Encapsulación que oculta los detalles de implementación

Otro aspecto comúnmente asociado con OOP es la idea de *encapsulation*, lo
que significa que los detalles de implementación de un objeto no son
accesibles para codificar utilizando ese objeto. Por lo tanto, la única forma
de interactuar con un objeto es a través de su API pública; el código que
utiliza el objeto no debería poder acceder a las partes internas del objeto y
cambiar los datos o el comportamiento directamente. Esto permite al
programador cambiar y refactorizar las partes internas de un objeto sin
necesidad de cambiar el código que usa el objeto.

Discutimos cómo controlar la encapsulación en el Capítulo 7: podemos usar la
palabra clave `pub` para decidir qué módulos, tipos, funciones y métodos en
nuestro código deberían ser públicos, y por defecto todo lo demás es privado.
Por ejemplo, podemos definir una estructura `AveragedCollection` que tiene un
campo que contiene un vector de valores `i32`. La estructura también puede
tener un campo que contiene el promedio de los valores en el vector, lo que
significa que el promedio no tiene que calcularse a pedido siempre que
alguien lo necesite. En otras palabras, `AveragedCollection` almacenará en
caché el promedio calculado para nosotros. El listado 17-1 tiene la
definición de la estructura `AveragedCollection`:

<span class="filename">Filename: src/lib.rs</span>

```rust
pub struct AveragedCollection {
    list: Vec<i32>,
    average: f64,
}
```

<span class="caption">Listado 17-1: una estructura `AveragedCollection` que
mantiene una lista de enteros y el promedio de los elementos en la
colección</span>

La estructura está marcada como `pub` para que otros códigos puedan usarla,
pero los campos dentro de la estructura permanecen privados. Esto es
importante en este caso porque queremos asegurarnos de que cada vez que se
agregue o elimine un valor de la lista, el promedio también se actualice.
Hacemos esto implementando los métodos `add`, `remove` y `average` en la
estructura, como se muestra en el Listado 17-2:

<span class="filename">Filename: src/lib.rs</span>

```rust
# pub struct AveragedCollection {
#     list: Vec<i32>,
#     average: f64,
# }
impl AveragedCollection {
    pub fn add(&mut self, value: i32) {
        self.list.push(value);
        self.update_average();
    }

    pub fn remove(&mut self) -> Option<i32> {
        let result = self.list.pop();
        match result {
            Some(value) => {
                self.update_average();
                Some(value)
            },
            None => None,
        }
    }

    pub fn average(&self) -> f64 {
        self.average
    }

    fn update_average(&mut self) {
        let total: i32 = self.list.iter().sum();
        self.average = total as f64 / self.list.len() as f64;
    }
}
```

<span class="caption">Listado 17-2: Implementaciones de los métodos públicos
`add`, `remove`, y `average` en `AveragedCollection`</span>

Los métodos públicos `add`, `remove` y `average` son las únicas formas de
modificar una instancia de `AveragedCollection`. Cuando un elemento se agrega
a `list` usando el método `add` o eliminado con el método `remove`, las
implementaciones de cada llama al método privado `update_average` que maneja
la actualización del `average` campo también.

Dejamos los campos `list` y `average` en privado, por lo que no hay forma de
código externo para agregar o eliminar elementos directamente al campo `list`
de otra manera, el campo `average` podría perder su sincronización cuando
cambie la `list`.El método `average` devuelve el valor en el campo
`average`, lo que permite código para leer el `average` pero no modificarlo.

Porque hemos encapsulado los detalles de implementación de la estructura
`AveragedCollection`, podemos cambiar fácilmente aspectos, como la estructura
de datos, en el futuro. Por ejemplo, podríamos usar un `HashSet<i32>` en
lugar de un `Vec<i32>` para el campo `list`. Siempre y cuando las firmas del
`add`, los métodos públicos `remove`, y `average` permanecen igual, código
usando `AveragedCollection` no necesitaría cambiar. Si hiciéramos público
`list`, esto no sería necesariamente el caso: `HashSet<i32>` y `Vec<i32>`
tienen diferentes métodos para agregar y eliminar elementos, por lo que el
código externo probablemente tenga que cambiar si modificara `list`
directamente.

Si la encapsulación es un aspecto requerido para que un lenguaje se considere
objeto orientado, entonces Rust cumple con ese requisito. La opción de usar
`pub` o no para diferentes partes de código permiten la encapsulación de
detalles de implementación.

### Herencia como un sistema tipo y como código compartido

*Inheritance* (*Herencia*) es un mecanismo mediante el cual un objeto puede
heredar de otro definición del objeto, obteniendo así los datos y el
comportamiento del objeto padre sin tienes que definirlos de nuevo.

Si un lenguaje debe tener herencia para ser un lenguaje orientado a objetos,
entonces Rust no es uno. No hay forma de definir una estructura que herede del
padre los campos de struct y las implementaciones de métodos. Sin embargo, si
estás acostumbrado a tener herencia en su caja de herramientas de
programación, puede usar otras soluciones en Rust, dependiendo de su razón
para alcanzar la herencia en primer lugar.

Usted elige la herencia por dos razones principales. Una es para reutilizar
el código: puedes implementar un comportamiento particular para un tipo, y la
herencia le permite reutilizar esa implementación para un tipo diferente.
Puedes compartir el código de Rust usando implementaciones de método de
*trait* predeterminado, que viste en el Listado 10-14
cuando agregamos una implementación predeterminada del método `summarize` en
el *trait* `Summary`. Cualquier tipo que implemente el *trait* `Summary`
tendría el método `summarize` disponible sin ningún otro código. Esto es
similar a una clase padre que tiene una implementación de un método y un
hijo heredera clase también teniendo la implementación del método. También
podemos anular el implementación predeterminada del método `summarize` cuando
implementamos el *trait* `Summary`, que es similar a una clase secundaria que
anula la implementación de un método heredado de una clase padre.

La otra razón para usar la herencia se relaciona con el sistema de tipo: para
habilitar un tipo de hijo para ser utilizado en los mismos lugares que el
tipo principal. Esto es también llamado *polimorfismo*, lo que significa que
puede sustituir varios objetos entre sí en tiempo de ejecución si comparten
ciertas características.

> ### Polimorfismo
>
> Para muchas personas, el polimorfismo es sinónimo de herencia. Pero en
> realidad es un concepto más general que se refiere a un código que puede
> funcionar con datos de múltiples tipos. Para la herencia, esos tipos son
> generalmente subclases.
>
> En su lugar, Rust utiliza genéricos para abstraer sobre diferentes tipos
> posibles y límites de *trait* para imponer restricciones sobre lo que
> dichos tipos deben proporcionar. Esto a veces se llama
> *bounded parametric polymorphism* (*polimorfismo paramétrico limitado*).

La herencia ha caído en desuso recientemente como una solución de diseño de
programación en muchos lenguajes de programación porque a menudo corre el
riesgo de compartir más código de lo necesario. Las subclases no siempre
deben compartir todas las características de su clase principal, pero lo
harán con la herencia. Esto puede hacer que el diseño de un programa sea
menos flexible. También introduce la posibilidad de llamar a métodos en
subclases que no tienen sentido o que causan errores porque los métodos no se
aplican a la subclase. Además, algunos idiomas solo permitirán que una
subclase herede de una clase, restringiendo aún más la flexibilidad del
diseño de un programa.

Por estas razones, Rust toma un enfoque diferente, usando *trait objects* en
lugar de herencia. Veamos cómo los *trait objects* permiten el polimorfismo
en Rust.
