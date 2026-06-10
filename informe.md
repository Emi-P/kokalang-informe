# Introducción

El paradigma de programación funcional promueve funciones puras, cuyo resultado depende únicamente de sus argumentos. Sin embargo, en la práctica las funciones pueden tener efectos secundarios como no terminar, lanzar excepciones o modificar estado.

Como alternativa, se plantea el enfoque de los *algebraic effects and handlers*, que representa los efectos como operaciones abstractas, como `get` y `set` para estado mutable o `raise` para excepciones.

Koka es un lenguaje funcional que destaca por explicitar los efectos en el tipo de las funciones y por el uso de *effect handlers* para dar semántica a su manejo.

En este escrito se introduce el sistema de tipos de Koka, en particular sus reglas de inferencia de efectos, y se analiza cómo esta información puede utilizarse para la implementación y composición de *effect handlers*.

# El sistema de tipos (Row-Polymorphic Effect Types)

Como sugiere el nombre, los efectos en Koka se representan mediante *rows*, es decir, colecciones extensibles de etiquetas que describen los efectos presentes en un cálculo. Estas etiquetas pueden combinarse para formar conjuntos de efectos compuestos.

El sistema es *row-polymorphic*, lo que significa que las funciones pueden ser polimórficas respecto a los efectos que contienen. Esto permite abstraer sobre los efectos concretos de un cálculo y facilita la composición modular de programas con distintos tipos de efectos.

# Tipos con efectos en Koka


<!-- ------------------------------------------------------------- -->
# Referencias

[1] Daan Leijen. *Koka: Programming with Row-Polymorphic Effect Types*. 2014. Disponible en: https://arxiv.org/pdf/1406.2061

[2] Daan Leijen. *Programming with Implicit Values, Functions, and Control*. Microsoft Research Technical Report, 2019. Disponible en: https://www.microsoft.com/en-us/research/wp-content/uploads/2019/03/implicits-tr-v2.pdf

[3] Andrej Bauer y Matija Pretnar. *An Introduction to Algebraic Effects and Handlers*. Disponible en: https://www.eff-lang.org/handlers-tutorial.pdf

[4] Gordon Plotkin y Matija Pretnar. *Handlers of Algebraic Effects*. Disponible en: https://homepages.inf.ed.ac.uk/gdp/publications/Effect_Handlers.pdf

[5] Koka Documentation. Disponible en: https://koka-lang.github.io/koka/doc/book.html

[6] Koka Community Documentation. Disponible en: https://koka-community.github.io/koka-docs/koka-docs.kk.html

[7] Repositorio oficial de Koka. Disponible en: https://github.com/koka-lang/koka

[8] Daan Leijen. *Koka Presentation*. Disponible en: https://www.youtube.com/watch?v=6OFhD_mHtKA
