# Historial de Cambios Proyecto Final

## [1.0.0] - 2026-05-10

* **Estructura del proyecto:** Estructura base completa de todas las clases del proyecto (esqueleto inicial en blanco, sin implementación interna (`Main`, `Game`, `Maze`, `Player` y `Enemy`) 
* **Licencia (`LICENSE`):** Agregado el archivo con los términos legales de uso y distribución del software.
* **Contribuidores (`CONTRIBUTORS.md`):** Registro formal de los autores, desarrolladores y colaboradores que participaron en la creación del proyecto, con sus respectivas tareas.

### Descripción
Inicialización del repositorio y definición de la arquitectura base del proyecto. Se establecieron los archivos correspondientes a todas las clases principales completamente en blanco, con el objetivo de estructurar la modularidad y la separación de responsabilidades antes de desarrollar la lógica interna. Asimismo, se integraron los documentos administrativos para la gestión de los derechos de autor y la declaración de los contribuidores del desarrollo.

## [1.0.1] - 2026-05-10

### Agregado

- **Esqueletos de componentes:** Delimitación formal de las áreas reservadas para los atributos (campos), constructores, métodos de negocio (lógica, game loop, inputs y renderizado) y destructores (`dispose`) en cada archivo `.jack`.

### Descripción
Se actualizó la arquitectura base del proyecto pasando de archivos completamente en blanco a esqueletos debidamente comentados y seccionados. Esta versión establece una guía clara para la implementación modular, definiendo de antemano la ubicación exacta de las variables de estado, la inicialización de los objetos en la matriz, las rutinas de actualización continua y la correcta gestión de liberación de memoria para prevenir fugas. 

Con esta base documental, las clases quedan preparadas para iniciar la integración del código y la lógica interna.

## [1.1.0] - 2026-05-12
### Agregado
* **Jugabilidad del Player:** Implementación completa del movimiento y físicas del jugador.
* **Sprite Inicial:** Se dibujó el primer diseño básico del jugador (una cara).
* **Sistema de Colisiones:** Se implementó la detección de colisiones del jugador contra las paredes del laberinto.

### Descripción
Primera prueba funcional enfocada exclusivamente en el jugador. Se logró que el avatar (con su primer sprite) se moviera correctamente por el mapa y respetara los límites físicos del laberinto, confirmando que la captura de teclado y el motor de físicas base funcionaban. La clase `Enemy` se mantuvo como un esqueleto vacío.

## [1.2.0] - 2026-05-14
### Agregado
* **Entidad Enemiga Genérica:** Integración de la clase `Enemy` al bucle del juego.
* **Sprite Genérico:** Se le asignó una "cara" o diseño base al enemigo para poder visualizarlo en pantalla.
* **Movimiento Acoplado:** Primera iteración del desplazamiento del enemigo.

### Descripción
Integración del primer prototipo de enemigo. En esta etapa, ambas entidades tenían diseños genéricos. La principal característica fue el movimiento acoplado: el enemigo carecía de autonomía y se movía estrictamente al mismo tiempo que el jugador; si el jugador detenía su input, el enemigo también se detenía. Se identificó un desbalance crítico donde el enemigo se desplazaba a una velocidad excesivamente alta.

## [1.2.1] - 2026-05-15
### Modificado
* **Balance de Velocidades (ms):** Ajuste en las pausas de ejecución (`Sys.wait`) y en la actualización de fotogramas.

### Descripción
Fase de balanceo inicial. Se realizaron múltiples iteraciones subiendo y bajando los milisegundos (ms) de retraso en las rutinas de movimiento tanto del `Player` como del `Enemy` para calibrar la velocidad, logrando que el juego fuera controlable y no penalizara injustamente al jugador.

## [1.3.0] - 2026-05-17
### Agregado
* **Diseño de Interfaz (UI):** Creación del apartado visual externo al laberinto.

### Descripción
Revisión estética del proyecto. El equipo se enfocó en el diseño de la interfaz de usuario para garantizar que fuera agradable, intuitiva y visualmente linda, preparando el lienzo para la integración de los niveles definitivos.

## [1.4.0] - 2026-05-19
### Agregado
* **Sistema de Niveles:** Definición de temáticas e integración de la progresión de los 3 niveles del juego.
* **Diseño de Personajes Finales:** Sustitución de los sprites genéricos por los personajes temáticos definitivos (Ratón para el jugador, Gato y Serpientes para los enemigos).

### Modificado
* **Consolidación Visual:** Acoplamiento definitivo de la interfaz de usuario terminada con los nuevos sprites y los mapas de los niveles.

### Descripción
Transformación del prototipo a un juego temático. Se estructuró el flujo de la partida dividiéndola en tres niveles con dificultades escalonadas. Se integraron todos los recursos artísticos finales, uniendo la interfaz pulida con los diseños oficiales de los personajes para cada etapa.

## [1.5.0] - 2026-05-22
### Agregado
* **Inteligencia Artificial (IA):** Implementación de la lógica de persecución autónoma. El enemigo ahora persigue al jugador independientemente de si este último se mueve o está quieto.
* **Aleatoriedad y Pathfinding:** Configuración de aparición (spawn) aleatoria con zonas seguras y mejora en la búsqueda de rutas para esquivar paredes.

### Arreglado
* **Sobrescritura de Píxeles:** Solución a los bugs visuales donde el movimiento de las entidades alteraba el color de las paredes del laberinto.

### Descripción
Actualización crítica de mecánicas. Se reemplazó el movimiento acoplado por una IA real, dotando a los enemigos de la capacidad de cazar activamente al jugador. Se resolvieron fallos de comportamiento (como amagues en las esquinas y retrocesos de las serpientes) y se implementó la aleatoriedad de inicio.

## [1.6.0] - 2026-05-25
### Modificado
* **Optimización de Memoria:** Revisión exhaustiva del ciclo de vida de los objetos e implementación estricta de los métodos `dispose()`.
* **Ajustes Menores:** Refinamiento de "cositas básicas" (ajustes de paleta de color del gato, corrección de excepciones como el `ERR6`, y limpieza de renderizado).

### Descripción
Fase de depuración y optimización técnica. El enfoque principal fue auditar el código fuente para detectar y eliminar fugas de memoria (Memory Leaks), garantizando que todos los objetos instanciados (matrices, sprites, entidades) liberaran correctamente su espacio en la RAM al ser destruidos o al cambiar de nivel.

## [1.7.0] - 2026-05-27
### Agregado
* **Documentación Final:** Inclusión de guías de usuario (`User_Guide.md`), documentación de clases y detalles de ejecución.

### Eliminado
* **Directorio de Pruebas:** Eliminación de la carpeta `/test` y archivos residuales del desarrollo.

### Descripción
Cierre del ciclo de desarrollo. Se redactó y adjuntó toda la documentación técnica y de usuario requerida para la entrega final. Paralelamente, se limpió el repositorio eliminando entornos de prueba y archivos innecesarios para dejar una versión de producción pulida y lista para ser compilada.

## [1.8.0] - 2026-05-28

### Agregado
* **Material Audiovisual Educativo:** Inclusión de videos explicativos detallados para cada una de las clases principales (`Main`, `Game`, `Player`, `Maze`, `Enemy` y `Random`) detallando su lógica interna y acoplamiento estructural.
* **Integridad de Archivos (`.md5`):** Generación e integración de las sumas de verificación en formato `.md5` para todos los binarios y archivos fuentes compilados, garantizando la validación de la entrega.

### Descripción
Fase de validación, auditoría y entrega final. Se grabó y estructuró un conjunto de guiones y material en video para sustentar de manera clara y comprensible el funcionamiento interno de cada módulo del software, facilitando la comprensión de la lógica del ciclo de juego, la inteligencia artificial de persecución y la gestión de aleatoriedad. Adicionalmente, se implementó un control de integridad técnica mediante archivos de verificación criptográfica de datos (`.md5`), asegurando que los archivos entregados correspondan fielmente a la versión final de producción sin alteraciones ni corrupciones durante la transferencia.

