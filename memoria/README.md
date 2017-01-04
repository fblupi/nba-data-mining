# Estadísticas NBA

>> Francisco Javier Bolívar Lupiáñez
>> José Manuel Moya Baena

## Introducción

El baloncesto es un deporte donde se pueden rescatar, más fácilmente que en otros deportes de equipo como el fútbol, las estadísticas individuales y relevantes de cada jugador pero el uso que se hacen de ellas es muy pobre pues en la mayoría de las webs solo acaban mostrando quiénes son los jugadores que más puntos, rebotes y asistencias por partido hacen.

Es por ello que vemos en este un buen campo para realizar minería de datos, por lo que el *dataset* escogido ha sido el de las estadísticas individuales de cada jugador durante la temporada 2012/2016.

### *Dataset*

El *dataset* escogido cuenta con los siguientes datos:

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

![img/preprocesamiento/statistics.png](Estadísticas)

Aquí ya podemos observar como datos como tiros intentados y tiros anotados (de campo, de tres y libres) tienen bastante parecido y como muchas columnas tienen una enorme asimetría probablemente provocada por la diferencia que hay entre los que juegan muchos minutos y los que juegan pocos ya que a más minutos se jueguen mayor será la cantidad de puntos, rebotes o asistencias realizadas. Esto nos puede llevar a empezar a pensar en que una columna puntos, asistencias, rebotes... por minuto sea mucho más significativa para realizar comparaciones. Siempre teniendo en cuenta que a más minutos juega un jugador es porque, supuestamente, es más bueno.

#### Diagrama de cajas

A continuación vamos a ver los diagramas de cajas de cada una de las columnas usando el nodo *box plot* de KNIME.

![img/preprocesamiento/boxplot-all.png](Diagrama de cajas)

Las sospechas de lo desequilibradas que podrían estar algunas de las columnas se ven despejadas con este diagrama donde vemos que la mayoría de los datos están muy desequilibrados ya que casi todos tienen su mínimo en cero (algún jugador que haya jugado muy pocos minutos y no ha podido sumar en su casillero personal de esa estadística) y muchos que despuntan por arriba (aquellos que juegan muchos minutos y además los rentan con buenos números).

#### Correlación linear

Algunos datos podrían ser eliminados o sustituidos por otros. Ya hemos visto que los minutos influyen mucho en el resto de estadísticas. Con la matriz de correlación lo vamos a ver más claramente:

![img/preprocesamiento/correlation-matrix-original-dataset.png](Matriz de correlación)

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

La matriz de correlación con los nuevos datos es la siguiente:

![img/preprocesamiento/correlation-matrix-new-dataset.png](Nueva matriz de correlación)

Donde se sigue viendo una relación fuerte entre minutos y partidos de inicio y entre porcentaje de tiros de tres y anotados (algo que no se ve tan claramente en los tiros libres).

#### Diagrama de cajas

Mediante *box plot* agrupando por posiciones vamos a ver si algún dato tiene relación con alguna posición en particular para tenerlo en cuenta posteriormente a la hora de realizar agrupaciones.

TODO

#### Diagrama de dispersión

Directamente con los datos del *dataset* se ha asignado un color a cada posición:

 * Base: Cyan
 * Escolta: Verde
 * Alero: Amarillo
 * Ala-pívot: Magenta
 * Pívot: Rojo

A continuación se ha realizado un *scatter plot* interactivo para poder comparar varias variables:

##### Relación Equipo y +/-

![img/preprocesamiento/scatter-plot-balance](Equipo +/-)

Con este gráfico se puede ver claramente quién fue el mejor equipo, ya que es el que acumula el mejor ratio puntos a favor/en contra. El mejor equipo es Golden State Warriors (GSW), con una diferencia considerable sobre Cleveland Cavalieres (CLE), San Antonio Spurs (SAN), Oklahoma-City Thunder (OKL) y Los Ángeles Clippers (LAC). Los peores serían Los Ángeles Lakers (LAL), seguido de cerca por Philadelphia 76ers (PHI) y con algo de diferencia sobre Phoenix Suns (PHO) y Brooklin Nets (BRO).
