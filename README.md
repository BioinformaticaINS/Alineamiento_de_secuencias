# Alineamiento de secuencias

## Introducción

El alineamiento de secuencias es una de las tareas fundamentales en bioinformática, ya que nos permite comparar secuencias de ADN, ARN o proteínas para identificar similitudes, diferencias y relaciones evolutivas. Esto es esencial para entender la función de genes, predecir estructuras proteicas y estudiar la evolución molecular.

## ¿Qué es una alineamiento de secuencias?

Una alineamiento de secuencias (específicamente alineamiento por pares) significa **organizar dos secuencias de forma que las regiones con similitudes se alineen**. Por ejemplo, el alineamiento de `GATTACA` y `GATCA` podría verse así:

```
GATTACA
||| |
GATCA--
```

A simple vista, las alineaciones suelen tener sentido de forma intuitiva, pero cuidado, ya que nuestra intuición puede ser engañosa. Y cada persona puede intuir una alineamiento diferente.

Nuestro cerebro tiende a identificar patrones y similitudes de manera natural. Una vez que percibimos una similitud, nos resulta difícil reconocer otras alternativas que podrían ser igual de buenas (o incluso mejores) que la que estamos observando. Es difícil “dejar de ver” un patrón. Por ejemplo, podríamos haber dispuesto las secuencias así:

```
GATTACA
|||  ||
GAT--CA
```

o también así:

```
GATTACA
|| | ||
GA-T-CA
```

Aquí vemos dos alineaciones más de las mismas secuencias que también parecen concordar bastante bien. Una vez que observamos un patrón, puede llevar un esfuerzo considerable notar que podríamos haber ordenado las letras de otras formas igual de válidas.

## ¿Para qué se utilizan los alineamiento de secuencias?

Los alineamientos tienen dos casos de uso principales:

* **Encontrar regiones similares entre dos secuencias: investigar la relación entre secuencias.**
* **Encontrar qué secuencia (entre muchas alternativas) es más similar a una secuencia de consulta (entrada)**

## ¿Qué determina un alineamiento?

Cada alineamiento está determinada por dos factores:

1. **Algoritmo de alineamiento**: global y local .
2. **Puntuación de alineamiento (parámetros)**: valores numéricos que ajustan la disposición.

Recuerda siempre que:

- **Diferentes algoritmos suelen producir diferentes alineamientos para las mismas secuencias.**
- **Diferentes puntuaciones también suelen generar distintos alineamientos para las mismas secuencias.**

## ¿Cómo se generan los alineamientos?

Supón que tienes las siguientes alineamiento de las secuencias `GATTACA` con `GATCA`:

```
GATTACA      GATTACA      GATTACA
|||.|        |||  ||      || | ||
GATCA--      GAT--CA      GA-T-CA
```

¿Cuál de estas es el alineamiento considerado como “mejor”, “correcta” o “significativa”?

El “mejor” alineamiento depende de cómo valoras la forma en que se alinean las bases. 

¿Consideras que una discrepancia es menos perturbadora que los espacios vacíos? 

> **El valor que asignes a una coincidencia, discrepancia o espacio vacío se denomina puntuación.**

**El concepto más importante de alineamiento es el siguiente:**

- `No existe un alineamiento universalmente mejor.`
- `Solo existe el mejor alineamiento en relación con una elección de puntuación. Cambiar la puntuación generalmente cambiará lo que se selecciona como el mejor alineamiento.`
- `Los algoritmos de alineamiento encuentran la disposición que produce la puntuación máxima en toda el alineamiento.`

Los algoritmos de alineamiento de secuencias aceleran el proceso de seleccionar el alineamiento con la mejor puntuación. Puedes pensar en los alineadores como métodos para evaluar rápidamente todas las combinaciones posibles y, de esas, elegir los alineamientos con la mejor puntuación.

## ¿Cómo funciona la puntuación de alineamiento?

Las puntuaciones son valores, tanto positivos como negativos, que un algoritmo asignará a varias bases al alinearlas de cierta manera. Un alineador intenta crear una disposición que maximice la puntuación.

Por ejemplo, podríamos elegir las siguientes puntuaciones:

- `1` punto por coincidencia.
- `-1` punto por discrepancia.
- `-2` puntos por abrir un espacio.
- `-1` punto por extender un espacio ya abierto.

Luego le indicamos al alineador que encuentre los mejores alineamientos utilizando estas puntuaciones. Con este sistema, los alineamientos se puntuarían así (las puntuaciones se muestran en la parte inferior):

```
GATTACA      GATTACA      GATTACA
|||.|        |||  ||      || | ||
GATCA--      GAT--CA      GA-T-CA

   0            2             1
```

Con esta puntuación, el segundo alineamiento tiene la puntuación más alta (2) y se considera el "mejor" alineamiento.

Cuando las secuencias tienen diferentes longitudes, a menudo queremos encontrar la secuencia más larga que cubra la secuencia más corta. Para ajustar esto, aplicamos una corrección que no penaliza los espacios al final de ninguna de las secuencias. Si aplicamos esta corrección, las puntuaciones serían:

```
GATTACA      GATTACA      GATTACA
|||.|        |||  ||      || | ||
GATCA--      GAT--CA      GA-T-CA

   3            2             1
```

Ahora, el primer alineamiento tiene la puntuación máxima y se convierte en el "mejor alineamiento". Otros cambios en la puntuación podrían afectar qué el alineamiento se considere mejor.

## ¿Cómo elijo las puntuaciones?

En la mayoría de los casos, se comienza con puntuaciones conocidas, calculadas a partir de la tasa de sustitución a lo largo de la historia evolutiva. Existen varios esquemas de puntuación alternativos creados a partir de estas observaciones. Para obtener matrices de puntuación recomendadas, puedes consultar, por ejemplo:

```bash
curl -O ftp://ftp.ncbi.nlm.nih.gov/blast/matrices/NUC.4.4
cat NUC.4.4
```

