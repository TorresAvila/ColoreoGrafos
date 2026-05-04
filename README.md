# Optimización de Horarios Académicos mediante Coloreo de Grafos

**Maestría en Ciencias de la Computación — Admisión 2026**
Luis Javier Torres Avila

## 1. El Problema

Una coordinación académica necesita asignar bloques de tiempo de dos horas a nueve clases. La tarea parece simple hasta que se consideran las restricciones: un mismo grupo de estudiantes no puede estar en dos clases al mismo tiempo, y un docente tampoco puede impartir dos clases simultáneamente. Con nueve clases distribuidas entre tres grupos y tres docentes, los conflictos potenciales se multiplican rápidamente y la asignación manual se vuelve propensa a errores.

La pregunta central de este proyecto es: ¿cuál es el menor número de bloques de tiempo suficiente para programar todas las clases sin violar ninguna restricción? Y más importante aún, ¿cómo encontrar ese mínimo de forma sistemática y demostrable?

La respuesta está en traducir el problema a un lenguaje matemático donde estas preguntas ya tienen respuesta: la teoría de grafos.

Esta propuesta aborda el Problema de Asignación de Horarios Académicos definido, donde una coordinación académica debe asignar bloques de tiempo de 2 horas a nueve clases académicas (de la A a la I), sujeta a dos restricciones operativas estrictas: ningún grupo de estudiantes y ningún docente puede ser asignado a más de una clase dentro del mismo bloque. Al modelar formalmente estas restricciones como un **Grafo de Conflictos**, el desafío logístico de la programación se convierte en un problema combinatorio bien estudiado conocido como **Coloreo de Vértices de Grafos**, donde cada color representa un bloque de tiempo y el número mínimo de colores requeridos corresponde al **Número Cromático** `χ(G)

## 2. Bases teóricas
### 2.1 Grafo (*Graph*)

Un **grafo** es una estructura definida como el par  `G = (V, E)` donde **V** es un conjunto de **vértices** y **E** es un conjunto de **aristas**, siendo cada arista un par no ordenado `{u, v}` con `u, v ∈ V` y `u ≠ v`. En este proyecto, `|V| = 9` con un vértice por clase y arista E para un conflicto de horario entre dos clases.

### 2.2 Vértice y Arista (*Vertex and Edge*)

El **vértice** es la parte fundamental que constituye a un grafo. Aquí, cada vértice `v ∈ V` corresponde a una clase académica: `V = {A, B, C, D, E, F, G, H, I}`.

Una **arista** `{u, v} ∈ E` se traza entre la clase `u` y la clase `v` si y solo si:

```
Grupo(u) = Grupo(v)      [Conflicto de Grupo — mismos estudiantes]
    O
