# Introducción

El paradigma de programación funcional promueve funciones puras, cuyo resultado depende únicamente de sus argumentos. Sin embargo, en la práctica las funciones pueden tener efectos secundarios como no terminar, lanzar excepciones o modificar estado.

Como alternativa, se plantea el enfoque de los *algebraic effects and handlers*, que representa los efectos como operaciones abstractas, como `get` y `set` para estado mutable o `raise` para excepciones.

Koka es un lenguaje funcional que destaca por explicitar los efectos en el tipo de las funciones y por el uso de *effect handlers* para dar semántica a su manejo.

En este escrito se introduce el sistema de tipos de Koka, en particular sus reglas de inferencia de efectos, y se analiza cómo esta información puede utilizarse para la implementación y composición de *effect handlers*.

# El sistema de tipos (Row-Polymorphic Effect Types)

Como sugiere el nombre, los efectos en Koka se representan mediante *rows*, es decir, colecciones extensibles de etiquetas que describen los efectos presentes en un cálculo. Estas etiquetas pueden combinarse para formar conjuntos de efectos compuestos.

El sistema es *row-polymorphic*, lo que significa que las funciones pueden ser polimórficas respecto a los efectos que contienen. Esto permite abstraer sobre los efectos concretos de un cálculo y facilita la composición modular de programas con distintos tipos de efectos.

# Tipos con efectos en Koka

En Koka, el tipo de una función no solo describe el tipo de sus argumentos y su resultado, sino también los efectos que puede realizar durante su ejecución. Un tipo de función tiene la forma general

$$
A \to B ; ! ; \varepsilon
$$

donde $A$ es el tipo de entrada, $B$ el tipo de salida y $\varepsilon$ denota el *row* de efectos asociados al cálculo.

Por ejemplo, una función que puede lanzar excepciones puede tener un tipo como

$$
\texttt{Int} \to \texttt{Int} ; ! ;\\langle \texttt{exn} \rangle
$$

mientras que una función pura no tiene efectos asociados:

$$
\texttt{Int} \to \texttt{Int} ; ! ; \langle \rangle
$$

El sistema de tipos de Koka propaga estos efectos de forma composicional: si una función llama a otra que produce efectos, dichos efectos se reflejan automáticamente en el tipo de la función llamadora. Esto permite razonar estáticamente sobre el comportamiento de un programa, sin necesidad de ejecución.

Además, gracias al *row polymorphism*, los efectos pueden permanecer parcialmente abstractos. Esto permite escribir funciones genéricas que no fijan de antemano los efectos exactos que utilizan, facilitando la reutilización y la composición de código con distintos contextos de ejecución.

# Ejemplos de efectos

Las siguientes funciones ilustran algunos de los efectos que Koka registra en los tipos.

```koka
fun square1( x : int ) : total int {
    x * x
}

fun square3( x : int ) : div int {
    x * square3( x )
}

fun square4( x : int ) : exn int {
    throw("oops");
    x * x
}
```

Los efectos forman parte del tipo de las funciones. En el ejemplo anterior, `square1` posee el efecto `total`, que identifica a los cálculos puros y terminantes.

Por otro lado, `square3` posee el efecto `div`, indicando que la computación puede divergir debido a la llamada recursiva sin una condición de corte.

Finalmente, `square4` posee el efecto `exn`, indicando que la evaluación puede finalizar lanzando una excepción mediante la operación `throw`.

De esta manera, el sistema de tipos permite describir estáticamente ciertos aspectos observables del comportamiento de los programas.