El archivo muestra que la contribución de puntuación por hacer coincidir una A con una A es de 5 puntos, mientras que la contribución de una coincidencia entre una A y una T es de -4 puntos (es decir, una penalización de 4 puntos).

Al observar visualmente la matriz de puntuación anterior, ya podemos deducir que favorecerá las discrepancias. Por ejemplo, generará fácilmente una alineamiento con discrepancias de la siguiente forma:

```bash
pip install bio --upgrade

bio align ATG ACG --local -match 5 -mismatch 4
```

lo cual produce:

```
ATG
|.|
ACG
```

La segunda coincidencia G/G añadirá 5 puntos, y esta contribución superará la penalización de -4 introducida por la discrepancia. Si, en cambio, escribiéramos:

```bash
bio align ATG ACG --local -match 4 -mismatch 5
```

el alineamiento local sería simplemente:

```
A
|
A
```

el alineamiento no puede extenderse más hacia la izquierda, ya que **"no vale la pena"** añadir una penalización de -5 cuando la recompensa es solo de 4.

Elegir la matriz adecuada, especialmente para secuencias de proteínas, es aún más complicado.

## Tipos de matrices de puntuación

### 1. **Matrices para secuencias de nucleótidos (ADN/ARN)**
Estas matrices son más simples, ya que solo hay 4 bases posibles (A, T, C, G para ADN; A, U, C, G para ARN). Las más utilizadas son:

#### a) **Matriz de identidad**
   - Asigna un valor fijo para coincidencias y otro para discrepancias.
   - Ejemplo:
     - Coincidencia: `+1`
     - Discrepancia: `-1`
   - Es útil para alineamientos simples, pero no tiene en cuenta las sustituciones más probables en la evolución.

#### b) **Matriz de sustitución de nucleótidos**
   - Asigna puntuaciones basadas en la probabilidad de que una base se sustituya por otra durante la evolución.
   - Ejemplo: Matriz **NUC.4.4** (usada en BLAST):
     - Coincidencia A-A: `+5`
     - Coincidencia A-T: `-4` (penalización porque A y T no suelen sustituirse fácilmente).

---

### 2. **Matrices para secuencias de proteínas (aminoácidos)**
Estas matrices son más complejas, ya que hay 20 aminoácidos posibles. Las más utilizadas son:

#### a) **Matrices PAM (Point Accepted Mutation)**
   - Desarrolladas por Margaret Dayhoff en 1978.
   - Basadas en la probabilidad de que un aminoácido se sustituya por otro en secuencias evolutivamente relacionadas.
   - Ejemplos:
     - **PAM1:** Representa un 1% de cambio en las secuencias.
     - **PAM250:** Representa un 250% de cambio (más útil para secuencias distantes).
   - Las matrices PAM son útiles para estudiar relaciones evolutivas a largo plazo.

#### b) **Matrices BLOSUM (BLOcks SUbstitution Matrix)**
   - Desarrolladas por Steven Henikoff y Jorja Henikoff en 1992.
   - Basadas en bloques conservados de secuencias de proteínas.
   - Ejemplos:
     - **BLOSUM62:** Ideal para secuencias moderadamente relacionadas (usada por defecto en BLAST).
     - **BLOSUM45:** Para secuencias más distantes.
     - **BLOSUM80:** Para secuencias muy similares.
   - Las matrices BLOSUM son más utilizadas que las PAM en la actualidad.