Docente(u) = Docente(v)  [Conflicto de Docente — mismo instructor]
```

Esta regla de aristas define cómo se construye el grafo de conflictos a partir de los datos: se itera sobre cada par de clases y se aplica la regla anterior.

### 2.3 Adyacencia (*Adjacency*)

Dos vértices `u` y `v` son **adyacentes** (vecinos) si la arista `{u, v} ∈ E`. El conjunto completo de vecinos de un vértice `u` — formalmente su **vecindario** — es:

```
N(u) = { v ∈ V | {u, v} ∈ E }
```

El **grado** del vértice `u` es `deg(u) = |N(u)|`, y representa el número total de conflictos que esa clase tiene con otras. Un vértice de grado alto es el más restringido y más difícil de programar. En este grafo, cada vértice tiene grado 4 donde cada clase conflictúa con sus 2 compañeros de grupo y sus 2 compañeros de docente.

El vecindario `N(u)` es el concepto formal que rige tanto la lógica de los algoritmos como el análisis de complejidad: el algoritmo Greedy debe recorrer `N(u)` para cada vértice `u`, y el trabajo total realizado sobre todos los vértices es igual a `∑ deg(u) = 2|E|`

### 2.4 Clique (*Clique / Subgrafo Completo*)

Un **clique** es un subconjunto donde cada par de vértices es adyacente

El **clique máximo** `ω(G)` es el subconjunto más grande de este tipo. Éste constituye el piso mínimo del horario onde no es poible crear un conjutno mas pequeño de bloques dado que todos los subconjuntos conflictúan erntre sí.

Esto se debe a que cada vértice en un clique conflictúa con todos los demás. No es posible programar un clique de tamaño menor a estas restricciones. En este proyecto, cada uno de los tres grupos (G1, G2, G3) y cada uno de los tres docentes (T1, T2, T3) forman un clique de tamaño 3, por lo que `ω(G) ≥ 3`, lo que significa que **se requieren al menos 3 bloques de tiempo**.

### Número Cromático (*Chromatic Number*)

El **número cromático** `χ(G)` es el número mínimo de colores necesarios para un coloreo válido de vértices.

Un horario que usa exactamente su número cromático de bloques es óptimo. Encontrar el número cromático para un grafo arbitrario es un problema NP-completo. Por eso este proyecto emplea algoritmos heurísticos con soluciones válidas y eficientes, aunque sin garantizar la minimalidad.

### Coloreo de Vértices (*Vertex Coloring*)

El **coloreo de vértices** asigna una etiqueta (color) a cada vértice, de modo que no haya dos vértices adyacentes con el mismo color.

En el contexto de la programación de horarios, cada color es un bloque de tiempo distinto. La regla de coloreo garantiza  que ningún docente imparte dos clases simultáneamente y ningún grupo asiste a dos clases a la vez.


## 3. Resolución del problema

### 3.1 Modelado a partir de una tabla

La Tabla 1 del proyecto no es solo una lista de clases sino una especificación de conflictos. Cada fila describe una clase con dos atributos que determinan si puede coexistir con otra: el grupo al que pertenece y el docente que la imparte. Dos clases están en conflicto si comparten alguno de estos atributos.

**Tabla 1. Datos de Clases**

| Id_Clase | Materia | Grupo | Docente |
|---|---|---|---|
| A | Mate I | G1 | T1 |
| B | Mate II | G1 | T2 |
| C | Mate III | G1 | T3 |
| D | Física I | G2 | T1 |
| E | Física II | G2 | T2 |
| F | Física III | G2 | T3 |
| G | Prog I | G3 | T1 |
| H | Prog II | G3 | T2 |
| I | Prog III | G3 | T3 |

Mirando la tabla, A y B comparten el grupo G1: si se programan al mismo tiempo, los estudiantes de G1 tendrían dos clases simultáneas, lo que es imposible. A y D, en cambio, comparten el docente T1: si coinciden en horario, T1 estaría impartiendo dos clases a la vez. Estas dos situaciones — conflicto de grupo y conflicto de docente — son las únicas fuentes de restricción. Cualquier par de clases que no comparta ni grupo ni docente puede, en principio, programarse en el mismo bloque.


### 3.2 Representación de conflictos usando grafos

Como anteriormente hemos visto definido anteriormente, un grafo es un conjunto de vértices V y un conjunto de aristas E que los conectan y permite representar exactamente las relaciones que importan. Se crea un vértice por cada clase, formando el conjunto `V = {A, B, C, D, E, F, G, H, I}`, y se dibuja una arista entre dos clases si y solo si comparten grupo o docente:

```
Arista {u, v} existe  ⟺  Grupo(u) = Grupo(v)  O  Docente(u) = Docente(v)
```

Dos vértices conectados por una arista se llaman **adyacentes** e implica que dos clases adyacentes no pueden ocurrir en el mismo bloque de tiempo. Una clase con grado alto tiene muchos conflictos y es, por tanto, la más difícil de programar.

Aplicando la regla anterior a cada par de clases de la Tabla 1 se obtienen 18 aristas en total:

Grupo G1 genera el triángulo `{A,B}`, `{A,C}`, `{B,C}`. Grupo G2 genera `{D,E}`, `{D,F}`, `{E,F}`. Grupo G3 genera `{G,H}`, `{G,I}`, `{H,I}`. Docente T1 genera el triángulo `{A,D}`, `{A,G}`, `{D,G}`. Docente T2 genera `{B,E}`, `{B,H}`, `{E,H}`. Docente T3 genera `{C,F}`, `{C,I}`, `{F,I}`.

El resultado es un grafo donde cada clase conflictúa exactamente con 2 compañeros de grupo y 2 compañeros de docente, acumulando siempre 4 vecinos.

### 3.3 Construcción  de Aristas

Expandiendo la seccion anterior a través de la aplicación de la regla de conflicto a cada par de clases se obtienen las siguientes aristas:

**Clique del Grupo G1 — {A, B, C}:**
`{A,B}` — ambas G1 | `{A,C}` — ambas G1 | `{B,C}` — ambas G1

**Clique del Grupo G2 — {D, E, F}:**
`{D,E}` — ambas G2 | `{D,F}` — ambas G2 | `{E,F}` — ambas G2

**Clique del Grupo G3 — {G, H, I}:**
`{G,H}` — ambas G3 | `{G,I}` — ambas G3 | `{H,I}` — ambas G3

**Clique del Docente T1 — {A, D, G}:**
`{A,D}` — ambas T1 | `{A,G}` — ambas T1 | `{D,G}` — ambas T1

**Clique del Docente T2 — {B, E, H}:**
`{B,E}` — ambas T2 | `{B,H}` — ambas T2 | `{E,H}` — ambas T2

**Clique del Docente T3 — {C, F, I}:**
`{C,F}` — ambas T3 | `{C,I}` — ambas T3 | `{F,I}` — ambas T3

**Total: `|E| = 18` aristas.** Cada vértice tiene exactamente grado 4 (2 compañeros de grupo + 2 compañeros de docente).

### 3.4 Almacenamiento del grafo

Para que los algoritmos puedan recorrer el grafo, se necesita una estructura de datos concreta. La elección es una **lista de adyacencia**: para cada vértice `u`, se mantiene la lista `Adj[u]` con todos sus vecinos. El costo en memoria es de `Θ(V + E)`(alternativamente se puede utilizar una matriz de adjacencia pero ésta es menos eficiente).

Incorporando los procesos anteriores tenemos por resultado la siguiente lista:

| Clase | Adj[clase] | Razón |
|---|---|---|
| A | [B, C, D, G] | B,C comparten G1; D,G comparten T1 |
| B | [A, C, E, H] | A,C comparten G1; E,H comparten T2 |
| C | [A, B, F, I] | A,B comparten G1; F,I comparten T3 |
| D | [E, F, A, G] | E,F comparten G2; A,G comparten T1 |
| E | [D, F, B, H] | D,F comparten G2; B,H comparten T2 |
| F | [D, E, C, I] | D,E comparten G2; C,I comparten T3 |
| G | [H, I, A, D] | H,I comparten G3; A,D comparten T1 |
| H | [G, I, B, E] | G,I comparten G3; B,E comparten T2 |
| I | [G, H, C, F] | G,H comparten G3; C,F comparten T3 |

La lista también puede visualizarse como una **matriz de adyacencia** simétrica:

```
       A  B  C  D  E  F  G  H  I
  A  [ 0  1  1  1  0  0  1  0  0 ]
  B  [ 1  0  1  0  1  0  0  1  0 ]
  C  [ 1  1  0  0  0  1  0  0  1 ]
  D  [ 1  0  0  0  1  1  1  0  0 ]
  E  [ 0  1  0  1  0  1  0  1  0 ]
  F  [ 0  0  1  1  1  0  0  0  1 ]
  G  [ 1  0  0  1  0  0  0  1  1 ]
  H  [ 0  1  0  0  1  0  1  0  1 ]
  I  [ 0  0  1  0  0  1  1  1  0 ]
