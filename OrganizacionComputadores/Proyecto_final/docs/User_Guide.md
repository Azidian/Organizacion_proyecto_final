# Guía de Usuario — Maze Intelligence
*Tu guía completa para escapar del laberinto (o morir en el intento)*

---

## 1. Contexto: ¿De qué trata el juego?

Eres una criatura pequeña atrapada en un laberinto. En algún lugar de ese laberinto hay una salida marcada con una **X**. Tu único objetivo es llegar a ella. El problema: hay uno o dos depredadores con inteligencia artificial que conocen el laberinto tan bien como tú (probablemente mejor) y van a calcular la ruta óptima para interceptarte.

Cada uno de los tres niveles del juego tiene su propio laberinto, sus propios personajes y su propia dificultad.  

---

## 2. Controles

Se pueden usar cualquiera de los dos esquemas de control o mezclarlos libremente:

| Acción | Esquema WASD | Esquema Flechas |
|---|:---:|:---:|
| **Mover arriba** | `W` / `w` | `↑` |
| **Mover abajo** | `S` / `s` | `↓` |
| **Mover izquierda** | `A` / `a` | `←` |
| **Mover derecha** | `D` / `d` | `→` |
| **Pausar / Reanudar** | `P` / `p` | `P` / `p` |
| **Continuar (menús)** | `Espacio` | `Espacio` |
| **Reintentar Nivel** | `R` / `r` | `R` / `r` |
| **Salir del Juego** | `Q` / `q` | `Q` / `q` |

---

## 3. Pantallas del Juego

### Pantalla de Inicio
La primera pantalla que verás es el título: **MAZE** en letras grandes de pixel art, seguido del subtítulo **I N T E L L I G E N C E**. Debajo encontrarás un listado de los tres niveles disponibles y los controles. Cuando estés listo, presiona `[ESPACIO]` para comenzar desde el Nivel 1.

### Introducción de Nivel
Antes de comenzar a jugar, el juego te presenta los mundos con su temática:
- El número del nivel en grande.
- El enfrentamiento (ej. "Ratón vs Gato").
- La dificultad (Fácil / Normal / Extrema).

###  Juego 
El laberinto ocupa la parte central de la pantalla. Tú apareces como un sprite **blanco** (el jugador siempre tiene fondo blanco; los enemigos siempre tienen fondo negro). La salida es una celda con una **X** negra sobre fondo blanco.
- **Muros:** Celdas negras sólidas.
- **Pasillos:** Celdas blancas vacías.

### Pantalla de Pausa
En cualquier momento puedes presionar `[P]` para pausar. El estado del juego se congela por completo. Para reanudar, presiona `[P]` de nuevo y el juego continuará exactamente donde lo dejaste.

### Game Over y Reintentos
Si un enemigo llega a tu misma celda, el juego termina. Se mostrará el nivel en el que fallaste y tendrás opciones:
- **`[R]` Reintentar:** Inicia el nivel actual desde el principio (¡tienes reintentos infinitos!).
- **`[Q]` Salir:** Cierra el juego.

### Nivel Superado / Victoria
Al pisar la **X**, pasas al siguiente nivel. Al completar los tres niveles, verás la pantalla de **¡VICTORIA!** con un resumen de los enemigos derrotados a lo largo de la sesión.

---

## 4. Los Niveles en Detalle

El juego se desplaza celda a celda (16×16 px). Hay una pequeña pausa entre movimientos que disminuye en cada nivel, haciendo el juego progresivamente más rápido y exigente.

### Nivel 1 — Ratón vs. Gato `[Fácil]`

| Característica | Detalle |
|---|---|
| **Jugador** | Ratón (Clásico: cara blanca, orejas cuadradas, bigotes) |
| **Enemigo** | Gato (cara negra redonda, ojos con brillo, boquita) |
| **Velocidad Jugador** | 1 mov. cada 9 ciclos (~144 ms) |
| **Velocidad Enemigo**| 1 mov. cada 8 ciclos (~128 ms) |
| **Spawn Inicial** | Jugador en `(1, 1)` | Salida en `(1, 15)` |

> **Descripción:** El laberinto es amplio, con varios pasillos alternativos y mucho espacio para maniobrar. La salida está en la misma fila que el inicio.

### Nivel 2 — Ardilla vs. Gato `[Normal]`

| Característica | Detalle |
|---|---|
| **Jugador** | Ardilla (súper cola, mejillas regordetas, ojazos, dientecitos) |
| **Enemigo** | Gato (igual al Nivel 1) |
| **Velocidad Jugador** | 1 mov. cada 6 ciclos (~96 ms) |
| **Velocidad Enemigo**| 1 mov. cada 7 ciclos (~112 ms) |
| **Spawn Inicial** | Jugador en `(1, 1)` | Salida en `(9, 15)` |

