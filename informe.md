## Introducción

El paradigma de programación funcional promueve funciones puras, cuyo resultado depende únicamente de sus argumentos. Sin embargo, en la práctica las funciones pueden tener efectos secundarios como no terminar, lanzar excepciones o modificar estado.

Como alternativa, se plantea el enfoque de los *algebraic effects and handlers*, que representa los efectos como operaciones abstractas, como `get` y `set` para estado mutable o `raise` para excepciones.

Koka es un lenguaje funcional que destaca por explicitar los efectos en el tipo de las funciones y por el uso de *effect handlers* para dar semántica a su manejo.

En este escrito se introduce el sistema de tipos de Koka, en particular sus reglas de inferencia de efectos, y se analiza cómo esta información puede utilizarse para la implementación y composición de *effect handlers*.

## El sistema de tipos con efectos (*Row-polymorphic Effect Types*)

El sistema de tipos de tipos con efectos (*effect types*) de Koka, incorpora información sobre los efectos que una función puede producir durante su ejecución. Para ello, los efectos se representan mediante *effect rows*, es decir, colecciones extensibles de etiquetas que describen los efectos presentes en un cálculo.

Por ejemplo, $\langle\texttt{exn}\rangle$ representa el efecto de lanzar excepciones. Si $f$ es una función de $\texttt{Int}$ en $\texttt{Int}$ que puede lanzar excepción, entonces tendrá la signatura:

$$f: \texttt{int} \to \langle\texttt{exn}, \texttt{div}\rangle\ \texttt{int}$$

Si además puede divergir, su signatura sería:

$$f: \texttt{int} \to \langle\texttt{exn}, \texttt{div}\rangle\ \texttt{int}$$

El sistema es *row-polymorphic*, lo que significa que las funciones pueden ser polimórficas respecto a los efectos que contienen. La función `map` sobre listas de enteros en este lenguaje tiene tipo:

$$\texttt{map} : (\texttt{list<int>}, \texttt{int} \to \varepsilon \ \texttt{int}) \to \varepsilon \ \texttt{list<int>}$$

Donde esta signatura se lee cómo: `map` toma una lista de enteros, y una función $\texttt{Int} \to \texttt{Int}$ que puede tener un efecto secundario $\varepsilon$. La transformación que realiza `map` sobre esta lista, propaga el efecto secundario $\varepsilon$ y produce una lista de enteros.

Los tipos de efecto se representan mediante filas de efectos (*effect rows*), es decir, colecciones de etiquetas que describen los efectos que una expresión puede realizar. Una fila puede ser vacía ($\langle\rangle$), una variable de efecto polimórfica ($\mu$) o una extensión de otra fila mediante una nueva etiqueta ($\langle l \mid \varepsilon\rangle$). De esta manera, una fila cerrada tiene la forma $\langle l_1, \ldots, l_n\rangle$, mientras que una fila abierta tiene la forma $\langle l_1, \ldots, l_n \mid \mu\rangle$, indicando que además de los efectos conocidos puede contener otros efectos aún no determinados.

Un aspecto distintivo del sistema de tipos en Koka es la equivalencia de efectos. Esta resulta fundamental para comparar, unificar e inferir efectos sin depender del orden en el aparecen escritos. Algunas reglas de equivalencia se muestran en la Figura 1.

![Figura 1: Reglas de equivalencia de efectos]()

Por ejemplo, como resultado de aplicar las reglas, tenemos $\langle\texttt{exn}, \texttt{div}\rangle \equiv \langle\texttt{div}, \texttt{exn}\rangle$. Pues $\texttt{exn} \neq \texttt{div}$ y por regla (EQ-SWAP) $\langle\texttt{div} \mid \langle\texttt{exn} \mid \langle\rangle\rangle\rangle \equiv \langle\texttt{exn} \mid \langle\texttt{div} \mid \langle\rangle\rangle\rangle$, es decir, $\langle\texttt{div}, \texttt{exn}\rangle \equiv \langle\texttt{exn}, \texttt{div}\rangle$.

Nótese que no se tiene una regla de reducción de efectos del estilo $\langle l \mid \langle l \mid \varepsilon\rangle\rangle \equiv \langle l \mid \varepsilon\rangle$. Esto es importante porque significa que el sistema de tipos diferencia entre efectos duplicados, por ejemplo $\langle\texttt{exn}, \texttt{exn}\rangle \not\equiv \langle\texttt{exn}\rangle$. Si bien dificulta la intuición del usuario en el tipo de función, se utiliza debido a que permite [TO DO: cerrar la idea con el ejemplo de catch, pero más concreto].