```

Con el grafo construido y representado, el problema de horarios podemos aplicar la técnica de coloreo de grafos.

## 4. Coloreo de Grafos

### 4.1 Justificación de la solución

El **coloreo de vértices** consiste en asignar una etiqueta o "color" a cada vértice de modo que no haya dos vértices adyacentes con el mismo color:

```
∀ {u, v} ∈ E  ⟹  color(u) ≠ color(v)
```

Si cada color representa un bloque de tiempo, entonces la regla de coloreo garantiza que ningún par de clases en conflicto comparta horario. No es necesario verificar cada restricción por separado  porque  una asignación de colores válida es un horario válido.

El objetivo de optimización también se traduce: el **número cromático** `χ(G)` es el número mínimo de colores necesarios para un coloreo válido, y corresponde directamente al menor número de bloques de tiempo requerido

Encontrar ese mínimo es la pregunta que este proyecto intenta responder. Considerando, hla implicación teórica definida con anterioridad: la determinación del número cromático para  un grafo cualquiera es un problema **NP-completo**, lo que significa que no se conoce ningún algoritmo de tiempo polinomial capaz de resolverlo de forma óptima en todos los casos posibles. Esto no hace el problema irresoluble — hace que la estrategia correcta sea combinar algoritmos heurísticos eficientes con argumentos teóricos que permitan acotar y en algunos casos determinar el óptimo exacto.

## 5. Aplicación del Algoritmo Greedy

El **algoritmo Greedy** para coloreo de grafos opera de la siguiente forma: procesar los vértices en un orden determinado y, para cada uno, asignar el color más pequeño disponible que no conflictúe con ninguno de sus vecinos ya coloreados. Nunca se reevalua una decisión tomada. La idea es que, tomando la mejor opción local en cada paso, se llegue a un resultado globalmente eficiente aunque no necesariamente óptimo.

```
procedimiento GREEDY_HORARIO(G, V_orden):
    asignacion ← {}

    para cada clase u en V_orden:
        # ¿qué bloques ya usaron mis vecinos?
        prohibidos ← { asignacion[v] | v ∈ G.Adj[u] y v ya asignado }

        # tomar el bloque más pequeño disponible
        bloque ← 1
        mientras bloque ∈ prohibidos:
            bloque ← bloque + 1

        asignacion[u] ← bloque

    retornar asignacion
