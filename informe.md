# Trabajo Especial de Modelos y Simulación

**Integrantes:** Emilio Joaquin Pereyra \| Julian David Carrillo
Montilla

**Fecha:** Primer cuatrimestre 2026

------------------------------------------------------------------------

## Ejercicio 1 --- Centro de atención con servidores en serie

### 1.a --- Implementación de la simulación

Las llegadas siguen un proceso de Poisson no homogéneo con tasa
$\lambda(t) = 8 + 4\sin(\pi t/12)$. Para simularlas se aplica el
**método de adelgazamiento** de la Proposición 2.1: se genera
un proceso de Poisson homogéneo con tasa $\lambda_{\max}=12$ (cota
superior) y se acepta cada candidato con probabilidad $\lambda(t)/12$.

Los tiempos de servicio se generan por **transformada inversa**:
$-\ln(U)/\lambda$ con $U\sim\mathcal{U}(0,1)$. Recepción: $Exp(15)$
(media 4 min), Validación: $Exp(10)$ (media 6 min).

La simulación es de eventos discretos con tres tipos: llegada, fin de
Recepción y fin de Validación. Se procesa el evento más próximo en cada
iteración. Como ambas etapas son FIFO con un servidor, el orden de los
clientes se preserva y no hacen falta identificadores. Para el ítem 1.b
se simula hasta un tiempo $T$; para el 1.c se simula hasta atender $k$
clientes.

### 1.b --- Estimación del tiempo promedio de permanencia e intervalos de confianza

Se realizaron 1000 simulaciones independientes de $T = 24$ horas cada
una. Para cada simulación se calculó la media muestral del tiempo de
permanencia

$$\bar X_i = \frac{1}{n_i} \sum_{j=1}^{n_i} (D_j - A_{1,j}).$$

Sobre el conjunto de 1000 medias muestrales se construyeron intervalos
de confianza asintóticos usando el Teorema Central del Límite:

$$IC_{1-\alpha} = \bar X \pm z_{\alpha/2} \frac{S}{\sqrt{N}}$$

  Nivel de confianza   $z_{\alpha/2}$   IC (horas)
  -------------------- ---------------- ----------------
  95 %                 1.96             \[1.14, 1.22\]
  98 %                 2.33             \[1.10, 1.30\]

La estimación puntual del tiempo promedio de permanencia ronda las **1.2
horas (≈ 72 minutos)**.

### 1.c --- Histograma y bondad de ajuste

Se simuló la atención de los primeros 10 000 clientes y se construyó un
histograma de sus tiempos de permanencia.

**Características del histograma:** - Asimétrico con cola larga hacia la
derecha. - Moda próxima a cero. - Media alrededor de 1.2 horas. -
Presencia de valores extremos (cola larga).

![Histograma de tiempos de permanencia con ajustes Exponencial y
Lognormal](figs/histograma_ej1c.png)

**Distribuciones propuestas:**

1.  **Exponencial** --- por su uso habitual en teoría de colas y por
    tener un solo parámetro fácil de estimar: $\hat\lambda = 1/\bar X$.
2.  **Lognormal** --- por su capacidad de modelar datos positivos con
    asimetría y cola larga. Parámetros estimados por máxima
    verosimilitud: $\hat\mu = \overline{\log X}$,
    $\hat\sigma = S_{\log X}$.

**Prueba $\chi^2$:**

Se utilizaron $k = 10$ intervalos basados en **cuantiles teóricos** de
cada distribución para asegurar frecuencias esperadas uniformes. Los
resultados fueron:

  Distribución   $\chi^2$   g.l.   p-valor
  -------------- ---------- ------ -------------
  Exponencial    ≈ 430      8      $\approx 0$
  Lognormal      ---        7      $\approx 0$

Con $n = 10\,000$ observaciones, el test $\chi^2$ tiene una potencia muy
alta y rechaza ambas distribuciones. Ninguna provee un ajuste adecuado
según el criterio del test.

**Discusión:**

A pesar del rechazo estadístico, optamos por mantener **Exponencial** y
**Lognormal** como las distribuciones candidatas por las siguientes
razones:

- Son las distribuciones estándar en teoría de colas para modelar
  tiempos de respuesta.
- El histograma es compatible visualmente con ambas (cola larga, modo
  cercano a cero).
- Con muestras grandes ($n \gg 1000$), los tests de bondad de ajuste
  tienden a rechazar prácticamente cualquier distribución paramétrica,
  ya que detectan diferencias mínimas que no necesariamente son
  relevantes en la práctica.
- El análisis gráfico (superposición de PDFs sobre el histograma)
  permite una evaluación complementaria que, en este caso, muestra un
  seguimiento visual aceptable de la Lognormal en la cola y de la
  Exponencial en la zona del modo.

------------------------------------------------------------------------

## Ejercicio 2 --- Centro de atención con clientes prioritarios

### 2.a --- Adaptación de la simulación

