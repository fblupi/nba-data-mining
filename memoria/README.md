# Estadísticas NBA

> Francisco Javier Bolívar Lupiáñez

> José Manuel Moya Baena

## Introducción

El baloncesto es un deporte donde se pueden rescatar, más fácilmente que en otros deportes de equipo como el fútbol, las estadísticas individuales y relevantes de cada jugador pero el uso que se hacen de ellas es muy pobre pues en la mayoría de las webs solo acaban mostrando quiénes son los jugadores que más puntos, rebotes y asistencias por partido hacen.

Es por ello que vemos en este un buen campo para realizar minería de datos, por lo que el *dataset* escogido ha sido el de las estadísticas individuales de cada jugador durante la temporada 2012/2016.

### *Dataset*

El *dataset* escogido contiene las estadísticas de los 476 jugadores que jugaron durante la temporada 2015/2016 algún partido en la NBA. Las estadísticas cuentan con los siguientes datos:

* *Player*: Nombre del jugador
* *Team*: Iniciales del equipo donde juega
* *PS*: Posición en la que juega. Puede ser:
  * *PG*: Base (*point guard*)
  * *SG*: Escolta (*small guard*)
  * *SF*: Alero (*small forward*)
  * *PF*: Ala-pívot (*power forward*)
  * *C*: Pívot (*center*)
* *GP*: Partidos jugados
* *Min*: Minutos jugados
* *FGM*: Tiros de campo anotados
* *FGA*: Tiros de campo intentados
* *3M*: Tiros de tres anotados
* *3A*: Tiros de tres intentados
* *FTM*: Tiros libres anotados
* *FTA*: Tiros libres intentados
* *OR*: Rebotes ofensivos
* *TR*: Rebotes
* *AS*: Asistencias
* *ST*: Robos
* *TO*: Pérdidas
* *BK*: Bloqueos
* *PF*: Faltas personales
* *DQ*: Descalificaciones
* *PTS*: Puntos
* *TC*: Faltas técnicas
* *Sta*: Partidos comenzados desde el quinteto inicial
* *+/-*: Balance de puntos a favor o en contra del equipo mientras el jugador está en el campo

Todos los datos son números enteros a excepción del nombre del jugador, el equipo y la posición que son cadenas de caracteres.

## Minería de datos

Hemos decidido realizar el trabajo utilizando la herramienta KNIME debido a que es sencilla de utilizar y ofrece una retroalimentación visual que nos permite ver el espacio de trabajo de forma directa.

### Pre-procesamiento de datos

La primera fase será la de realizar un pre-procesado de los datos mediante técnicas de estadística descriptiva con el objetivo de conocer nuestro *dataset* para poder realizar posteriormente otras tareas.

#### Datos estadísticos

Usando el nodo *statistics* de KNIME directamente sobre nuestro *dataset* podemos realizar un primer vistazo a nuestros datos.

Con este podemos ver para cada columna el valor mínimo, máximo, media, desviación típica, asimetría estadística, curtosis, datos perdidos, datos con valor infinito y un histograma con las repeticiones de cada valor.

!["Estadísticas"](img/preprocesamiento/statistics.png)

Aquí ya podemos observar como datos como tiros intentados y tiros anotados (de campo, de tres y libres) tienen bastante parecido y como muchas columnas tienen una enorme asimetría probablemente provocada por la diferencia que hay entre los que juegan muchos minutos y los que juegan pocos ya que a más minutos se jueguen mayor será la cantidad de puntos, rebotes o asistencias realizadas. Esto nos puede llevar a empezar a pensar en que una columna puntos, asistencias, rebotes... por minuto sea mucho más significativa para realizar comparaciones. Siempre teniendo en cuenta que a más minutos juega un jugador es porque, supuestamente, es más bueno.

#### Diagrama de cajas

A continuación vamos a ver los diagramas de cajas de cada una de las columnas usando el nodo *box plot* de KNIME.

!["Diagrama de cajas"](img/preprocesamiento/boxplot.png)

Las sospechas de lo desequilibradas que podrían estar algunas de las columnas se ven despejadas con este diagrama donde vemos que la mayoría de los datos están muy desequilibrados ya que casi todos tienen su mínimo en cero (algún jugador que haya jugado muy pocos minutos y no ha podido sumar en su casillero personal de esa estadística) y muchos que despuntan por arriba (aquellos que juegan muchos minutos y además los rentan con buenos números).

#### Correlación linear

Algunos datos podrían ser eliminados o sustituidos por otros. Ya hemos visto que los minutos influyen mucho en el resto de estadísticas. Con la matriz de correlación lo vamos a ver más claramente:

