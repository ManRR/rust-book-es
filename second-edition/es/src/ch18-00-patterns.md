# Patterns and Matching

Los patrones son una sintaxis especial en Rust para hacer coincidir con la
estructura de tipos, tanto complejos como simples. El uso de patrones junto
con las expresiones `match` y otras construcciones le da más control sobre el
flujo de control de un programa. Un patrón consiste en una combinación de los
siguientes:

* Literals
* Destructured arrays, enums, structs, or tuples
* Variables
* Wildcards
* Placeholders

Estos componentes describen la forma de los datos con los que estamos
trabajando, y luego los comparamos con los valores para determinar si nuestro
programa tiene los datos correctos para continuar ejecutando un fragmento de
código en particular.

Para usar un patrón, lo comparamos con algún valor. Si el patrón coincide con
el valor, usamos las partes de valor en nuestro código. Recuerde las
expresiones `match` en el Capítulo 6 que utilizan patrones, como el ejemplo
de la máquina clasificadora de monedas. Si el valor se ajusta a la forma del
patrón, podemos usar las piezas nombradas. Si no lo hace, el código asociado
con el patrón no se ejecutará.

Este capítulo es una referencia sobre todo lo relacionado con patrones.
Cubriremos los lugares válidos para usar patrones, la diferencia entre
patrones refutables e irrefutables y los diferentes tipos de sintaxis de
patrones que podría ver. Al final del capítulo, sabrá cómo usar patrones para
expresar muchos conceptos de una manera clara.