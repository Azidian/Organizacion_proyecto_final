# Documentación de Clases — Proyecto MAZE INTELLIGENCE

## Tabla de Contenido

1. [Visión general del proyecto](#1-visión-general-del-proyecto)
2. [Arquitectura y relaciones entre clases](#2-arquitectura-y-relaciones-entre-clases)
3. [Clase `Main`](#3-clase-main)
4. [Clase `Game`](#4-clase-game)
   - [Máquina de estados](#41-máquina-de-estados)
   - [Campos de instancia](#42-campos-de-instancia)
   - [Métodos](#43-métodos)
5. [Clase `Maze`](#5-clase-maze)
   - [Representación interna](#51-representación-interna)
   - [Construcción de niveles](#52-construcción-de-niveles)
   - [Métodos de acceso y renderizado](#53-métodos-de-acceso-y-renderizado)
6. [Clase `Player`](#6-clase-player)
   - [Sistema de movimiento con throttling](#61-sistema-de-movimiento-con-throttling)
   - [Sprites por nivel](#62-sprites-por-nivel)
7. [Clase `Enemy`](#7-clase-enemy)
   - [Inteligencia artificial con BFS](#71-inteligencia-artificial-con-bfs)
   - [Tipos de comportamiento](#72-tipos-de-comportamiento)
   - [Gestión de memoria estática](#73-gestión-de-memoria-estática)
   - [Sprites por nivel](#74-sprites-por-nivel)
8. [Clase `UI`](#8-clase-ui)
   - [Pantallas del juego](#81-pantallas-del-juego)
   - [Helpers de dibujo](#82-helpers-de-dibujo)
9. [Clase `Random`](#9-clase-random)
10. [Resumen de interacciones entre clases](#10-resumen-de-interacciones-entre-clases)

---

## 1. Visión general del proyecto

**MAZE INTELLIGENCE** es un juego de laberintos desarrollado en lenguaje **Jack** para la plataforma **Hack** (arquitectura nand2tetris). El jugador controla un personaje que debe escapar por la salida de un laberinto evitando ser atrapado por uno o dos enemigos controlados por una inteligencia artificial de búsqueda en anchura (BFS).

El juego consta de **3 niveles** de dificultad creciente, cada uno con una temática de personajes distinta:

| Nivel | Personaje del jugador | Enemigo(s) | Dificultad |
|---|---|---|---|
| 1 | Ratón | Gato | Baja |
| 2 | Ardilla | Perro (sprite de Gato) | Media |
| 3 | Pájaro | Serpiente + segundo enemigo | Alta |

La pantalla de la plataforma Hack mide **512 × 256 píxeles**. El área de juego ocupa una cuadrícula de **17 columnas × 11 filas**, donde cada celda mide **16 × 16 px**, resultando en un campo de juego de 272 × 176 px.

---

## 2. Arquitectura y relaciones entre clases

El proyecto está organizado en **7 clases** con responsabilidades bien separadas:

```
Proyecto_final/
├── README.md                           ← Documentación principal (este archivo)
└── OrganizacionComputadores/           ← Submódulo y repositorio principal
    ├── CHANGELOG.md                    ← Registro de cambios
    ├── CONTRIBUTORS.md                 ← Lista de contribuidores
    ├── LICENSE.md                      ← Licencia del proyecto
    └── Proyecto_final/
        ├── docs/                       ← Documentación extendida
        │   ├── User_Guide.md           ← Guía completa del jugador
        │   └── Clases.md               ← Documentación técnica de clases y diagramas
        └── src/                        ← Código fuente Jack
            ├── Main.jack               ← Punto de entrada del programa
            ├── Game.jack               ← Máquina de estados y coordinador principal
            ├── Maze.jack               ← Datos del laberinto y renderizado de celdas
            ├── Player.jack             ← Input de teclado, movimiento y sprite del jugador
            ├── Enemy.jack              ← IA BFS, control de velocidad y sprite del enemigo
            ├── UI.jack                 ← Todas las pantallas, menús y títulos en pixel art
            └── Random.jack             ← Generador de números pseudoaleatorios
```

**Principios de diseño:**

- `Game` es el único orquestador: posee referencias a todos los demás objetos y coordina su ciclo de vida (creación, actualización, destrucción).
- `Maze` es la fuente de verdad del mapa. `Player` y `Enemy` le consultan si una celda es transitable antes de moverse, y le piden que "borre" su sprite anterior al redibujar.
- `UI` es completamente estática (no hay instancias). Contiene únicamente `function`, no `method`.
- `Random` es también estática y se puede usar desde cualquier punto sin instanciar nada.
- La memoria se gestiona manualmente: cada clase tiene un método `dispose()` que llama a `Memory.deAlloc(this)`.

---

## 3. Clase `Main`

**Archivo:** `Main.jack`

Punto de entrada del programa. Es deliberadamente mínima: su única responsabilidad es arrancar el juego y garantizar que la memoria quede liberada al terminar.

### Método

#### `function void main()`

```
1. Crea un objeto Game  →  let game = Game.new()
2. Ejecuta el juego     →  do game.run()
3. Libera la memoria    →  do game.dispose()
```

No contiene ninguna lógica propia. Toda la orquestación está delegada en `Game`. Esta separación asegura que siempre se llame a `dispose()`, incluso si `run()` termina antes de lo esperado.

### Video explicativo:
 **https://youtu.be/wKzBs_aFkG8?si=7HzbuFGRPwKN_xgo**  


---

## 4. Clase `Game`

**Archivo:** `Game.jack`

Controlador principal del juego. Implementa una **máquina de estados finitos** con 8 estados que gestiona el ciclo de vida completo: menús, carga de nivel, bucle de juego, pausa y pantallas de resultado.

### 4.1 Máquina de estados

Los estados se representan como enteros almacenados en campos estáticos (equivalentes a constantes):

| Constante | Valor | Descripción |
|---|---|---|
| `STATE_START` | 0 | Pantalla de título inicial |
| `STATE_LEVEL` | 1 | Pantalla de introducción de nivel |
| `STATE_PLAYING` | 2 | Bucle de juego activo |
| `STATE_PAUSED` | 3 | Juego pausado |
| `STATE_WIN` | 4 | Nivel completado con éxito |
| `STATE_LOSE` | 5 | Game Over (el enemigo atrapó al jugador) |
| `STATE_VICTORY` | 6 | Victoria total (los 3 niveles completados) |
| `STATE_LEVEL_LIST` | 7 | Pantalla de selección/lista de niveles |

El flujo normal de una partida completa es:

```
START → LEVEL_LIST → LEVEL(1) → PLAYING → WIN
                              → LEVEL(2) → PLAYING → WIN
                              → LEVEL(3) → PLAYING → VICTORY
                                         → LOSE → (reintentar o salir)
```

### 4.2 Campos de instancia

| Campo | Tipo | Descripción |
|---|---|---|
| `maze` | `Maze` | Referencia al laberinto activo |
| `player` | `Player` | Referencia al jugador |
| `enemy1` | `Enemy` | Enemigo principal |
| `enemy2` | `Enemy` | Segundo enemigo (solo nivel 3) |
| `state` | `int` | Estado actual de la máquina |
| `currentLevel` | `int` | Nivel en curso (1, 2 ó 3) |
| `exitRow`, `exitCol` | `int` | Coordenadas de la celda de salida en la cuadrícula |
| `hasSecondEnemy` | `boolean` | Indica si hay un segundo enemigo activo |
| `attempts` | `int` | Contador de reintentos del nivel (límite: 1) |

### 4.3 Métodos

#### `constructor Game new()`
Inicializa todas las constantes de estado como campos estáticos. Establece `currentLevel = 1`, `state = STATE_START` y todos los objetos (`maze`, `player`, `enemy1`, `enemy2`) a `null`. No asigna memoria para los objetos del nivel todavía; eso ocurre en `loadLevel`.

---

#### `method void run()`
El **bucle principal del juego**. Itera indefinidamente mientras los manejadores de estado devuelvan `true`. En cada iteración evalúa `state` y llama al manejador correspondiente:

```
while (continuar) {
    if state = STATE_START      → handleStart()
    if state = STATE_LEVEL_LIST → handleLevelList()
    if state = STATE_LEVEL      → handleLevelIntro()
    if state = STATE_PLAYING    → playTick()
    if state = STATE_PAUSED     → handlePause()
    if state = STATE_WIN        → handleLevelWin()
    if state = STATE_LOSE       → handleGameOver()
    if state = STATE_VICTORY    → handleVictory()
}
```

Los manejadores que pueden terminar el programa (`handleLevelWin`, `handleGameOver`, `handleVictory`) retornan `boolean`; cuando devuelven `false`, el bucle se interrumpe.

---

#### `method void handleStart()`
Muestra la pantalla de título llamando a `UI.drawStart()` y luego transiciona el estado a `STATE_LEVEL_LIST`.

---

#### `method void handleLevelList()`
Muestra la pantalla con la lista de los tres niveles disponibles (`UI.drawLevelList()`). Reinicia `currentLevel` a 1 y el contador de intentos a 0, carga el nivel 1 mediante `loadLevel(1)` y transiciona a `STATE_LEVEL`.

---

#### `method void handleLevelIntro()`
Muestra la pantalla de introducción al nivel actual con `UI.drawLevelIntro(currentLevel)`. Luego dibuja el estado inicial del tablero de juego con `drawGameScreen()` y transiciona a `STATE_PLAYING`.

---

#### `method void handlePause()`
Muestra la pantalla de pausa con `UI.drawPause(currentLevel)`. Cuando el jugador reanuda, redibuja el tablero de juego con `drawGameScreen()` y vuelve a `STATE_PLAYING`.

---

#### `method boolean handleLevelWin()`
Muestra la pantalla de nivel superado (`UI.drawLevelWin(currentLevel)`). Espera la elección del jugador:
- Si presiona `[Q]`: retorna `false`, terminando el programa.
- Si presiona `[ESPACIO]`: incrementa `currentLevel`, llama a `loadLevel(currentLevel)` y transiciona a `STATE_LEVEL`. Retorna `true`.

---

#### `method boolean handleGameOver()`
Muestra la pantalla de Game Over (`UI.drawGameOver(currentLevel)`) e incrementa `attempts`. Si `attempts > 1` (se agotaron los reintentos), fuerza la salida retornando `false`. De lo contrario, espera:
- Si presiona `[R]`: recarga el nivel actual con `loadLevel(currentLevel)` y transiciona a `STATE_LEVEL`.
- Si presiona `[Q]`: retorna `false`.

---

#### `method boolean handleVictory()`
Muestra la pantalla de victoria total (`UI.drawVictory()`). Siempre retorna `false`, terminando el bucle principal. No ofrece opción de reinicio para garantizar la liberación correcta de toda la memoria asignada durante la sesión.

---

#### `method void loadLevel(int lvl)`
Responsable de preparar todos los objetos necesarios para un nivel. Su lógica es:

1. **Libera** todos los objetos del nivel anterior llamando a sus destructores (si no son `null`): `maze.dispose()`, `player.dispose()`, `enemy1.dispose()`, `enemy2.dispose()`.
2. **Crea** nuevos objetos con los parámetros específicos del nivel:

| Nivel | Posición jugador | Posición enemy1 | Posición enemy2 | Intervalo enemy1 | Tipo enemy1 |
|---|---|---|---|---|---|
| 1 | (1, 1) | (9, 15) | — | 12 ticks | Directo (1) |
| 2 | (1, 1) | (9, 15) | — | 8 ticks | Directo (1) |
| 3 | (1, 1) | (9, 15) | (5, 8) | 6 ticks | Directo (1) |

3. En el nivel 3 activa `hasSecondEnemy = true` y crea `enemy2`.
4. Define `exitRow` y `exitCol` (la celda de salida del laberinto).

---

#### `method void playTick()`
Ejecuta **un fotograma de juego**. Se llama una vez por iteración del bucle mientras el estado es `STATE_PLAYING`. Su secuencia interna es:

1. Detecta la tecla `[P]` → transiciona a `STATE_PAUSED` y retorna.
2. Llama a `player.handleInput()` para procesar el movimiento del jugador.
3. Llama a `enemy1.update(playerRow, playerCol, 0, 0)`.
4. Si `hasSecondEnemy`, llama también a `enemy2.update(...)`.
5. Redibuja la celda de salida (`drawExit()`), el jugador (`player.draw()`) y los enemigos (`enemy1.draw()`, `enemy2.draw()`).
6. **Detección de colisión jugador-enemigo:** si las posiciones coinciden → `state = STATE_LOSE`.
7. **Detección de llegada a la salida:** si `player.getRow() = exitRow` y `player.getCol() = exitCol` → `state = STATE_WIN` (o `STATE_VICTORY` si `currentLevel = 3`).
8. Llama a `Sys.wait(16)` para mantener aproximadamente 60 fotogramas por segundo.

---

#### `method void drawGameScreen()`
Dibuja el estado visual completo del tablero. Se usa al comenzar un nivel o al salir de la pantalla de pausa. Pasos:
1. Limpia la pantalla.
2. Llama a `maze.draw()` para renderizar toda la cuadrícula.
3. Llama a `drawExit()`.
4. Llama a `player.draw()` y a `enemy1.draw()` (y `enemy2.draw()` si aplica).

---

#### `method void drawExit()`
Dibuja la celda de salida como un rectángulo blanco (pasillo) con una **X** formada por dos líneas diagonales negras cruzadas, para que sea fácilmente identificable por el jugador.

---

#### `method void dispose()`
Libera en orden todos los objetos vivos: `maze`, `player`, `enemy1`, `enemy2`, y finalmente el propio objeto `Game` con `Memory.deAlloc(this)`. Se llama desde `Main.main()` al terminar la ejecución.

### Video explicativo:
**https://youtu.be/OSD-7_3vpx4** 

---

## 5. Clase `Maze`

**Archivo:** `Maze.jack`

Representa el laberinto como una **matriz bidimensional plana** almacenada en un `Array` unidimensional (almacenamiento fila-mayor). Se encarga de cargar el diseño del nivel, proveer acceso a las celdas y renderizar el laberinto en pantalla.

### 5.1 Representación interna

#### Constantes estáticas

| Constante | Valor | Descripción |
|---|---|---|
| `ROWS` | 11 | Número de filas de la cuadrícula |
| `COLS` | 17 | Número de columnas de la cuadrícula |
| `CELL_SIZE` | 16 | Tamaño de cada celda en píxeles |

El array `map` tiene `ROWS × COLS = 187` enteros. La celda en la posición `(row, col)` se almacena en el índice `row * COLS + col`. Cada celda vale:

- `1` → **muro** (negro, bloqueante)
- `0` → **pasillo** (blanco, transitable)

#### Campos de instancia

| Campo | Tipo | Descripción |
|---|---|---|
| `map` | `Array` | Array plano de 187 enteros con el mapa |
| `level` | `int` | Nivel activo, determina qué layout se carga |

#### `constructor Maze new(int lvl)`
1. Asigna las constantes estáticas a variables locales.
2. Crea el array `map` con `Array.new(187)`.
3. Inicializa todas las celdas a `1` (todo muros).
4. Según el nivel, llama a `buildLevel1`, `buildLevel2` o `buildLevel3` para "abrir" los pasillos.

### 5.2 Construcción de niveles

Los tres layouts se definen mediante llamadas a `setRow`, que escribe los 17 valores de cada fila del mapa. Jack no soporta literales de array, por lo que cada valor de columna se pasa explícitamente como parámetro.

#### `function void setRow(Array m, int cols, int row, int c0 … c16)`
Función auxiliar. Calcula el índice base como `row * cols` y asigna individualmente los 17 valores de la fila. Requiere los 17 parámetros explícitamente porque Jack no soporta arrays literales en código.

#### `function void buildLevel1(Array m, int cols)`
Genera un laberinto de **complejidad baja** con pasillos amplios y pocas paredes internas. El recorrido es relativamente directo. Temática: Ratón escapando del Gato.

#### `function void buildLevel2(Array m, int cols)`
Genera un laberinto de **complejidad media** con caminos más sinuosos, algunos pasillos estrechos y bifurcaciones que obligan a elegir rutas. Temática: Ardilla escapando del Perro.

#### `function void buildLevel3(Array m, int cols)`
Genera un laberinto de **complejidad máxima**, muy denso, con pasillos de 1 celda de ancho y pocas bifurcaciones. Además, hay dos enemigos activos en este nivel. Temática: Pájaro escapando de la Serpiente.

### 5.3 Métodos de acceso y renderizado

#### `method int getCell(int row, int col)`
Retorna el valor entero de la celda `(row, col)` directamente del array. **No** realiza validación de límites; es responsabilidad del llamador asegurarse de que las coordenadas son válidas.

#### `method boolean isPassable(int row, int col)`
Versión segura del acceso. Verifica primero que las coordenadas estén dentro del rango `[0, ROWS-1]` × `[0, COLS-1]`. Retorna `true` únicamente si la celda vale `0` (pasillo). Este es el **único método que usan `Player` y `Enemy`** para validar si pueden moverse a una celda.

#### `method int getRows()` / `method int getCols()` / `method int getCellSize()` / `method int getLevel()`
Getters simples que exponen las constantes y el nivel activo. `Enemy` usa `getRows()` y `getCols()` para calcular los límites del campo BFS.

#### `method void draw()`
Recorre **toda la cuadrícula** y dibuja cada celda con su color correcto: negro (`setColor(true)`) para muros, blanco (`setColor(false)`) para pasillos. Se llama una sola vez al iniciar o reanudar un nivel, ya que es una operación costosa.

#### `method void drawCell(int row, int col)`
Redibuja **únicamente** la celda indicada con su color correcto. Es usada por `Player` y `Enemy` para "borrar" su sprite de la posición anterior antes de moverse: llaman `maze.drawCell(row, col)` con su posición actual, luego actualizan sus coordenadas y se dibujan en la nueva posición.

#### `method void dispose()`
Libera el array `map` con `map.dispose()` y luego el propio objeto con `Memory.deAlloc(this)`.


#### Vídeo Explicativo:
**https://youtu.be/wZLwARiHJT4**


---

## 6. Clase `Player`

**Archivo:** `Player.jack`

Representa al personaje controlado por el usuario. Gestiona la lectura del teclado, el movimiento con control de velocidad progresivo por nivel y el renderizado de tres sprites distintos.

### 6.1 Sistema de movimiento con throttling

Jack corre en un bucle sin interrupciones y `Keyboard.keyPressed()` detecta si una tecla está presionada en ese instante. Sin un control de velocidad, el jugador se movería decenas de celdas por segundo, haciendo el juego injugable.

La solución implementada es un **temporizador de espera** (`moveTimer` / `moveDelay`): solo se procesa el movimiento cuando `moveTimer` alcanza el umbral `moveDelay`, tras lo cual se reinicia a 0.

#### Campos de instancia

| Campo | Tipo | Descripción |
|---|---|---|
| `row`, `col` | `int` | Posición actual en la cuadrícula |
| `maze` | `Maze` | Referencia al laberinto para validar movimientos |
| `cellSize` | `int` | Tamaño de celda en px, obtenido del laberinto |
| `level` | `int` | Nivel actual, determina qué sprite se dibuja |
| `moveDelay` | `int` | Ticks mínimos entre movimientos consecutivos |
| `moveTimer` | `int` | Contador de ticks desde el último movimiento |

#### `constructor Player new(Maze theMaze, int startRow, int startCol, int lvl)`
Asigna la posición inicial, la referencia al laberinto y el nivel. Configura `moveDelay` en función de la dificultad:

| Nivel | `moveDelay` | Velocidad aproximada |
|---|---|---|
| 1 | 9 ticks | ~144 ms por celda |
| 2 | 6 ticks | ~96 ms por celda |
| 3 | 4 ticks | ~64 ms por celda |

#### `method int getRow()` / `method int getCol()`
Getters que exponen la posición actual. `Game` los usa en cada tick para comparar con las posiciones de los enemigos y la celda de salida.

#### `method boolean handleInput()`
Corazón del control del jugador. Se llama una vez por fotograma:

1. Incrementa `moveTimer`. Si `moveTimer < moveDelay` → retorna `false` sin procesar nada (throttling).
2. Lee la tecla con `Keyboard.keyPressed()`.
3. Mapea la tecla a un delta de posición:
   - `W` / `w` / `↑` (flecha arriba) → `newRow = row - 1`
   - `S` / `s` / `↓` (flecha abajo) → `newRow = row + 1`
   - `A` / `a` / `←` (flecha izquierda) → `newCol = col - 1`
   - `D` / `d` / `→` (flecha derecha) → `newCol = col + 1`
4. Si `maze.isPassable(newRow, newCol)` y la posición cambiaría:
   - Llama a `maze.drawCell(row, col)` para limpiar la celda anterior.
   - Actualiza `row` y `col`.
   - Reinicia `moveTimer = 0`.
   - Retorna `true`.
5. Si el destino es un muro, fuera de límites, o no hay tecla presionada → retorna `false`.

### 6.2 Sprites por nivel

Todos los sprites miden **16 × 16 px** y se dibujan con primitivas `Screen` (rectángulos y líneas). Las coordenadas píxel se calculan como `px = col * cellSize`, `py = row * cellSize`.

#### `method void drawRaton()` — Nivel 1
Sprite de un ratón de aspecto amigable:
- Fondo de cara rectangular blanca con borde negro.
- Dos orejas cuadradas grandes en las esquinas superiores.
- Ojos negros de 2×2 px.
- Nariz puntito negro centrada.
- Bigotes finos horizontales a ambos lados.
- Boca curva sonriente formada por dos segmentos angulados.

#### `method void drawArdilla()` — Nivel 2
Sprite de una ardilla con rasgos característicos:
- Cara blanca más ancha con mejillas abultadas (rectángulos laterales).
- Orejas puntiagudas formadas con líneas diagonales.
- Ojos negros compactos.
- Nariz centrada.
- Dos dientes blancos prominentes con borde negro en la parte inferior.

#### `method void drawPajaro()` — Nivel 3
Sprite de un pájaro visto de frente:
- Copete de tres plumas en la parte superior formado con líneas diagonales simétricas.
- Cara blanca rectangular con borde negro.
- Ojos simétricos con pupila blanca (punto de reflejo).
- Pico triangular negro centrado apuntando hacia abajo.
- Indicadores de alas en las esquinas inferiores.

#### `method void dispose()`
Libera el objeto con `Memory.deAlloc(this)`.

### Video explicativo

**https://youtu.be/ZxsLRNG3gok?si=mCAVFk27iRliF00d**


---

## 7. Clase `Enemy`

**Archivo:** `Enemy.jack`

Agente autónomo que persigue al jugador usando una inteligencia artificial basada en **búsqueda en anchura (BFS)**. Cambia de sprite según el nivel y gestiona su memoria de forma eficiente mediante arrays estáticos compartidos.

### 7.1 Inteligencia artificial con BFS

El algoritmo BFS (Breadth-First Search) garantiza que el enemigo siempre tome el **camino más corto posible** hacia su objetivo dentro del laberinto, sin quedar atrapado en callejones sin salida.

#### Funcionamiento del algoritmo (método `update`)

El método `update` se llama una vez por fotograma. Internamente:

**Paso 1 — Control de velocidad**
Solo se ejecuta si `moveTimer >= moveInterval`. De lo contrario, incrementa el contador y retorna sin moverse. Esto permite ajustar la velocidad del enemigo independientemente de la del jugador.

**Paso 2 — Determinar el objetivo**
Según el tipo de enemigo (`enemyType`):
- Tipo 1 (directo): el objetivo es la posición actual del jugador `(playerRow, playerCol)`.
- Tipo 2 (anticipador): el objetivo es la posición `(playerRow + dirRow*2, playerCol + dirCol*2)`, es decir, 2 pasos adelante de donde el jugador se dirige. Si esa celda es un muro, cae de vuelta al tipo directo.

**Paso 3 — Inicializar el campo de distancias**
Se rellena el array `sharedDistField` (187 entradas) con el valor `9999`, que representa "no alcanzado aún".

**Paso 4 — Expansión BFS desde el objetivo**
Se coloca el objetivo en la cola `sharedBfsQueue` con distancia `0`. Luego, en un bucle:
- Se extrae la celda frontal de la cola.
- Se examinan sus 4 vecinos (arriba, abajo, izquierda, derecha).
- Por cada vecino transitable (`maze.isPassable`) que aún tenga distancia `9999`, se le asigna `distancia_actual + 1` y se encola.
- El proceso continúa hasta que la cola se vacía, creando un mapa completo de distancias desde el objetivo hasta todo el laberinto alcanzable.

**Paso 5 — Elegir el mejor paso**
El enemigo examina sus 4 celdas vecinas y elige moverse a la que tenga el **valor más bajo** en `sharedDistField` (la más cercana al objetivo). Si ningún vecino mejora la posición actual, el enemigo no se mueve.

**Paso 6 — Aplicar movimiento**
Si se encontró un mejor vecino:
1. Llama a `maze.drawCell(row, col)` para borrar su sprite actual.
2. Actualiza `row`, `col`, `lastRow`, `lastCol`.

#### `method void tryEscape(int playerRow, int playerCol)`
Sistema de rescate anti-atascos. Si el enemigo queda atrapado oscilando entre dos celdas (lo cual puede ocurrir en ciertos pasillos estrechos), este método intenta moverse en un orden de prioridad fijo: abajo → derecha → izquierda → arriba. Se mueve en la primera dirección viable que encuentre.

### 7.2 Tipos de comportamiento

| Tipo | Valor | Comportamiento |
|---|---|---|
| Directo | 1 | Persigue siempre la posición actual del jugador. Es predecible pero efectivo en laberintos estrechos. |
| Anticipador | 2 | Intenta ir 2 pasos por delante del jugador. Más agresivo en espacios abiertos. Si el punto anticipado es un muro, se comporta como tipo 1. |

En la implementación actual, todos los enemigos (incluido `enemy2` en el nivel 3) son de tipo 1. El tipo 2 está implementado y funcional, disponible para configuración futura.

### 7.3 Gestión de memoria estática

Un problema crítico en Jack es la **fragmentación del heap**: si se asignan y liberan arrays frecuentemente (por ejemplo, al recargar un nivel), el heap puede fragmentarse hasta el punto de que no haya un bloque contiguo suficientemente grande para una nueva asignación, lanzando el error `ERR6`.

La solución implementada es usar **arrays BFS estáticos** compartidos entre todos los enemigos:

| Campo estático | Tamaño | Uso |
|---|---|---|
| `sharedDistField` | 187 enteros | Campo de distancias BFS (una entrada por celda del laberinto) |
| `sharedBfsQueue` | 187 enteros | Cola de nodos BFS |
| `bfsReady` | `boolean` | Bandera que indica si los arrays ya fueron inicializados |

Estos arrays se crean **una sola vez** (cuando se instancia el primer `Enemy` y `bfsReady` es `false`) y nunca se liberan. En reinicios de nivel, los nuevos enemigos reutilizan los mismos arrays, eliminando el riesgo de fragmentación.

Como Jack es **single-threaded**, el hecho de que dos enemigos compartan los mismos arrays es completamente seguro: nunca ejecutan `update()` simultáneamente.

El método `dispose()` del enemigo **no libera** `sharedDistField` ni `sharedBfsQueue`. Solo libera el objeto enemigo en sí con `Memory.deAlloc(this)`.

### 7.4 Sprites por nivel

Los sprites del enemigo también miden **16 × 16 px**.

#### `method void drawGato()` — Niveles 1 y 2
Sprite de un gato con cara **negra** (contrasta con el fondo blanco del laberinto):
- Cara negra rectangular con esquinas suavizadas (píxeles blancos en las 4 esquinas).
- Orejas negras en la parte superior con interior blanco.
- Ojos grandes blancos con pupila negra 2×2 px y un destello blanco en el ángulo superior.
- Nariz blanca centrada pequeña.
- Boca blanca curva con dos segmentos angulados.
- Bigotes blancos dobles a ambos lados.

#### `method void drawPerro()` — Nivel 2
En la implementación actual, este método simplemente delega a `drawGato()`. El Perro comparte visualmente el sprite del Gato. Está separado en un método propio para facilitar una futura personalización del sprite.

#### `method void drawSerpiente()` — Nivel 3
Sprite de una serpiente de aspecto reptiliano:
- Cabeza negra con forma redondeada (dos rectángulos superpuestos cruzados para simular la forma ovalada).
- Ojos de reptil: rectángulos blancos horizontales (esclerótica) con pupila vertical negra y destello blanco.
- Escamas: patrón de puntos/rectángulos blancos en el centro de la cabeza.
- Lengua bífida blanca en la parte inferior, formada por tres líneas: un tronco central y dos ramas divergentes.
- Patrón de escamas laterales formado por líneas diagonales.

---

## 8. Clase `UI`

**Archivo:** `UI.jack`

Clase puramente estática que centraliza todo el código de presentación visual: pantallas de menú, transiciones entre niveles, títulos en pixel art y los iconos en miniatura de los personajes. Separa completamente la capa de presentación de la lógica del juego.

Todos sus métodos son `function` (no `method`), lo que significa que se invocan como `UI.nombreMetodo(...)` sin necesitar un objeto instanciado.

### 8.1 Pantallas del juego

#### `function void drawStart()`
Dibuja la pantalla de título completa:
1. Limpia la pantalla y dibuja el marco decorativo con `drawFrame()`.
2. Renderiza el logo "MAZE" en pixel art de ~20 px de alto con `drawTitle()`.
3. Imprime "I N T E L L I G E N C E" y la sección de controles con texto normal.
4. Dibuja manualmente las cuatro flechas direccionales (↑ ↓ ← →) con líneas `Screen`, ya que la fuente estándar de Jack no las incluye.
5. Muestra "[PRESIONA ESPACIO]" con **efecto de parpadeo**: el texto alterna entre visible e invisible cada 15 ticks (~240 ms) usando un contador y `Sys.wait(16)`.
6. Espera a que la tecla ESPACIO se suelte antes de retornar, para evitar que el input se propague al siguiente estado.

---

#### `function void drawLevelList()`
Pantalla de presentación de los tres niveles disponibles:
- Muestra el título "NIVELES" en pixel art con `drawNivelesTitle()`.
- Lista los tres niveles con su temática y dificultad en texto normal.
- Muestra los iconos en miniatura de los 6 personajes con `drawMiniIcons()`.
- Usa el efecto de parpadeo en el botón ESPACIO para iniciar.

---

#### `function void drawLevelIntro(int currentLevel)`
Pantalla de presentación previa a cada nivel:
- Muestra "NIVEL X" en pixel art con `drawLargeNivelTitle(currentLevel)`.
- Muestra el enfrentamiento de personajes (ej. "Ratón vs Gato") con los iconos de ambos usando `drawVersusIcons(currentLevel, py)`.
- Describe la dificultad del nivel y el objetivo del juego en texto.
- Espera que el jugador presione ESPACIO para comenzar.

---

#### `function int drawLevelWin(int currentLevel)`
Pantalla de nivel superado:
- Muestra "¡NIVEL SUPERADO!" en pixel art con `drawNivelSuperadoTitle()`.
- Indica el nivel completado y cuál viene a continuación.
- **Retorna el keycode** de la tecla que el jugador presionó:
  - `32` (ESPACIO) → continuar al siguiente nivel.
  - `81` ó `113` (`Q`/`q`) → salir del juego.
- Esto permite a `Game.handleLevelWin()` tomar la decisión de flujo sin que `UI` conozca la lógica del juego.

---

#### `function int drawGameOver(int currentLevel)`
Pantalla de Game Over:
- Muestra "GAME OVER" en pixel art con `drawGameOverTitle()`.
- Indica el nivel en el que falló el jugador.
- Ofrece las opciones `[R]` para reintentar y `[Q]` para salir.
- **Retorna el keycode** de la elección del jugador.

---

#### `function int drawVictory()`
Pantalla de victoria final:
- Muestra "¡VICTORIA!" en pixel art con `drawVictoryTitle()`.
- Presenta un resumen de los enemigos superados en los tres niveles.
- Ofrece únicamente `[Q]` para salir.
- **Retorna el keycode** (siempre `Q`). `Game.handleVictory()` ignorará el valor y terminará el programa de todas formas.

---

#### `function void drawPause(int currentLevel)`
Pantalla de pausa:
- Muestra "PAUSA" en pixel art con `drawPauseTitle()`.
- Indica el nivel actual y la instrucción para reanudar.
- Espera al jugador con una **triple barrera** de seguridad para evitar que la misma pulsación de `[P]` que activó la pausa la desactive inmediatamente:
  1. Espera a que no haya ninguna tecla presionada.
  2. Espera a que se presione específicamente `[P]`.
  3. Espera a que se suelte `[P]`.

---

### 8.2 Helpers de dibujo

Estos métodos son privados por convención (no hay modificadores de acceso en Jack) y son llamados únicamente desde las pantallas principales de `UI`.

#### `function void drawFrame()`
Dibuja un marco decorativo de doble borde alrededor de toda la pantalla (512 × 256 px) usando cuatro rectángulos concéntricos alternando negro y blanco.

---

#### `function void drawTitle()`
Dibuja las letras **"MAZE"** en pixel art de ~20 px de alto, usando rectángulos y líneas. La letra `Z` utiliza `Screen.drawLine()` diagonal para lograr el trazo inclinado característico.

---

#### `function void drawNivelesTitle()`
Dibuja **"NIVELES"** en pixel art de ~20 px de alto.

---

#### `function void drawLargeNivelTitle(int num)`
Dibuja **"NIVEL"** seguido del dígito (`1`, `2` ó `3`) en pixel art de ~20 px. El dígito se selecciona con tres condicionales `if` sobre el parámetro `num`.

---

#### `function void drawPauseTitle()`
Dibuja **"PAUSA"** en pixel art.

---

#### `function void drawNivelSuperadoTitle()`
Dibuja **"¡NIVEL SUPERADO!"** completo en pixel art de ~20 px. Es el título más largo del juego; las coordenadas absolutas de cada letra están cuidadosamente calculadas para que encajen en la pantalla de 512 px de ancho.

---

#### `function void drawGameOverTitle()`
Dibuja **"GAME OVER"** en pixel art.

---

#### `function void drawVictoryTitle()`
Dibuja **"¡VICTORIA!"** en pixel art.

---

#### `function void drawVersusIcons(int currentLevel, int py)`
Dibuja los dos iconos de personajes enfrentados en la pantalla de introducción de nivel: el personaje del jugador a la izquierda y el enemigo a la derecha, centrados en la coordenada Y indicada (`py`). Los iconos se dibujan con las mismas primitivas `Screen` que los sprites del juego, pero con coordenadas absolutas adaptadas al menú. La lógica selecciona qué par de personajes mostrar según `currentLevel`:
- Nivel 1 → Ratón (izq.) vs Gato (der.)
- Nivel 2 → Ardilla (izq.) vs Perro/Gato (der.)
- Nivel 3 → Pájaro (izq.) vs Serpiente (der.)

---

#### `function void drawMiniIcons()`
Dibuja los **6 iconos en miniatura** de todos los personajes del juego (3 jugadores + 3 enemigos) en la pantalla de lista de niveles, organizados en una cuadrícula de 2 columnas × 3 filas. Usa las mismas primitivas de sprite pero con coordenadas absolutas fijas para encajar dentro del layout de la pantalla de selección de nivel.

---

## 9. Clase `Random`

**Archivo:** `Random.jack`

Utilidad global completamente estática que provee generación de **números pseudoaleatorios** mediante un **Generador Congruencial Lineal (LCG)**. No necesita ser instanciada; todos sus métodos son `function`.

Un LCG genera una secuencia de números aplicando repetidamente la fórmula:

```
seed_nuevo = (seed_anterior × a) + c
```

Donde `a = 25173` y `c = 13849` son constantes elegidas para producir una secuencia con buen período en el rango de enteros de 16 bits de Jack.

### Campos estáticos

| Campo | Tipo | Descripción |
|---|---|---|
| `seed` | `int` | Semilla actual de la secuencia. Se actualiza en cada llamada a `nextInt`. |

### Métodos

#### `function void setSeed(int newSeed)`
Establece la semilla directamente a un valor conocido. Si se usa la misma semilla al inicio, la secuencia de números generados será siempre idéntica. Útil para reproducibilidad en pruebas.

---

#### `function void addSeed(int amount)`
Suma un valor a la semilla actual sin reemplazarla. Se puede llamar periódicamente durante el juego (por ejemplo, en cada tick) para introducir variación basada en el tiempo de reacción del jugador, haciendo que la secuencia sea más impredecible entre partidas.

---

#### `function int nextInt(int max)`
Genera y retorna el siguiente número pseudoaleatorio en el rango `[0, max - 1]`:

1. Aplica la fórmula LCG: `seed = (seed * 25173) + 13849`.
2. Guarda el nuevo seed en `val`.
3. Si `val` es negativo (overflow de entero con signo), lo convierte a positivo con `val = -val`. Si el resultado es todavía negativo (caso especial de `MIN_VALUE`), lo fija a `0`.
4. Calcula el módulo: `val - ((val / max) * max)`, equivalente a `val % max` (Jack no tiene operador `%`).

**Nota:** La uniformidad de la distribución no es perfecta (sesgo del módulo), pero es suficiente para las necesidades de un videojuego sencillo.

---

## 10. Resumen de interacciones entre clases

La siguiente tabla resume qué clase usa qué otra clase y para qué propósito:

| Clase que llama | Clase usada | Propósito |
|---|---|---|
| `Main` | `Game` | Crear, ejecutar y destruir la instancia principal |
| `Game` | `Maze` | Crear laberinto, dibujarlo, consultar posición de salida |
| `Game` | `Player` | Crear, obtener posición, procesar input, dibujar, destruir |
| `Game` | `Enemy` | Crear, actualizar (IA), dibujar, destruir |
| `Game` | `UI` | Mostrar todas las pantallas de menú y transición |
| `Player` | `Maze` | Validar movimiento (`isPassable`), borrar sprite anterior (`drawCell`) |
| `Enemy` | `Maze` | Validar celdas en BFS (`isPassable`), borrar sprite anterior (`drawCell`), obtener dimensiones (`getRows`, `getCols`) |
| `UI` | *(ninguna)* | Usa solo primitivas del SO (`Screen`, `Output`, `Keyboard`, `Sys`) |
| `Random` | *(ninguna)* | Utilidad matemática pura, sin dependencias |

**Regla clave de dependencia:** las dependencias fluyen hacia abajo (`Game` → `Maze`/`Player`/`Enemy`/`UI`). Ninguna clase de nivel inferior conoce a `Game`, lo que facilita el mantenimiento y la extensión del proyecto.