!["Matriz de correlación"](img/preprocesamiento/correlation-matrix-original-dataset.png)

Efectivamente vemos como la columna de los puntos se ve tremendamente influenciada con los minutos alcanzando una correlación cercana a uno y a su vez influye en puntos de campo intentados y anotados.

También hay una relación enorme entre tiros intentados y anotados (tanto de campo, como de tres, como libres), por lo que podría ser una buena idea cambiar la columna de tiros intentados por porcentaje de acierto que nos puede dar más información.

Y no solo los puntos se ven influenciados por los minutos. Otro dato que, lógicamente, se ve afectado es el de partidos jugados, a más partidos juegas más minutos y viceversa y como los minutos es un datos más preciso que los partidos se podría prescindir de este dato ya que también tenemos el número de partidos que se empiezan desde el quinteto con el que podemos obtener los jugadores más buenos (que suelen salir de inicio).

**CONCLUSIÓN:** Vamos a modificar las siguientes columnas del *dataset*:

* Partidos jugados será borrado por su enorme relación con los minutos.
* Tiros de campo anotados e intentados serán borrados por su enorme relación con los puntos.
* Tiros de tres anotados por tiros de tres anotados por minuto.
* Tiros de tres intentados por porcentaje de acierto en tiros de tres.
* Tiros libres anotados por tiros libres por minuto.
* Tiros libres intentados por porcentaje de acierto en tiros libres.
* Rebotes ofensivos será borrado por su enorme relación con los rebotes.
* Rebotes por rebotes por minuto.
* Asistencias por asistencias por minuto.
* Robos por robos por minuto.
* Pérdidas por pérdidas por minuto.
* Bloqueos por bloqueos por minuto.
* Faltas personales por faltas personales por minuto.
* Puntos por puntos por minuto.
* Partidos de inicio será borrado por su enorme relación con los minutos.

También eliminaré aquellas filas con jugadores que hayan jugado menos de 100 minutos ya que pueden haber jugado tan pocos que todos los tiros que hayan hecho los hayan metido y salgan como si hubiesen hecho mucho aunque en realidad apenas han aportado porque han jugado poco. Con este filtro se ha pasado de las 476 filas iniciales a 431.

La matriz de correlación con los nuevos datos es la siguiente:

!["Nueva matriz de correlación"](img/preprocesamiento/correlation-matrix-new-dataset.png)

Donde ya no se ven relaciones tan fuertes entre variables.

#### Diagrama de cajas

Mediante *box plot* agrupando por posiciones vamos a ver si algún dato tiene relación con alguna posición en particular para tenerlo en cuenta posteriormente a la hora de realizar agrupaciones.

##### Tiros de 3 anotados

!["Box Splot 3M"](img/preprocesamiento/boxplot-3m.png)

Aquí se puede observar como los pívot apenas anotan tiros de tres salvo casos aislados. Los ala-pívot tampoco suelen anotar mucho. Mientras que los bases, escoltas y aleros suelen anotar más o menos por igual.

##### Porcentaje tiros de 3

!["Box Splot 3A"](img/preprocesamiento/boxplot-3a.png)

En cuanto al porcentaje, muchos pivots y ala-pívots ni siquiera tiran y los que tiran lo hacen con un porcentaje muy bajo salvo casos aislados. Al igual que con los anotados, las otras tres posiciones siguen más o menos el mismo comportamiento.

##### Tiros libres anotados

!["Box Splot FTM"](img/preprocesamiento/boxplot-ftm.png)

En este dato el comportamiento por posición no varía mucho. Si anotan tiros libres es porque les hacen faltas, lo cual indica que no se cometen faltas sobre jugadores de un puesto en particular.

##### Porcentaje tiros libres

!["Box Splot FTA"](img/preprocesamiento/boxplot-fta.png)

En cuanto al porcentaje de aciertos desde la línea de personal, los pívots son los peores tiradores y los bases los mejores, pero no difieren tanto los unos de los otros.

##### Rebotes

!["Box Splot TR"](img/preprocesamiento/boxplot-tr.png)

Este dato si es muy determinante, se puede observar claramente como un jugador que atrape muchos rebotes va a ser muy posiblemente pívot o ala-pívot y muy poco probable será que sea base o escolta.

##### Asistencias

!["Box Splot AS"](img/preprocesamiento/boxplot-as.png)

En cuanto a las asistencias el comportamiento es casi el opuesto. Los bases son los que más asistencias reparten on una diferencia considerable sobre el resto. Hay algunos casos de escoltas y aleros que reparten tanto como ellos, pero son casos aislados.