```

El conjunto `prohibidos` recoge los bloques de los vecinos ya programados. El ciclo avanza desde el bloque 1 hasta encontrar uno libre. A lo largo de toda la ejecución, el total de verificaciones de vecinos suma `∑ deg(u) = 2|E| = 36` — cada arista se inspecciona exactamente dos veces, una desde cada extremo — lo que confirma una complejidad de `Θ(V + E)`.

### 5.1 Ejecución con el orden requerido: B, A, H, F, I, E, D, C, G

El orden específico de procesamiento determina el resultado. Al ejecutar el algoritmo con el orden indicado, cada paso consulta la lista de adyacencia del vértice actual y verifica qué bloques están prohibidos:

| Paso | Clase | Vecinos ya asignados | Bloques prohibidos | Bloque asignado |
|---|---|---|---|---|
| 1 | **B** | (ninguno) | {} | **Bloque 1** |
| 2 | **A** | B | {1} | **Bloque 2** |
| 3 | **H** | B | {1} | **Bloque 2** |
| 4 | **F** | (ninguno) | {} | **Bloque 1** |
| 5 | **I** | H, F | {2, 1} | **Bloque 3** |
| 6 | **E** | B, F, H | {1, 2} | **Bloque 3** |
| 7 | **D** | A, E, F | {2, 3, 1} | **Bloque 4** |
| 8 | **C** | A, B, F, I | {2, 1, 3} | **Bloque 4** |
| 9 | **G** | H, I, A, D | {2, 3, 4} | **Bloque 1** |

El resultado es una asignación válida de 4 bloques:

| Clase | A | B | C | D | E | F | G | H | I |
|---|---|---|---|---|---|---|---|---|---|
| Bloque | 2 | 1 | 4 | 4 | 3 | 1 | 1 | 2 | 3 |

Nótese el último paso: la clase G tiene cuatro vecinos ya asignados — H (Bloque 2), I (Bloque 3), A (Bloque 2) y D (Bloque 4) — pero el Bloque 1 no aparece en ninguno de ellos. El algoritmo lo toma, reutilizando un bloque existente en lugar de abrir uno nuevo. Esto ilustra la naturaleza oportunista del Greedy: siempre busca reciclar antes de crear.

Para confirmar que no hay conflictos, basta recorrer las 18 aristas y verificar que los extremos tienen bloques distintos. Por ejemplo: `{A,B}` → Bloques 2 y 1 ✓; `{A,D}` → Bloques 2 y 4 ✓; `{D,G}` → Bloques 4 y 1 ✓. Las 18 aristas pasan la verificación.

## 6. Comparación con el Algoritmo de Welsh-Powell

### 6.1 Abordando el orden en el grafo

Como anteriormente ha sido expuesto, el algoritmo greedy es suceptible a encontrar soluciones según el orden que se solicite. Si se deja un vértice muy conflictivo para el final (ya que han sido asignados muchos bloques) puede verse forzado a tomar un bloque nuevo cuando podría haber reciclado uno existente en otro orden. El algoritmo de Welsh-Powell resuelve esto con un paso previo al coloreo: ordenar los vértices de mayor a menor grado antes de aplicar el mismo procedimiento del algoritmo Greedy.

La intuición es que las clases más conflictivas (mayor grado) deben colorearse primero, mientras hay más opciones disponibles. Las clases menos conflictivas se procesan al final y pueden reutilizar bloques con más libertad.

```
procedimiento WELSH_POWELL(G):
    vertices_ordenados ← ordenar V por deg(u) de forma decreciente
    retornar GREEDY_HORARIO(G, vertices_ordenados)