> **Descripción:** Es un nivel más complejo. Los pasillos son más estrechos, hay más obstáculos y la salida está en la esquina diagonal opuesta. A cambio, ahora **eres ligeramente más rápido que el enemigo** (16 ms de ventaja por movimiento). Aprovéchala en las rectas largas.

### Nivel 3 — Pájaro vs. 2 Serpientes `[Extrema]`

| Característica | Detalle |
|---|---|
| **Jugador** | Pájaro (copete de plumas, pico triangular, ojitos blancos) |
| **Enemigos (x2)** | Serpientes (cabeza elongada, ojos reptil, lengua bífida) |
| **Velocidad Jugador** | 1 mov. cada 4 ciclos (~64 ms) |
| **Velocidad Enemigos**| 1 mov. cada 3 ciclos (~48 ms) |
| **Spawn Inicial** | Jugador en `(9, 1)` | Salida en `(5, 15)` (centro-derecha) |

> **Descripción:** Todo está en contra tuya. El laberinto es el más denso, con pasillos estrechos. Hay **dos enemigos** que calculan rutas independientes. Las serpientes se mueven un **33% más rápido que tú**. Para ganar, debes atravesar el corazón del laberinto sin dudar.

---

## 5. Consejos Prácticos y Estrategia

- **Planifica antes de actuar (Nivel 1 y 2):** El BFS de los enemigos calcula el camino más corto, pero todavía tienes ventaja suficiente para que un plan razonablemente bueno funcione.
- **Cero vacilaciones (Nivel 3):** No hay tiempo para pensar en el laberinto mientras lo recorres. La velocidad del jugador es alta, pero las serpientes siempre llevan ventaja. Si te detienes a pensar, mueres.
- **Aprovecha la Zona Segura inicial:** Las serpientes aparecen en coordenadas aleatorias, pero siempre a una distancia Manhattan mínima de 8 celdas de ti. Usa esa pequeña ventana inicial para alejarte del centro del mapa antes de que las rutas se crucen.
- **Domina las bifurcaciones:** No intentes "despistar" dando vueltas; la IA no se confunde. Si las serpientes se acercan desde direcciones opuestas, busca un pasillo perpendicular y cruza. Perseguirte por un pasillo recto es la situación donde el BFS es más letal; una bifurcación a tiempo fuerza al algoritmo a recalcular la ruta, dándote ticks vitales.

---

## 6. Funcionalidades y Mecánicas Avanzadas

El juego fue programado desde cero en el lenguaje Jack para el hardware simulado Hack OS. Resolvimos los siguientes desafíos técnicos:

1. **Algoritmo de Búsqueda BFS (Inteligencia Artificial):**
   Los enemigos no usan un simple seguimiento euclidiano. La IA implementa una **Búsqueda en Anchura (BFS)**. En cada movimiento, el enemigo genera un mapa de calor (`distField`) inundando el laberinto desde el jugador hacia afuera. Luego avanza a la celda adyacente con menor distancia. Esto garantiza la ruta óptima absoluta y evita atascos.

2. **Spawn Aleatorio y "Safe Zone":**
   En los niveles avanzados, el spawn enemigo usa `Random.nextInt()`. Para evitar muertes injustas instantáneas, un bucle `while` rechaza cualquier coordenada cuya distancia Manhattan (`|x1 - x2| + |y1 - y2|`) con respecto al jugador sea inferior a 8 casillas.

3. **Optimización de Memoria (Adiós al ERR6):**
   En hardware simulado, instanciar enormes `Array.new(187)` para las colas BFS de cada enemigo fragmentaba rápidamente el Heap, causando el Error 6 tras reiniciar. La solución fue convertir `sharedDistField` y `sharedBfsQueue` a propiedades de clase **`static`**. Se instancian una única vez y todos los enemigos comparten la misma memoria por turnos. Esto habilita **reintentos infinitos**.

4. **Sistema Inteligente de Re-Renderizado:**
   Mover un sprite de 16x16 píxeles puede dejar basura en pantalla. Antes de actualizar las coordenadas de cualquier entidad, el código invoca `maze.drawCell(row, col)`, lo que limpia instantáneamente la casilla repintando la celda de piso específica, logrando movimiento limpio sin parpadeos (*ghosting*) ni destrucción del mapa.

5. **Animación y Ciclo de CPU (Ticks):**
   Para asegurar fluidez sin sobrecargar el procesador, el bucle principal usa `Sys.wait(16)` (~60 FPS). Las velocidades se controlan mediante temporizadores ("ticks"): si una serpiente tiene velocidad `3`, mueve su coordenada una vez cada 3 ciclos de la CPU.
