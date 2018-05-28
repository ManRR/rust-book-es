# Comprender la propiedad (Ownership)

La propiedad (*Ownership*), es la característica más exclusiva de Rust, y permite a Rust hacer
garantías en la seguridad de la memoria sin necesidad de un recolector de basura.
Por lo tanto, es importante entender cómo funciona la propiedad en Rust. En este
capítulo, vamos a hablar sobre la propiedad, así como varias características relacionadas:
préstamos (*borrowing*), rebanadas (*slices*), y cómo Rust establece los datos en la memoria.