##### Robos

!["Box Splot ST"](img/preprocesamiento/boxplot-st.png)

En el apartado de robos se vuelve a ver más equilibrio pero con dominio de los bases seguidos de aleros y escoltas.

##### Pérdidas

!["Box Splot TO"](img/preprocesamiento/boxplot-to.png)

En las pérdidas también hay un ligero dominio de los bases. El resto de posiciones apenas difieren.

##### Bloqueos

!["Box Splot BK"](img/preprocesamiento/boxplot-bk.png)

En los bloqueos el comportamiento es similar al de los rebotes con dominio de las posiciones que juegan en la pintura (pívots y ala pívots).

##### Faltas personales

!["Box Splot PF"](img/preprocesamiento/boxplot-pf.png)

Las faltas personales también parecen ser cometidas más por los hombres grandes (pívots y ala pívots) pero no con una diferencia considerable.

##### Descalificaciones

!["Box Splot DQ"](img/preprocesamiento/boxplot-dq.png)

Al haber pocas descalificaciones a lo largo de la temporada no es un dato muy relevante a la hora de caracterizar una posición pero se ve como los pívot son los que más aportan en este dato.

##### Puntos

!["Box Splot PTS"](img/preprocesamiento/boxplot-pts.png)

En los puntos la diferencia tampoco es significativa entre posiciones comportándose casi por igual

##### Faltas técnicas

!["Box Splot TC"](img/preprocesamiento/boxplot-tc.png)

Con las faltas técnicas pasa algo parecido a lo que pasa con las descalificaciones, al haber tan pocas no va a influir mucho a la hora de caracterizar una posición y más viendo como hay un reparto equilibrado con los pocos datos que hay.

**RESUMEN INFLUENCIAS:**

* Base:
  * Muy positivas: Asistencias, robos
  * Positivas: Tiros libres anotados, porcentaje tiros libres, pérdidas
  * Negativas: -
  * Muy negativas: Rebotes, bloqueos
* Escolta:
  * Muy positivas: -
  * Positivas: Porcentaje tiros de tres
  * Negativas: Rebotes
  * Muy negativas: -
* Alero:
  * Muy positivas: -
  * Positivas: -
  * Negativas: -
  * Muy negativas: -
* Ala pívot:
  * Muy positivas: Rebotes, bloqueos
  * Positivas: Faltas personales
  * Negativas: Tiros de 3 anotados, porcentaje tiros de 3
  * Muy negativas: -
* Pívot:
  * Muy positivas: Rebotes, bloqueos
  * Positivas: Faltas personales, descalificaciones
  * Negativas: Porcentaje tiros de 3, porcentaje tiros libres
  * Muy negativas: Tiros de tres anotados

No se ha encontrado ningún atributo determinante para el alero y es que es una posición muy polivalente que aporta algo en todos los aspectos del juego.

#### Diagrama de dispersión

Directamente con los datos del *dataset* se ha asignado un color a cada posición:

 * Base: Cyan
 * Escolta: Verde
 * Alero: Amarillo
 * Ala-pívot: Magenta
 * Pívot: Rojo

A continuación se ha realizado un *scatter plot* interactivo para poder comparar varias variables:

##### Relación Equipo y +/-

!["Equipo +/-"](img/preprocesamiento/scatter-plot-balance.png)

Con este gráfico se puede ver claramente quién fue el mejor equipo, ya que es el que acumula el mejor ratio puntos a favor/en contra. El mejor equipo es Golden State Warriors (GSW), con una diferencia considerable sobre Cleveland Cavalieres (CLE), San Antonio Spurs (SAN), Oklahoma-City Thunder (OKL) y Los Ángeles Clippers (LAC). Los peores serían Los Ángeles Lakers (LAL), seguido de cerca por Philadelphia 76ers (PHI) y con algo de diferencia sobre Phoenix Suns (PHO) y Brooklin Nets (BRO).

##### Asistencias y robos

!["Asistencias robos"](img/preprocesamiento/scatter-plot-as-st.png)

Previamente vimos que las asistencias y los robos eran las características positivas más determinantes de los bases y vemos en este gráfico de dispersión como efectivamente se cumple.

##### Rebotes y bloqueos

!["Rebotes bloqueos"](img/preprocesamiento/scatter-plot-tr-bk.png)

Al igual que las asistencias y los robos eran las características positivas más influyentes para los bases, para los pívots y ala pívots lo eran los rebotes y bloqueos y vemos que incluso se diferencia mejor el grupo que con asistencias y robos.