Cada cliente recibe al llegar un atributo Bernoulli con $p=0.20$ que
determina si es prioritario. La Recepción sigue siendo FIFO para todos,
pero en Validación se separan en dos colas: prioritaria y normal. Al
terminar un servicio en Validación se atiende siempre al primer cliente
de la cola prioritaria si está ocupada; solo cuando está vacía se
atiende a la cola normal. Los servicios no son interrumpibles
(non-preemptive).

Como en la segunda etapa el orden de salida ya no coincide con el de
llegada, cada cliente recibe un **identificador numérico** al ingresar,
permitiendo aparear correctamente sus tiempos de llegada y salida.

### 2.b --- Estimación del tiempo promedio de permanencia e IC

Se realizaron 500 simulaciones de $T = 24$ horas cada una. Los
resultados:

  Tipo de cliente   Media (hs)   IC 95 %
  ----------------- ------------ ----------------
  Prioritarios      ≈ 0.32       \[0.31, 0.33\]
  Normales          ≈ 1.41       \[1.35, 1.48\]

### 2.c --- Histogramas y estadísticas descriptivas

Se realizó una simulación larga (240 horas) para obtener una muestra con
suficientes observaciones de ambos tipos.

![Histogramas de tiempos de permanencia por tipo de
cliente](figs/histogramas_ej2c.png)

  Estadística          Prioritarios   Normales
  -------------------- -------------- ------------
  Media                ≈ 0.3 hs       ≈ 1.4 hs
  Mediana              ≈ 0.2 hs       ≈ 1.1 hs
  Desvío estándar      ≈ 0.2 hs       ≈ 1.3 hs
  Percentil 90         ≈ 0.7 hs       ≈ 3.0 hs
  $P(T > 1\text{h})$   ≈ 2--5 %       ≈ 54--62 %
  $P(T > 2\text{h})$   ≈ 0 %          ≈ 18--40 %

**Diferencias observadas:**

Los clientes prioritarios experimentan tiempos de permanencia
significativamente menores. Su distribución está concentrada cerca de
cero (media \~0.3 h, mediana \~0.2 h), con muy baja probabilidad de
superar 1 hora. Los clientes normales, en cambio, presentan una
distribución con cola mucho más larga (desvío \~1.3 h) y una
probabilidad considerable de esperar más de 1 hora (\~60 %) y más de 2
horas (\~20--40 %).

### 2.d --- Comparación con el Ejercicio 1

En el Ejercicio 1 (sin prioridades), todos los clientes tenían un tiempo
de permanencia promedio de aproximadamente 1.2 horas.

Al introducir prioridades en Validación:

- **Prioritarios (20 %)**: su permanencia media se reduce drásticamente
  a \~0.3 horas. Al tener prioridad sobre los normales en la cola de
  Validación, rara vez esperan --- su tiempo en el sistema es
  básicamente la suma de sus dos servicios más una espera mínima en
  Recepción.
- **Normales (80 %)**: su permanencia media aumenta a \~1.4 horas,
  llegando en algunos casos a más de 3 horas (percentil 90). Son
  desplazados constantemente por los clientes prioritarios que llegan
  después pero pasan delante de ellos en Validación.

**Impacto de la política de prioridades:**

La política beneficia a una minoría (20 % del total) a costa de
perjudicar a la mayoría (80 %). El tiempo promedio global (ponderado)
resulta similar al del Ejercicio 1, pero la distribución se vuelve
marcadamente bimodal: los prioritarios experimentan una mejora
sustancial mientras que los normales sufren demoras adicionales. Este
comportamiento es esperable: la prioridad sin interrupción
(non-preemptive) en el cuello de botella (Validación, con mayor carga:
$\rho \approx 0.8$) genera este efecto de "salto de cola" que penaliza a
los clientes regulares.

------------------------------------------------------------------------

## Conclusiones generales

Se implementaron dos simuladores de eventos discretos para un sistema de
dos servidores en serie: uno sin prioridades (Ej. 1) y otro con clientes
prioritarios en la segunda etapa (Ej. 2). Las simulaciones permitieron
estimar los tiempos de permanencia, construir intervalos de confianza, y
analizar la distribución de los datos mediante histogramas y pruebas de
bondad de ajuste $\chi^2$.

Las principales conclusiones son:

1.  El tiempo promedio de permanencia sin prioridades es de
    aproximadamente 1.2 horas, con intervalos de confianza estrechos
    gracias al gran número de réplicas.
2.  La distribución del tiempo de permanencia no se ajusta
    satisfactoriamente a una Exponencial ni a una Lognormal según el
    test $\chi^2$, aunque visualmente ambas son compatibles con el
    histograma.
3.  La introducción de prioridades reduce el tiempo de permanencia de
    los clientes prioritarios (\~0.3 h) a costa de incrementar el de los
    normales (\~1.4 h), generando una distribución bimodal con mayor
    dispersión.