```

El único costo adicional es el ordenamiento: `O(V log V)`. El coloreo en sí sigue siendo `Θ(V + E)`. La complejidad total es `Θ(V log V + E)`.

### 6.2 Ejecución y resultado por Welsh-Powwll

Como todos los vértices tienen grado 4, el ordenamiento por grado no distingue entre ellos. Resolviendo el empate alfabéticamente, el orden resultante es A, B, C, D, E, F, G, H, I:

| Paso | Clase | Vecinos ya asignados | Bloques prohibidos | Bloque asignado |
|---|---|---|---|---|
| 1 | **A** | (ninguno) | {} | **Bloque 1** |
| 2 | **B** | A | {1} | **Bloque 2** |
| 3 | **C** | A, B | {1, 2} | **Bloque 3** |
| 4 | **D** | A | {1} | **Bloque 2** |
| 5 | **E** | B, D | {2} | **Bloque 1** |
| 6 | **F** | C, D, E | {1, 2, 3} | **Bloque 4** |
| 7 | **G** | A, D | {1, 2} | **Bloque 3** |
| 8 | **H** | B, E, G | {1, 2, 3} | **Bloque 4** |
| 9 | **I** | C, F, G, H | {3, 4} | **Bloque 1** |

Welsh-Powell también obtiene 4 bloques. Esto no es una coincidencia, sino que al ser simétrico el grafo provisto en el ejemplo, el algoritmo no ofrece ninguna ventaja porque ningun vértice tiene mayor restriccion que el resto.

### 6.3 Comparación directa

| Propiedad | Greedy (orden requerido) | Welsh-Powell (orden por grado) |
|---|---|---|
| Orden de procesamiento | B, A, H, F, I, E, D, C, G | A, B, C, D, E, F, G, H, I |
| Complejidad temporal | `Θ(V + E)` | `Θ(V log V + E)` |
| Bloques utilizados | 4 | 4 |
| Garantía de optimalidad | Ninguna | Ninguna |
| Sensibilidad al orden | Alta | Menor (orden principiado) |
| Ventaja sobre Greedy simple | — | En grafos irregulares, generalmente menos colores |

Ambos algoritmos encontraron 4 bloques, pero la el mínimo teórico establecido anteriormente  fue 3. ¿Significa esto que ambos algoritmos fallaron en alcanzar el óptimo? Esta pregunta lleva al punto clave del problema.

## 7. Alcanzando el mínimo

### 7.1 Mínimos locales en un problema NP-completo

Que el coloreo de grafos sea **NP-completo** tiene una consecuencia práctica que algoritmos heurísticos puedan quedar atrapados en un mínimo local. 

Para demostrar que  el valor mínimo es mayor o igual a 3 se necesita mostrar tanto que no puede hacerse con menos de 3 bloques. Basta exhibir un coloreo válido con 3 colores. La siguiente asignación lo logra (detallada en el apéndice A):

```
Bloque 1: A, E, I
Bloque 2: B, F, G
Bloque 3: C, D, H
```

Verificación sobre las 18 aristas del grafo:
```
{A,B}: 1 vs 2 ✓   {A,C}: 1 vs 3 ✓   {B,C}: 2 vs 3 ✓
{D,E}: 3 vs 1 ✓   {D,F}: 3 vs 2 ✓   {E,F}: 1 vs 2 ✓
{G,H}: 2 vs 3 ✓   {G,I}: 2 vs 1 ✓   {H,I}: 3 vs 1 ✓
{A,D}: 1 vs 3 ✓   {A,G}: 1 vs 2 ✓   {D,G}: 3 vs 2 ✓
{B,E}: 2 vs 1 ✓   {B,H}: 2 vs 3 ✓   {E,H}: 1 vs 3 ✓
{C,F}: 3 vs 2 ✓   {C,I}: 3 vs 1 ✓   {F,I}: 2 vs 1 ✓
```

Todas las aristas satisfechas. Esto prueba que `χ(G) ≤ 3`. Combinando con `χ(G) ≥ 3`, se concluye que **`χ(G) = 3` exactamente**: el horario mínimo teórico requiere 3 bloques y se puede construir.


## 8. Análisis de Complejidad

### 8.1 Complejidad del Greedy

La complejidad de `Θ(V + E)` surge de dos operaciones: iterar sobre los `V` vértices una vez, y para cada uno, recorrer sus vecinos en `Adj[u]`. El total de todas las inspecciones de vecinos está acotado por el **Lema de los Apretones de Manos**:

```
∑_{u ∈ V} deg(u) = 2|E|
```

Cada arista `{u, v}` se inspecciona exactamente dos veces — una desde `u` y otra desde `v`. Para este grafo: `9 × 4 = 36 = 2 × 18`. El trabajo total es `Θ(V) + Θ(E) = Θ(V + E)`.

| Fase | Operación | Costo |
|---|---|---|
| Iteración de vértices | Visitar cada `u ∈ V` una vez | `Θ(V)` |
| Recorrido de vecinos | Suma total de `\|Adj[u]\|` sobre todos los vértices | `Θ(2E) = Θ(E)` |
| Búsqueda de bloque libre | Ciclo `mientras`, acotado por los recorridos de vecinos | `O(E)` |
| **Total** | | **`Θ(V + E)`** |

La complejidad espacial es también `Θ(V + E)` — la lista de adyacencia más el diccionario de asignaciones.

### 8.2 Complejidad de Welsh-Powell

Welsh-Powell agrega un paso de ordenamiento antes del coloreo. Ordenar `V` vértices cuesta `Θ(V log V)`. El coloreo posterior es idéntico al Greedy. La complejidad total es `Θ(V log V + E)`.

### 8.3 Por qué la eficiencia importa

La diferencia entre `Θ(V + E)` y un enfoque de fuerza bruta no es de grados, sino de órdenes de magnitud. La siguiente tabla muestra la diferencia teórica evaluando con diferente numero de aristas:

| Clases (V) | Aristas (E) | Greedy `V+E` | Welsh-Powell `V log V + E` | Fuerza bruta `4^V` |
|---|---|---|---|---|
| 9 (este proyecto) | 18 | 27 | ~54 | 262,144 |
| 50 | ~300 | 350 | ~481 | ~10³⁰ |
| 100 | ~1,000 | 1,100 | ~1,764 | ~10⁶⁰ |
| 500 | ~10,000 | 10,500 | ~14,483 | ~10³⁰⁰ |
| 1,000 | ~50,000 | 51,000 | ~60,000 | ~10⁶⁰⁰ |

Un horario con 50 clases resolvería en microsegundos con Greedy o Welsh-Powell. Con fuerza bruta, las operaciones estan en órdenes de tamaño fuera de lo imaginable o visible. Las heurísticas implican, entonces, una herramienta poderosa para éste tipo de problemas donde buscamos soluciones aceptablemente eficientes en un mínimo local pero alcanzables en procesiamiento,


## 9. Cambiando el probema con restricciones de docentes

### 9.1 Del coloreo estándar al coloreo por listas

Hasta ahora se asumió que cualquier clase puede ir en cualquier bloque. Cuando los docentes tienen restricciones de disponibilidad, esto cambia: cada clase hereda las restricciones de su docente y solo puede programarse en ciertos bloques. El modelo se extiende al **coloreo por listas**, donde cada vértice `u` tiene su propio conjunto permitido `L(u)` y la asignación debe satisfacer:

```
asignacion(u) ∈ L(u)   Y   ∀ {u,v} ∈ E: asignacion(u) ≠ asignacion(v)
```

Este es un problema más difícil que el coloreo estándar puesto que el número mínimo de bloques que siempre garantiza una solución para cualquier configuración de listas se llama **número cromático de lista** `χₗ(G)`, y satisface `χₗ(G) ≥ χ(G)`.

El algoritmo se modifica mínimamente: en lugar de buscar el menor entero positivo disponible, se busca el menor entero disponible *dentro del conjunto permitido* del docente:

```
procedimiento HORARIO_COLOREO_LISTAS(G, V_orden, L):
    asignacion ← {}
    para cada clase u en V_orden:
        prohibidos ← { asignacion[v] | v ∈ G.Adj[u] y v ya asignado }
        disponibles ← L(u) \ prohibidos

        si disponibles está vacío:
            reportar INVIABLE y detener
        asignacion[u] ← min(disponibles)

    retornar asignacion
