# Escribir pruebas automatizadas

En su ensayo de 1972 “The Humble Programmer”, Edsger W. Dijkstra dijo que
“las pruebas del programa pueden ser una forma muy efectiva de mostrar la
presencia de errores, pero es irremediablemente inadecuado para mostrar su
ausencia”. ¡Eso no significa que no deberíamos intentar probar tanto como
podamos!

La corrección en nuestros programas es la medida en que nuestro código hace
lo que pensamos que haga. Rust está diseñado con un alto grado de
preocupación sobre la corrección de los programas, pero la corrección es
compleja y no es fácil de probar. El sistema de tipos de Rust soporta una
gran parte de esta carga, pero el sistema de tipos no puede detectar todo
tipo de incorrecciones. Como tal, Rust incluye soporte para escribir pruebas
automatizadas de software dentro del lenguaje.

Como ejemplo, digamos que escribimos una función llamada `add_two` que agrega
2 al número que se le pasa. La firma de esta función acepta un entero como
parámetro y devuelve un entero como resultado. Cuando implementamos y
compilamos esa función, Rust hace todo el chequeo de tipos y verificación de
préstamos que has aprendido hasta ahora para asegurarte de que, por ejemplo,
no estamos pasando un valor `String` o una referencia inválida a esta
función. Pero Rust *no puede* verificar que esta función hará exactamente lo
que pretendemos, que es devolver el parámetro más 2 en lugar de, digamos, el
parámetro más 10 o el parámetro menos 50. Ahí es donde entran las pruebas.

Podemos escribir pruebas que afirmen, por ejemplo, que cuando pasamos `3` a
la función `add_two`, el valor devuelto es `5`. Podemos ejecutar estas
pruebas cada vez que hagamos cambios a nuestro código para asegurarnos de que
ningún comportamiento correcto existente no haya cambiado.

Las pruebas son una habilidad compleja: aunque no podemos cubrir todos los
detalles sobre cómo escribir buenas pruebas en un capítulo, estudiaremos la
mecánica de las instalaciones de prueba de Rust. Hablaremos sobre las
anotaciones y macros disponibles para usted cuando escriba sus pruebas, el
comportamiento predeterminado y las opciones proporcionadas para ejecutar sus
pruebas, y cómo organizar las pruebas en pruebas unitarias y pruebas de
integración.