## La inferencia de efectos

La inferencia de efectos en Koka se formaliza mediante un sistema de reglas de tipado, algunas de las cuales se presentan en la Figura 3. Estas reglas determinan qué efectos pueden atribuirse a una expresión a partir de su estructura y del entorno de tipos. Para describirlas, primero se introduce una sintaxis reducida de expresiones del lenguaje.

![Figura 3: Reglas de tipado con efectos]()

La sintaxis de este pequeño cálculo lambda es suficiente para introducir algunas reglas interesantes de la inferencia de tipos con efectos. Las reglas de tipos se formulan respecto de un entorno de tipos (en general denotados $\Gamma$), que asocia variables con tipos. La ecuación:

$$\Gamma(z) = \sigma$$

Nos dice que $\Gamma$ le asocia el tipo $\sigma$ a la variable $z$.

Por ejemplo:

- $\Gamma(z) = \texttt{Int}$
- $\Gamma(y) = \texttt{Bool}$

Estos entornos se pueden extender mediante coma y dos puntos, esto es: si $\Gamma$ es cualquier entorno podemos cambiar un punto:

Si $\Gamma' = \Gamma, x : \sigma \implies \Gamma'(x) = \sigma$ (análogo a: $\Gamma' = [\Gamma \mid x : \sigma]$).

Esto viene de que Koka es un lenguaje tipado. Utilizaremos estos entornos para definir las reglas de tipos sobre cualquier expresión del cálculo que presentamos sintácticamente antes.

Una regla de tipos de la forma $\Gamma \vdash e : \sigma \mid \varepsilon$ impone que bajo el entorno $\Gamma$ la expresión $e$ tiene el tipo $\sigma$ con un efecto $\varepsilon$. Con esto se pueden analizar algunas reglas:

Analizamos la regla (LAM) con un ejemplo. Tiparemos $\lambda x.\,x$. Asumiendo $\Gamma, x:\texttt{Int}$, por la regla de variables tenemos:

$$\Gamma, x:\texttt{Int} \vdash x : \texttt{Int} \mid \langle\rangle$$

Sustituyendo en la regla tenemos:

$$
\frac{\Gamma, x:\texttt{Int} \vdash x : \texttt{Int} \mid \langle\rangle}{\Gamma \vdash \lambda x.\,x : \texttt{Int} \to \langle\rangle\ \texttt{Int} \mid \langle\rangle}
$$

O sea que en el entorno que tipa $x$ como un entero, la identidad $\lambda x.\,x$ es tipada como $\lambda x.\,x : \texttt{Int} \to \langle\rangle\ \texttt{Int}$, o sea una función de enteros en enteros, sin efectos.

Tipemos $\lambda x.\,\texttt{throw}()$, teniendo como premisa $\Gamma, x:\texttt{Int} \vdash \texttt{throw}() : \texttt{Int} \mid \langle\texttt{exn}\rangle$. Aplicando la regla tenemos que:

$$
\frac{\Gamma, x : \texttt{Int} \vdash \texttt{throw}() : \texttt{Int} \mid \langle\texttt{exn}\rangle}{\Gamma \vdash \lambda x.\,\texttt{throw}() : \texttt{Int} \to \langle\texttt{exn}\rangle\ \texttt{Int} \mid \langle\rangle}
$$

Hay que notar que en sí misma, el objeto $\lambda x.\,\texttt{throw}()$ no tiene un efecto secundario, pues se infiere $\langle\rangle$. Pero es una función del tipo $\texttt{Int} \to \langle\texttt{exn}\rangle\ \texttt{Int}$. Aunque parezca confuso nos dice muy elegantemente que $\lambda x.\,\texttt{throw}()$ podría existir como subexpresión pero que, si no es aplicada, no necesariamente se infiere el efecto $\langle\texttt{exn}\rangle$. Por ejemplo:

$$(\lambda z.\,(\lambda x.\,\texttt{throw}())$$

Claramente es la función que va a devolver constantemente $\lambda x.\,\texttt{throw}()$ sin aplicarla. Por lo que es razonable no inferir que tenga la posibilidad de lanzar una excepción.