```

### 9.2 Las restricciones del proyecto y su análisis

El proyecto especifica la siguiente disponibilidad:

```
T1: NO disponible en B1 y B4  →  L(A) = L(D) = L(G) = {B2, B3}
T2: NO disponible en B2 y B3  →  L(B) = L(E) = L(H) = {B1, B4}
T3: NO disponible en B1 y B3  →  L(C) = L(F) = L(I) = {B2, B4}
```

Aplicando el algoritmo con el orden requerido B → A → H → F → I → E → D → C → G:

| Paso | Clase | Docente | L(u) | Bloques prohibidos por vecinos | Disponibles | Asignado |
|---|---|---|---|---|---|---|
| 1 | **B** | T2 | {B1, B4} | {} | {B1, B4} | **B1** |
| 2 | **A** | T1 | {B2, B3} | {} | {B2, B3} | **B2** |
| 3 | **H** | T2 | {B1, B4} | B1 (vecino B) | {B4} | **B4** |
| 4 | **F** | T3 | {B2, B4} | {} | {B2, B4} | **B2** |
| 5 | **I** | T3 | {B2, B4} | B4 (H), B2 (F) | **{}** | **INVIABLE** |

La clase I solo puede ir en `{B2, B4}`, pero sus vecinos H y F ya ocuparon ambos bloques. El algoritmo no encuentra salida.

La pregunta natural es: ¿este fallo depende del orden de procesamiento, o el problema es imposible bajo cualquier asignación?

### 9.3 La inviabilidad es estructural, no algorítmica

Para responder, hay que ignorar el algoritmo y mirar directamente los subconjuntos y las listas. Las clases A, D y G forman un clique — todas se conflictúan entre sí — y todas están asignadas al docente T1, cuyo conjunto permitido es `{B2, B3}`. Para colorear ese clique de tamaño 3 se necesitan 3 colores distintos, pero solo hay 2 disponibles. Esta situación no tiene solución posible.

Existe un teorema formal que plantea ésta constrain teórica, el **teorema de Hall para coloreo por listas**, la cual establece que un clique de tamaño `k` puede colorearse por listas solo si la unión de los conjuntos permitidos de sus vértices contiene al menos `k` elementos distintos

Para el clique T1 `{A, D, G}`: `L(A) ∪ L(D) ∪ L(G) = {B2, B3}`, con solo 2 elementos pero `|C| = 3`. La condición se viola. El mismo problema ocurre independientemente con los cliques T2 y T3.

La inviabilidad es una imposibilidad matemática que ningún método puede superar. El sistema de restricciones tal como está especificado hace que no exista ningún horario válido, sin excepción.


## 10. Implementación Computacional

La implementación en Python materializa todo el razonamiento anterior: construye el grafo de conflictos directamente desde los datos de la tabla, aplica ambos algoritmos de coloreo, valida que no existan conflictos en el resultado, maneja las restricciones de disponibilidad docente y genera una visualización gráfica de los grafos coloreados.

### 10.1 Dependencias utilizadas en la implementación

Las siguientes dependencias fueron empleadas para la implementación de la solución:
`networkx` gestiona las estructuras de grafo y el layout visual. `matplotlib` renderiza la salida gráfica. `scipy` es usado internamente por NetworkX para el algoritmo de posicionamiento de nodos.

### 10.2 Código completo
Disponible en el repositorio: ####

### 10.3 Ejecución y salida esperada


La consola muestra los grafos de adyacencia, los resultados paso a paso de ambos algoritmos con su respectiva validación, el coloreo óptimo de 3 bloques y el análisis de inviabilidad bajo las restricciones de disponibilidad. La imagen generada `horario_coloreo_grafos.png` presenta tres paneles lado a lado: el coloreo Greedy, el de Welsh-Powell y el óptimo de 3 bloques, con nodos coloreados por bloque asignado y leyenda de bloques.

La implementación es general puesto que emplea variables constantes para modelar una asignación arbitraria cualquiera.

![Horario](./horario_coloreo_grafos.png)

## 11. Conclusión

Este proyecto resuelve el problema de asignación de horarios académicos siguiendo una trayectoria lógica que va de los datos al modelo, del modelo al algoritmo, y del algoritmo al análisis teórico.

La Tabla 1 se tradujo en un grafo de conflictos con 9 vértices y 18 aristas, representado mediante lista de adyacencia para eficiencia computacional. El Greedy aplicado en el orden B→A→H→F→I→E→D→C→G produjo un horario válido de 4 bloques al igual que el algoritmo por Wells-Powell.

El análisis teórico reveló que ambos algoritmos encontraron mínimos locales, mientras que se provee una explicación respecto al mínimo teórico, considerando el caso particular del ejemplo provisto.

La extensión al coloreo por listas mostró que las restricciones de disponibilidad docente especificadas en el proyecto son inviables al violar una restricción matemática fundamental.

## Apéndice A: Asignaciíon diagonal

El coloreo con 3 etiquetas `{A,E,I → B1}, {B,F,G → B2}, {C,D,H → B3}` surge por la forma particular del grafo.

La Tabla 1 tiene una estructura muy particular: 3 grupos forman las filas y 3 docentes forman las columnas de una cuadrícula 3×3, con exactamente una clase en cada celda. El patrón es conocido como _Torre de Ajedrez_.

```
         T1    T2    T3
  G1  |  A  |  B  |  C  |
  G2  |  D  |  E  |  F  |
  G3  |  G  |  H  |  I  |