![PAM vs BLOSUM](https://resources.qiagenbioinformatics.com/manuals/clcgenomicsworkbench/650/pam-blosum.png)

#### c) **Matrices especializadas**
   - Algunas matrices están diseñadas para casos específicos, como:
     - **Matrices para dominios proteicos:** Se enfocan en regiones específicas de las proteínas.
     - **Matrices para secuencias transmembrana:** Consideran las propiedades físicas de los aminoácidos en membranas celulares.

---

### 3. **Matrices personalizadas**
   - En algunos casos, se pueden crear matrices de puntuación personalizadas basadas en datos específicos del estudio. Por ejemplo:
     - Si se trabaja con un conjunto de secuencias muy específico (como virus o bacterias), se puede calcular una matriz que refleje las sustituciones más comunes en ese grupo.

---

### 4. **Matrices con penalización por gaps**
   - Además de las matrices de sustitución, es necesario definir cómo se penalizan los espacios (gaps) en un alineamiento. Esto se hace con dos parámetros:
     - **Penalización por abrir un gap (gap open):** Penaliza el inicio de un espacio.
     - **Penalización por extender un gap (gap extend):** Penaliza cada espacio adicional después del primero.
   - Ejemplo:
     - Gap open: `-11`
     - Gap extend: `-1`
   - La penalización por extensión de un espacio es típicamente mucho menor que la penalización por abrir uno, lo que tiene una justificación biológica.
   - 
---

### Resumen de los tipos de matrices

| **Tipo de matriz**       | **Aplicación**                                                                 | **Ejemplos**                     |
|---------------------------|-------------------------------------------------------------------------------|----------------------------------|
| **Identidad**             | Alineamientos simples de nucleótidos.                                         | Coincidencia: +1, Discrepancia: -1 |
| **NUC.4.4**               | Alineamientos de nucleótidos con sustituciones probables.                     | Usada en BLAST.                  |
| **PAM**                   | Estudios evolutivos de proteínas (secuencias distantes).                      | PAM1, PAM250.                    |
| **BLOSUM**                | Alineamientos de proteínas (secuencias moderadamente relacionadas o similares).| BLOSUM62, BLOSUM45.              |
| **Especializadas**        | Casos específicos (dominios proteicos, membranas, etc.).                      | Matrices personalizadas.         |
| **Con penalización gaps** | Define cómo se penalizan los espacios en un alineamiento.                     | Gap open: -10, Gap extend: -1.   |


### ¿Cómo elegimos la matriz correcta?

La matriz de sustitución define la puntuación de cada coincidencia y desajuste entre elementos de las secuencias (aminoácidos o nucleótidos), lo cual afecta drásticamente en el alineamiento. Aquí tienes algunos ejemplos:

- **Matrices para ADN:** Existen matrices específicas para alineamientos de secuencias de ADN, como EDNAFULL, que es una matriz estándar para comparar nucleótidos.
  
- **Matrices para proteínas:** Para proteínas, se utilizan matrices BLOSUM (p. ej., BLOSUM30, BLOSUM62, BLOSUM90) o PAM. Cada matriz tiene sus propias particularidades en cuanto a la puntuación de coincidencias y desajustes. Generalmente:
   - **BLOSUM30** es útil para secuencias distantes (más permisiva con desajustes).
   - **BLOSUM90** es más restrictiva y adecuada para secuencias más similares.

Selección de la Matriz de Puntuación de Similitud Correcta por William Pearson, el autor del programa FASTA.

Aquí hay algunas líneas del resumen:

Si bien las matrices “deep” proporcionan búsquedas de similitud muy sensibles, también requieren alineaciones de secuencia más largas y, a veces, pueden producir una sobreextensión de alineamiento en regiones no homólogas. Las matrices de puntuación más superficiales son más efectivas cuando se buscan dominios proteicos cortos, o cuando el objetivo es limitar el alcance de la búsqueda a secuencias que probablemente sean ortólogas entre organismos recientemente divergentes.

![image](https://biodataprog.github.io/GEN220_2019/Bioinformatics/images/orthologs.gif)


Del mismo modo, en las búsquedas de ADN, los parámetros de coincidencia y desajuste en las búsquedas de ADN reflejan una especie de "retroceso evolutivo" o antigüedad de la similitud, estableciendo también límites de dominio en función de las probabilidades de cambio entre nucleótidos a lo largo del tiempo.

### ¿Cómo “veo” la matriz?

`bio align` imprimirá la matriz (cuando utilice valores incorporados):

```bash
bio align -matrix PAM250 | head -15
```

impresiones:

```
#
# This matrix was produced by "pam" Version 1.0.6 [28-Jul-93]
#
# PAM 250 substitution matrix, scale = ln(2)/3 = 0.231049
#
# Expected score = -0.844, Entropy = 0.354 bits
#
# Lowest score = -8, Highest score = 17
#
   A  R  N  D  C  Q  E  G  H  I  L  K  M  F  P  S  T  W  Y  V  B  Z  X  *
A  2 -2  0  0 -2  0  0  1 -1 -1 -2 -1 -1 -3  1  1  1 -6 -3  0  0  0  0 -8
R -2  6  0 -1 -4  1 -1 -3  2 -2 -3  3  0 -4  0  0 -1  2 -4 -2 -1  0 -1 -8
N  0  0  2  2 -4  1  1  0  2 -2 -3  1 -2 -3  0  1  0 -4 -2 -2  2  1  0 -8
D  0 -1  2  4 -5  2  3  1  1 -2 -4  0 -3 -6 -1  0  0 -7 -4 -2  3  3 -1 -8
C -2 -4 -4 -5 12 -5 -5 -3 -3 -2 -6 -5 -5 -4 -3  0 -2 -8  0 -2 -4 -5 -3 -8
...
```

### ¿Por qué los valores de puntuación son números enteros?

En las matrices de sustitución (como BLOSUM o PAM), los valores de puntuación representan la probabilidad relativa de que una sustitución específica ocurra durante la evolución de las secuencias. Esta probabilidad se expresa en términos de logaritmos en base 2 (log2), que son más manejables en alineaciones de secuencias.

**Logaritmos y Probabilidades en Puntuaciones**

Los valores de puntuación se obtienen de la probabilidad de que dos aminoácidos específicos se sustituyan entre sí en relación con la probabilidad de que se alineen al azar.

Al representar las puntuaciones en logaritmos base 2 (log2), podemos expresar el cambio en probabilidad en términos de potencias de 2. Esto facilita la interpretación de las puntuaciones.

Así, una puntuación de 3 implica una probabilidad de sustitución de 2^3 = 8 veces más probable que al azar, mientras que una puntuación de 5 implica una probabilidad de 2^5 = 32 veces. La sustitución con puntuación 3 es cuatro veces más probable que una con puntuación 5.


---

## Cómo se muestran los alineamientos

No existe un formato universal para mostrar alineamientos, pero comúnmente usamos un formato visual con caracteres adicionales para interpretarlas:

- El carácter `-` indica un espacio.
- El carácter `|` muestra una coincidencia.
- El carácter `.` muestra una discrepancia.

Ejemplo:

```
ATGCAAATGACAAAT-CGA
||||   |||.||.| |||
ATGC---TGATAACTGCGA
```

De las bases anteriores, 13 son iguales (13 identidades), 5 bases están ausentes (5 espacios), y 2 bases son diferentes (2 discrepancias).

## ¿Importa cuál secuencia está en la parte superior?

Usualmente, a menos que se indique explícitamente lo contrario, leemos e interpretamos el alineamiento como si estuviéramos comparando la secuencia inferior con la superior.

```
ATGCAAATGACAAAT-CGA
||||   |||.||.| |||
ATGC---TGATAACTGCGA
```

Mientras que la palabra “espacio” es genérica y se refiere a cualquiera de las secuencias, si queremos ser más específicos, podríamos decir que el alineamiento anterior muestra tres deleciones de A y una inserción de G. La palabra “deleción” significa que la segunda secuencia tiene bases faltantes en comparación con la primera.

Podríamos generar y mostrar esta misma alineamiento al revés:

```
ATGC---TGATAACTGCGA
||||   |||.||.| |||
ATGCAAATGACAAAT-CGA
```

Este alineamiento se describiría ahora como: contiene tres inserciones de A seguidas posteriormente de una deleción de G en relación con la secuencia superior. La eliminación en una secuencia es una inserción en la otra; todo depende de lo que trata el estudio.

## Alineaciones globales y locales

Para ejecutar los ejemplos, instala el paquete `bio`:

```bash
pip install bio --upgrade
```

Puedes leer más sobre cómo funciona el método `bio align` en [https://www.bioinfo.help/bio-align.html](https://www.bioinfo.help/bio-align.html).

### Nota importante sobre alineamientos

Los algoritmos de alineamiento en `bio align` utilizan la implementación de BioPython, ideal para uso interactivo y exploratorio al alinear secuencias relativamente cortas (de hasta aproximadamente 30 kb de longitud).

El software especializado en alineamiento genómica suele operar con órdenes de magnitud más rápidos, aunque ajusta características a cambio de esa mayor velocidad. Dependiendo de tus necesidades, es posible que desees usar uno de los numerosos alineadores disponibles, como: `blast`, `blat`, `fasta36`, `mummer`, `minimap2`, `lastz`, `lastal`, `exonerate`, `vsearch`, `diamond`, etc.

La gran cantidad de opciones de alineadores, junto con el enorme número de parámetros que cada alineador permite personalizar, muestra que las alineaciones son tareas complejas y multifacéticas.

### Cómo realizar una alineamiento con `bio`

Verifica que el script funcione con:

```bash
bio align THISLINE ISALIGNED
```

Esto imprimirá:

```
# seq1 (8) vs seq2 (9)
# pident=36.4% len=11 ident=4 mis=2 del=2 ins=3
# semiglobal: score=8.0 matrix=BLOSUM62 gapopen=11 gapextend=1

THISLI--NE-
--||..--||-
--ISALIGNED
```

La herramienta `bio align` está diseñada de tal manera que permite utilizar tanto cadenas de texto como nombres de archivos FASTA. Podemos obtener una matriz de puntuación diferente:

```bash
wget -nc ftp://ftp.ncbi.nlm.nih.gov/blast/matrices/BLOSUM30
```

Y luego usar la matriz de puntuación personalizada de la siguiente manera:

```bash
bio align THISLINE ISALIGNED --matrix BLOSUM30
```

Esto imprimirá:

```
# seq1 (8) vs seq2 (9)
# pident=36.4% len=11 ident=4 mis=2 del=2 ins=3
# semiglobal: score=13.0 matrix=BLOSUM30 gapopen=11 gapextend=1

THIS--LINE-
--||--..||-
--ISALIGNED
```

En los ejemplos a continuación, utilizaremos secuencias proteicas hipotéticas `THISLINE` e `ISALIGNED`, palabras reales que también resultan ser secuencias peptídicas válidas. Estas secuencias se utilizaron por primera vez con un propósito similar, aunque en un contexto diferente, en el libro *Understanding Bioinformatics* de Marketa Zvelebil y Jeremy Baum.

### ¿Qué es una alineamiento global?

![image](https://microbenotes.com/wp-content/uploads/2023/03/Global-and-Local-Alignment-of-two-sequences.jpg)

**Una alineamiento global busca maximizar las similitudes a lo largo de toda la longitud de las secuencias, lo que significa que intenta encontrar la mejor correspondencia de extremo a extremo para ambas secuencias.** Esta técnica es especialmente útil cuando las secuencias tienen tamaños similares y queremos compararlas en su totalidad.

La herramienta `bio align`, por defecto, realiza una alineamiento semi-global. Esto significa que permite que haya "gaps" (huecos) al final de la secuencia de consulta (o "query") sin penalizarlos. En otras palabras, si al final de la consulta quedan bases sin alinear, no se les resta puntos. Este tipo de alineamiento es útil cuando queremos comparar una secuencia corta con una más larga, sin preocuparnos por los extremos.

**Por otro lado, cuando hacemos una alineamiento global, tratamos de alinear las dos secuencias en toda su longitud, de principio a fin.** En este caso, cada base de una secuencia debe estar alineada con una base de la otra secuencia, o si no hay coincidencia, se coloca un “gap” en la segunda secuencia.

Ejemplo: Imagina que tienes dos secuencias de ADN:

- Secuencia 1 (consulta): ACTG
- Secuencia 2: GACTGA

En una alineamiento semi-global, el programa puede hacer algo como esto:

```
   ACTG
   ||||
  GACTGA
```
Aquí, las bases coinciden sin importar que al final de la segunda secuencia haya más letras.

En cambio, en una alineamiento global, se alinea la totalidad de ambas secuencias y se usan “gaps” cuando es necesario para mantener la longitud. Así se vería:

```
 -ACTG-
  |||||
 GACTGA
```

Aquí, los huecos adicionales se agregan al final de la primera secuencia (`ACTG-`) para que coincida con la longitud de la segunda (`GACTGA`).

Usamos alineamientos globales cuando buscamos un arreglo que maximice las similitudes en toda la longitud de ambas secuencias:

```bash
bio align THISLINE ISALIGNED --global
```

eso imprime:

```
# seq1 (8) vs seq2 (9)
# pident=22.2% len=9 ident=2 mis=6 del=0 ins=1
# global: score=-7.0 matrix=BLOSUM62 gapopen=11 gapextend=1

THISLINE-
......||-
ISALIGNED
```

También podemos anular la brecha abierta y la penalización de extensión de la brecha:

```bash
bio align THISLINE ISALIGNED --global --open 7
```

ahora el alineamiento se ve así:

```
# seq1 (8) vs seq2 (9)
# pident=54.5% len=11 ident=6 mis=0 del=2 ins=3
# global: score=6.0 matrix=BLOSUM62 gapopen=5 gapextend=1

THIS-LI-NE-
--||-||-||-
--ISALIGNED
```

Tenga en cuenta cuán radicalmente diferente es el segundo alineamiento que de la primera. Todo lo que hicimos fue reducir la pena de abrir una brecha `11` a `7`. El alineamiento es más largo pero tiene más huecos. La compensación es fácilmente evidente.

Recuerde, un alineamiento encuentra la disposición que maximiza la puntuación de recompensas y penalizaciones en todas las secuencias.

### ¿Qué es un alineamieto local?

**Los alineamientos locales se utilizan cuando necesitamos encontrar la región de similitud máxima entre dos secuencias.** 

Esto es particularmente útil cuando solo una parte de las secuencias es relevante, por ejemplo, cuando una proteína contiene dominios específicos similares a otras proteínas. Al realizar alineamientos locales, los algoritmos buscan el intervalo de puntuación más alto (es decir, la región donde las secuencias son más similares) y lo reporta. Así, un alineamiento local puede ser una pequeña parte de cada secuencia, en lugar de intentar alinear todos los elementos de principio a fin.

```bash
bio align THISLINE ISALIGNED --local
```

Cuando se ejecuta como arriba, el alineamiento local generada con los parámetros predeterminados será sorprendentemente corta:

```
# seq1 (2) vs seq2 (2)
# pident=100.0% len=2 ident=2 mis=0 del=0 ins=0
# local: score=11.0 matrix=BLOSUM62 gapopen=11 gapextend=1

NE
||
NE
```

El algoritmo nos dice que estos dos aminoácidos coincidentes producen la puntuación más alta posible (11 en este caso) y cualquier otra alineamiento local entre las dos secuencias producirá una puntuación peor que 11.

Podemos usar otras matrices de puntuación como se muestra en ftp://ftp.ncbi.nlm.nih.gov/blast/matrices/, muchos de estos están incluidos con bio.

```
#  DNA matrices
# This matrix is the "standard" EDNAFULL substitution matrix.
wget ftp://ftp.ncbi.nlm.nih.gov/blast/matrices/NUC.4.4

# Protein matrices
# Get the BLOSUM30, BLOSUM62, and BLOSUM90 matrices.
wget -nc ftp://ftp.ncbi.nlm.nih.gov/blast/matrices/BLOSUM30
```

Obsérvese cómo el alineamiento local se ve afectada por el esquema de puntuación.

```
bio align THISLINE ISALIGNED -matrix BLOSUM30 --local
Usando el BLOSUM90 el esquema de puntuación produce una alineamiento mucho más larga:

# seq1 (4) vs seq2 (4)
# pident=50.0% len=4 ident=2 mis=2 del=0 ins=0
# local: score=15.0 matrix=BLOSUM30 gapopen=11 gapextend=1

LINE
..||
IGNE
```

## Entonces quieres alinear secuencias - Alineamiento por pares

El **alineamiento por pares** consiste en alinear dos secuencias, que llamaremos **consulta** (query) y **objetivo** (target). Este proceso es fundamental en bioinformática para comparar y analizar similitudes o diferencias entre secuencias.

### ¿Cómo elegir el software adecuado para el alineamiento?

Existen muchas herramientas para realizar alineamientos por pares. A continuación, te damos una guía sencilla para elegir la adecuada.

#### Preguntas clave para elegir un alineador:

1. **¿Qué tipo de secuencias vas a alinear?**  
   - ¿Son ADN, ARN o proteínas?

2. **¿Cuánto miden las secuencias?**  
   - Algunas herramientas funcionan mejor con secuencias largas o cortas.

3. **¿Qué diferencias esperas entre las secuencias?**  
   - Pequeñas mutaciones, inserciones, deleciones, etc.

4. **¿Cuántas secuencias necesitas alinear?**  
   - Esto afecta el tiempo de ejecución y los recursos necesarios.

5. **¿Necesitas el mejor alineamiento o varios alineamientos posibles?**  
   - Algunos programas solo ofrecen el alineamiento con la mejor puntuación, mientras que otros incluyen alineamientos secundarios.

6. **¿Qué información necesitas extraer del alineamiento?**  
   - ¿Solo quieres la secuencia alineada o también estadísticas?

7. **¿Cuál es el tiempo de ejecución esperado?**  
   - Algunos alineadores son rápidos pero menos precisos, mientras que otros son más lentos pero óptimos.

### ¿Qué pasa si elegimos mal la herramienta?

Si el software no es el adecuado, podrías enfrentar problemas como:

- Ejecuciones lentas.
- Bloqueos o consumo excesivo de memoria.
- Resultados inesperados o sin sentido.
  
No te preocupes, ¡nos ha pasado a todos! 🧠

### ¿Cómo elegir la secuencia "query"?

El término **query/target** afecta cómo interpretamos las diferencias entre las secuencias. Por ejemplo:

- Si una base está ausente en una secuencia, el alineador generará un **gap** (brecha).
  - Si la **query** es la secuencia más corta, se interpreta como una **deleción**.
  - Si la **query** es la más larga, se interpreta como una **inserción**.

En resumen: **Las diferencias siempre se describen en relación con la query.**

### Tipos de alineamientos y sus características

#### 1. **Decidir el tipo de secuencia**
   - ¿Vas a alinear ADN, ARN o péptidos?  
   - Algunos programas, como **lastz**, pueden manejar alfabetos extendidos de ADN (donde `W` significa A o T, por ejemplo).

#### 2. **Considerar regiones de baja complejidad**
   - Herramientas como **lastz** pueden ignorar regiones con letras en minúscula (secuencias de baja complejidad). Si necesitas incluirlas, asegúrate de configurar el programa correctamente.

### Rendimiento: Velocidad vs. Precisión

No todas las herramientas ofrecen el mismo balance entre velocidad y precisión:

1. **Herramientas rápidas**  
   - Útiles para alineamientos simples o secuencias similares.
   - Pueden usar heurísticas y no siempre encuentran alineamientos en secuencias muy diferentes.

2. **Herramientas precisas (matemáticamente óptimas)**  
   - Ideales para secuencias muy diferentes, pero suelen ser más lentas.

### Factores limitantes más comunes

1. **Longitud de las secuencias**  
   - Cuanto más largas sean las secuencias, mayor será el tiempo de cálculo.

2. **Número de alineamientos necesarios**  
   - Comparar muchas secuencias aumenta el tiempo de ejecución y el uso de memoria.

### Instalación de herramientas

Algunos alineadores populares que puedes instalar con **mamba**:

```bash
conda install -n base -c conda-forge mamba
mamba create -n bioinfo python=3.9
conda activate bioinfo
mamba install -c bioconda mafft exonerate lastz
```

## Recursos útiles

- [MAFFT](https://mafft.cbrc.jp/alignment/software/)
- [Exonerate](https://www.ebi.ac.uk/about/vertebrate-genomics/software/exonerate)
- [Lastz](http://www.bx.psu.edu/~rsharris/lastz/)

¡Con esta guía ya tienes las bases para realizar alineamientos por pares! 🚀

## Parte práctica: Herramientas para el alineamiento de secuencias

En esta sección exploraremos cómo realizar alineamientos de secuencias utilizando diferentes herramientas. 🧪

### Obteniendo los datos

Para nuestros ejemplos, usaremos datos reales. ¡Primero descarguemos las secuencias! 👇

```bash
# Obtener el genoma completo de SARS-CoV-2 y el SARS-CoV de murciélagos
bio fetch NC_045512 --format fasta > genome1.fa
bio fetch MN996532  --format fasta > genome2.fa

# Obtener la proteína S de SARS-CoV-2 y el SARS-CoV de murciélagos
bio fetch YP_009724390 --format fasta > pep1.fa
bio fetch QHR63300 --format fasta > pep2.fa

# Obtener transcrito y CDS del gen BRAF humano
bio fetch ENST00000288602 > transcript-full.fa
bio fetch ENST00000288602 --type cds > transcript-cds.fa
```

### Analizando las secuencias

Podemos ver estadísticas básicas de estas secuencias con `seqkit`:

```bash
seqkit stats *.fa
```

Salida esperada:

```plaintext
file                 format  type     num_seqs  sum_len  min_len  avg_len  max_len
genome1.fa           FASTA   DNA             1   29,903   29,903   29,903   29,903
genome2.fa           FASTA   DNA             1   29,855   29,855   29,855   29,855
pep1.fa              FASTA   Protein         1    1,273    1,273    1,273    1,273
pep2.fa              FASTA   Protein         1    1,269    1,269    1,269    1,269
transcript-cds.fa    FASTA   DNA             1    2,421    2,421    2,421    2,421
transcript-full.fa   FASTA   DNA             1  190,247  190,247  190,247  190,247
```

✨ **Nota:** Esta herramienta es excelente para verificar que los datos sean consistentes antes de continuar.

### Herramienta 1: `bio align`

`bio align` es una herramienta educativa basada en BioPython para aprender alineamientos por pares.

#### Características:
- Soporta alineamientos locales, globales y semi-globales.
- Reconoce automáticamente ADN y proteínas.
- Ideal para alineamientos rápidos y cortos.
- Limitado para secuencias largas (>10Kb).

#### Ejemplo 1: Alineando dos secuencias desde la línea de comandos

```bash
bio align GATTACA GATCA
```

Salida esperada:

```plaintext
# seq1 (7) vs seq2 (5)
# pident=57.1% len=7 ident=4 mis=1 del=2 ins=0
# semiglobal: score=2.0 match=1 mismatch=2 gapopen=11 gapextend=1

GATTACA
|||.|--
GATCA--
```

🎯 **Concepto clave:** Observa cómo se calculan las diferencias entre ambas secuencias (identidades, deleciones, etc.).

#### Ejemplo 2: Alineando proteínas desde archivos

```bash
bio align pep1.fa pep2.fa
```

Salida esperada:

```plaintext
# YP_009724390.1 (1273) vs QHR63300.2 (1269)
# pident=97.4% len=1273 ident=1240 mis=29 del=4 ins=0
# semiglobal: score=6541.0 matrix=BLOSUM62 gapopen=11 gapextend=1
```

### Herramienta 2: BLAST

BLAST (Basic Local Alignment Search Tool) es la herramienta más utilizada en bioinformática para alineamientos y búsquedas en bases de datos.

#### Características:
- Realiza alineamientos locales.
- Muy eficiente y ampliamente aceptada.
- Ofrece múltiples formatos de salida.

#### Ejemplo 1: Alineando genomas completos

```bash
blastn -query genome1.fa -subject genome2.fa -outfmt 7
```

Salida esperada:

```plaintext
# Query: NC_045512.2 Severe acute respiratory syndrome coronavirus 2 isolate Wuhan-Hu-1, complete genome
# Subject: MN996532.2 Bat coronavirus RaTG13, complete genome
# Fields: query id, subject id, % identity, alignment length, mismatches, gap opens, q. start, q. end, s. start, s. end, evalue, bit score
NC_045512.2    MN996532.2 96.14  29877  1130   7  1  29875  1  29855  0.0    48758
```

#### Ejemplo 2: Alineando proteínas

```bash
blastp -query pep1.fa -subject pep2.fa
```

Salida esperada:

```plaintext
BLASTP 2.2.29+


Query= YP_009724390.1 surface glycoprotein [Severe acute respiratory
syndrome coronavirus 2]

Length=1273

Subject= QHR63300.2 spike glycoprotein [Bat coronavirus RaTG13]

Length=1269


 Score =  2565 bits (6648),  Expect = 0.0, Method: Compositional matrix adjust.
 Identities = 1240/1273 (97%), Positives = 1252/1273 (98%), Gaps = 4/1273 (0%)

Query  1     MFVFLVLLPLVSSQCVNLTTRTQLPPAYTNSFTRGVYYPDKVFRSSVLHSTQDLFLPFFS  60
             MFVFLVLLPLVSSQCVNLTTRTQLPPAYTNS TRGVYYPDKVFRSSVLH TQDLFLPFFS
Sbjct  1     MFVFLVLLPLVSSQCVNLTTRTQLPPAYTNSSTRGVYYPDKVFRSSVLHLTQDLFLPFFS  60

...
```

#### Ejemplo 2: Alineando CDS a Transcritos

```bash
blastn -query transcript-cds.fa -subject transcript-full.fa  -outfmt 7
```

Salida esperada:

```plaintext
# BLASTN 2.2.29+
# Query: ENST00000288602.11
# Subject: ENST00000288602.11 chromosome:GRCh38:7:140734486:140924732:-1
# Fields: query id, subject id, % identity, alignment length, mismatches, gap opens, q. start, q. end, s. start, s. end, evalue, bit score
# 19 hits found
ENST00000288602.11 ENST00000288602.11 100.00 268    0  0  239    506    89859  90126  2e-141   496
ENST00000288602.11 ENST00000288602.11 100.00 180    0  0  1635   1814   147642 147821 2e-92    333
ENST00000288602.11 ENST00000288602.11 100.00 174    0  0  2248   2421   189963 190136 4e-89    322
ENST00000288602.11 ENST00000288602.11 100.00 162    0  0  980    1141   130265 130426 2e-82    300
ENST00000288602.11 ENST00000288602.11 100.00 149    0  0  712    860    123173 123321 3e-75    276
ENST00000288602.11 ENST00000288602.11 100.00 141    0  0  1  141    30 170    9e-71    261
ENST00000288602.11 ENST00000288602.11 100.00 139    0  0  1296   1434   141574 141712 1e-69    257
ENST00000288602.11 ENST00000288602.11 100.00 139    0  0  2109   2247   184783 184921 1e-69    257
```

### Herramienta 3: Minimap2

Minimap2 es un alineador altamente eficiente para lecturas largas o genomas completos.

#### Características:
- Alineamientos semi-globales.
- Soporta secuencias largas (hasta 100Mb).
- Produce formatos de salida PAF y SAM.

#### Ejemplo: Comparando dos genomas

```bash
minimap2 -c genome1.fa genome2.fa > alignment.paf
```

---

### Herramienta 4: MAFFT

MAFFT es un alineador de secuencias múltiples, pero también funciona en modo por pares.

#### Ejemplo: Alineando genomas

```bash
cat genome1.fa genome2.fa > all.fa
mafft --preservecase --auto all.fa > aligned.fa
```

🎯 **Salida clave:**  
Puedes usar `bio format` para transformar el alineamiento en un formato más claro.

---

## Herramienta 5: LASTZ

LASTZ es un alineador versátil diseñado para análisis comparativos de genomas.

#### Ejemplo:

```bash
lastz genome1.fa genome2.fa --format=paf
```

---

### Herramienta 6: Exonerate

Exonerate permite alineamientos versátiles (locales, globales, con y sin gaps).

#### Ejemplo:

```bash
exonerate --model affine:local transcript-cds.fa transcript-full.fa
```

---

### Herramienta 7: MUMmer

MUMmer es perfecto para comparar genomas completos rápidamente.

#### Ejemplo:

```bash
nucmer genome1.fa genome2.fa -p genome_alignment
show-coords -r genome_alignment.delta
```

---

## **Alineamiento Múltiple de Secuencias (MSA)**

### **¿Qué es el alineamiento múltiple?**
El alineamiento múltiple consiste en alinear **tres o más secuencias** (de ADN, ARN o proteínas) de manera que se maximice la similitud entre ellas. Esto permite identificar regiones conservadas, patrones evolutivos y relaciones funcionales entre las secuencias.

#### Ejemplo:
```
Secuencia 1: ATGCTAGCT
Secuencia 2: ATG-TAG-T
Secuencia 3: AT--TAGCT
```

---

### **Importancia del alineamiento múltiple**
- **Identificación de regiones conservadas:** Útil para encontrar dominios funcionales en proteínas o secuencias regulatorias en ADN.
- **Estudios evolutivos:** Permite reconstruir árboles filogenéticos y analizar relaciones entre especies.
- **Predicción de estructura y función:** Ayuda a inferir la estructura 3D de proteínas y su función biológica.
- **Diseño de primers:** En biología molecular, se usa para diseñar cebadores (primers) que amplifiquen regiones conservadas.

---

### **Métodos para realizar alineamientos múltiples**

#### a) **Métodos progresivos**
- **Cómo funcionan:** Se construye un alineamiento gradual, comenzando con las secuencias más similares y añadiendo las menos similares.
- **Ventajas:** Rápido y eficiente para un número moderado de secuencias.
- **Desventajas:** Depende del orden en que se añaden las secuencias, lo que puede llevar a errores.
- **Herramientas populares:** **Clustal Omega**, **MAFFT**, **T-Coffee**.

#### b) **Métodos basados en consenso**
- **Cómo funcionan:** Usan un perfil de consenso generado a partir de alineamientos pareados para guiar el alineamiento múltiple.
- **Ventajas:** Más precisos que los métodos progresivos.
- **Desventajas:** Computacionalmente más costosos.
- **Herramientas populares:** **MUSCLE**, **ProbCons**.

#### c) **Métodos basados en árboles filogenéticos**
- **Cómo funcionan:** Utilizan un árbol filogenético para guiar el alineamiento, asegurando que las secuencias más relacionadas se alineen primero.
- **Ventajas:** Muy precisos para estudios evolutivos.
- **Desventajas:** Requieren mucho tiempo y recursos computacionales.
- **Herramientas populares:** **PRANK**, **PASTA**.

#### d) **Métodos basados en modelos ocultos de Markov (HMM)**
- **Cómo funcionan:** Usan modelos estadísticos para representar patrones conservados en las secuencias.
- **Ventajas:** Muy útiles para alinear secuencias con baja similitud.
- **Desventajas:** Complejos de implementar y requieren entrenamiento previo.
- **Herramientas populares:** **HMMER**, **Kalign**.

---

### **Formatos de salida**
Los alineamientos múltiples se pueden guardar en diferentes formatos, como:
- **FASTA:** Formato simple y ampliamente utilizado.
- **CLUSTAL:** Formato legible con información de conservación.
- **PHYLIP:** Usado para análisis filogenéticos.
- **Stockholm:** Usado en bases de datos como Pfam.

---

### **Aplicaciones del alineamiento múltiple**
- **Análisis filogenético:** Reconstrucción de árboles evolutivos.
- **Identificación de motivos conservados:** En proteínas o ADN.
- **Diseño de primers:** Para PCR o secuenciación.
- **Predicción de estructura:** Inferir la estructura 3D de proteínas.

### **Consejos para realizar buenos alineamientos múltiples**
1. **Preprocesa las secuencias:** Elimina regiones no informativas (por ejemplo, intrones en ADN).
2. **Elige la herramienta adecuada:** Dependiendo del número y tipo de secuencias.
3. **Verifica el alineamiento:** Usa herramientas de visualización como **Jalview** o **AliView**.
4. **Ajusta parámetros:** Prueba diferentes matrices de sustitución y penalizaciones por gaps.

## Resumen

| **Herramienta** | **Propósito principal**                                                                 | **Tipo de alineamiento**                        | **Entradas típicas**                          | **Fortalezas**                                                                                   | **Limitaciones**                                                                                     |
|------------------|-----------------------------------------------------------------------------------------|-------------------------------------------------|-----------------------------------------------|---------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| **Bio Align**   | Alineamiento múltiple de secuencias biológicas.                                         | Global o local, dependiendo de la configuración.| Secuencias cortas o medianas (ADN, ARN, proteínas). | Herramienta general para análisis básicos; fácil de usar.                                        | No es ideal para grandes volúmenes de datos o genomas completos.                                    |
| **BLAST**       | Búsqueda rápida de similitudes locales entre secuencias.                               | Local.                                          | Secuencias individuales o pequeños conjuntos.  | Ampliamente utilizado, rápido y confiable para búsquedas de homología.                           | No está optimizado para alineamientos a gran escala o genomas completos.                            |
| **MUMmer**      | Alineamiento de genomas completos o fragmentos largos.                                 | Global o local, con énfasis en coincidencias exactas. | Genomas completos o fragmentos largos.        | Muy eficiente para comparar genomas cercanamente relacionados; maneja grandes volúmenes de datos. | Menos adecuado para secuencias muy divergentes o proteínas.                                          |
| **Exonerate**   | Alineamiento de secuencias con modelos específicos, como genes o proteínas.             | Local o semiglobal.                             | Secuencias de ADN, ARN o proteínas.           | Flexible y adaptable a diferentes modelos biológicos; útil para anotación génica.                | Puede ser lento para grandes conjuntos de datos; requiere más configuración que otras herramientas. |
| **MAFFT**       | Alineamiento múltiple de secuencias, especialmente para análisis filogenéticos.         | Global o local.                                 | Secuencias de ADN, ARN o proteínas.           | Preciso y eficiente para alineamientos múltiples; maneja grandes conjuntos de datos.             | Puede ser complejo para usuarios principiantes debido a sus múltiples opciones de configuración.     |
| **LASTZ**       | Búsqueda de similitudes locales entre genomas completos o fragmentos largos.            | Local.                                          | Genomas completos o fragmentos largos.        | Eficiente para comparar genomas grandes; flexible en la detección de similitudes divergentes.     | Requiere más tiempo de procesamiento en comparación con herramientas como Minimap2.                 |
| **Minimap2**    | Alineamiento rápido de secuencias largas (lecturas de nanoporos, PacBio) o genomas.     | Local o global, dependiendo de la configuración.| Lecturas largas, transcriptomas o genomas.    | Extremadamente rápido y eficiente para datos de secuenciación de tercera generación.              | Menos preciso para alineamientos muy específicos o análisis filogenéticos detallados.               |
| **MUSCLE**      | Alineamiento múltiple de secuencias, especialmente para análisis filogenéticos rápidos. | Global.                                         | Secuencias de ADN, ARN o proteínas.           | Rápido y preciso para alineamientos múltiples; fácil de usar y accesible.                        | Menos eficiente para conjuntos de datos extremadamente grandes o secuencias muy divergentes.        |
| **Clustal**     | Alineamiento múltiple de secuencias, con énfasis en precisión.                          | Global.                                         | Secuencias de ADN, ARN o proteínas.           | Fácil de usar; produce alineamientos precisos para conjuntos pequeños o medianos.                | Puede ser lento y menos eficiente para grandes conjuntos de datos; menos flexible que MAFFT o MUSCLE. |

---

### Resumen de uso recomendado:
- **Bio Align**: Para análisis básicos y educativos.
- **BLAST**: Para búsquedas rápidas de homología en bases de datos.
- **MUMmer**: Para comparaciones de genomas completos o fragmentos largos.
- **Exonerate**: Para alineamientos específicos, como la anotación de genes.
- **MAFFT**: Para alineamientos múltiples precisos en estudios filogenéticos.
- **LASTZ**: Para comparaciones detalladas de genomas grandes.
- **Minimap2**: Para datos de secuenciación de tercera generación y alineamientos rápidos.
- **MUSCLE**: Para alineamientos múltiples rápidos y precisos, especialmente en análisis filogenéticos.
- **Clustal**: Para alineamientos múltiples precisos en conjuntos pequeños o medianos de secuencias.

---

¡MUCHAS GRACIAS!