### Agrupamiento

Como hemos visto, hay cinco posiciones en el baloncesto, pero algunas de ellas se intercambian durante el partido y un mismo jugador puede jugar en varias posiciones durante una temporada por lo que dividir en tres posiciones puede resultar una tarea muy complicada. Por tanto, vamos a intentar agrupar en tres (las mismas tres en las que a veces se agrupan en algunas webs en lugar de las tres tradicionales):

* *Guard*: Juegan por fuera. Suelen ser los más bajos y agrupa a bases y escoltas aunque algunos aleros a veces ejercen esta función.
* *Center*: Juegan por dentro. Suelen ser los más altos y agrupa a pívots y ala pívots aunque algunos aleros a veces ejercen esta función.
* *Forward*: Realizan tareas mixtas, juegan tanto por fuera como por dentro por lo que suelen ser muy polivalentes. Agrupan a jugadores de cualquier posición, pero principalmente aleros y algunos escoltas y ala pívots.

Por tanto, esperamos que al realizar un agrupamiento no haya ningún base en el grupo de *center* ni un pívot en el grupo de *guard*.

#### K-medias

El primer método de agrupamiento que vamos a utilizar es el método de las k-medias usando el nodo *k-Means* de KNIME, usando el *dataset* modificado con estadísticas por minutos y utilizando las variables que vimos que eran más determinantes para definir una posición: tiros de tres anotados y su porcentaje de acierto, rebotes, asistencias, robos, pérdidas, bloqueos, faltas personales y descalificaciones.

!["Puntuación k-medias"](img/agrupamiento/entropy-scorer-k-means.png)

Los resultados del agrupamiento han dado una calidad de 0.331 que podría parecer baja, pero al visualizar la correspondencia posición por grupo generado nos podemos dar cuenta que los grupos generados parecen corresponderse bastante con lo predicho anteriormente. Entre los *center* están casi todos son pívots y ala pívots y la mayoría de los bases están en el grupo de los *guard*. El grupo de *forward* contiene casi todos los aleros y escoltas y algunos bases, pívots y ala pívots.

!["Grupo-posición k-medias"](img/agrupamiento/scatter-plot-k-means.png)

#### K-medioides

Se ha probado siguiendo el mismo enfoque que el mencionado con las k-medias el método de los k-medioides usando el nodo de KNIME *k-Medoids* usando la distancia Manhattan. El resultado ha sido similar obteniendo un ajuste de 0.306.

!["Grupo-posición k-medias"](img/agrupamiento/scatter-plot-k-medoids.png)

#### K-medias difusas

Siguiendo con los métodos de agrupamiento no supervisados particionales se ha probado el de las k-medias difusas usando el nodo de KNIME *Fuzzy c-Means* obteniendo un resultado similar al de las k-medias: 0.329.

!["Grupo-posición k-medias"](img/agrupamiento/scatter-plot-fuzzy-k-means.png)

### Clasificación

A la hora de clasificar se va a intentar predecir la posición de un jugador a partir de sus datos estadísticos. Ya hemos estado discutiendo lo complicado que puede resultar por la polivalencia de algunos jugadores y por cómo algunas posiciones comparten casi los mismos atributos (sobre todo los pívot y ala-pívots).

#### Árbol de decisión

El primer enfoque que se va a realizar es utilizando un árbol de decisión usando los nodos de KNIME *Decision Tree Learner* y *Decision Tree Predictor* usando como conjunto de entrenamiento 331 valores y 100 de test. El resultado ha sido más o menos el esperado con una precisión del 58%.

!["Clasificación árbol de decisión"](img/clasificacion/decision-tree-predictor-confusion-matrix.png)

Los errores que nos encontramos son en la mayoría de las ocasiones comprensibles por su parecido con la posición equivocada. En caso de los pívot las únicas confusiones han sido con ala-pívots y en el caso de los bases casi todas las confusiones ha sido con escoltas. La posición peor clasificada ha sido la de alero ya que, como hemos indicado reiteradamente, es muy complicada de predecir por ser jugadores muy polivalentes con cualidades y estadísticas individuales muy dispares los unos de los otros.

#### *Naive-Bayes*

Tras probar el árbol de decisión pasamos a probar cómo funciona el método de *Naive-Bayes* usando los nodos de KNIME *Naive Bayes Learner* y *Naive Bayes Predictor* y haciendo el mismo particionamiento obteniendo un resultado algo mejor de un 61%.

!["Clasificación Naive-Bayes"](img/clasificacion/naive-bayes-predictor-confusion-matrix.png)