```

El grafo de torre es un objeto estudiado en la teoría de grafos. Forma parte de la literatura de grafos y su tamaño es nxm.

Un **cuadrado latino** de orden `n` es una cuadrícula `n × n` donde `n` símbolos distintos aparecen exactamente una vez en cada fila y exactamente una vez en cada columna. Colorear el grafo de torre `Kₙ □ Kₙ` con `n` colores es exactamente lo mismo que construir un cuadrado latino de orden `n`: la regla de coloreo prohíbe repetir colores en la misma fila (mismo grupo) y en la misma columna (mismo docente), que es precisamente la definición de cuadrado latino.

El cuadrado latino más simple para n=3 es el cuadrado cíclico, donde cada fila es un desplazamiento de la anterior:

```
         T1    T2    T3
  G1  |  1  |  2  |  3  |
  G2  |  3  |  1  |  2  |
  G3  |  2  |  3  |  1  |
```

Leyendo las asignaciones: el símbolo 1 aparece en las celdas (G1,T1), (G2,T2) y (G3,T3), que corresponden a las clases A, E e I. El símbolo 2 aparece en (G1,T2), (G2,T3) y (G3,T1), que son B, F y G. El símbolo 3 aparece en (G1,T3), (G2,T1) y (G3,T2), que son C, D y H. El patrón "diagonal" es simplemente la consecuencia visual del desplazamiento cíclico entre filas.

Este razonamiento funciona porque la Tabla 1 tiene una estructura perfectamente balanceada. No se generaliza a instancias arbitrarias: si un grupo tuviera más clases que otro, o si un docente impartiera más clases que otro, la cuadrícula no sería cuadrada, el grafo no sería equivalente al grafo de torre, y el truco del cuadrado latino no aplicaría. Para cualquier coordinación real con estructura irregular, el número cromático debe calcularse algorítmicamente. El valor de este razonamiento en el contexto del proyecto es doble: permite demostrar que `χ(G) = 3` de forma rigurosa, y permite entender por qué los algoritmos heurísticos se quedan cortos en instancias con esta simetría.

| Concepto | Rol en la derivación |
|---|---|
| Cuadrícula 3×3 de la Tabla 1 | Expone la estructura de fila/columna |
| Grafo de torre `K₃ □ K₃` | Identifica el grafo como objeto matemático conocido |
| Resultado `χ(Kₙ □ Kₙ) = n` | Prueba `χ(G) = 3` sin búsqueda exhaustiva |
| Cuadrado latino de orden 3 | Proporciona el 3-coloreo explícito |
| Desplazamiento cíclico | Genera el patrón "diagonal" observado |
