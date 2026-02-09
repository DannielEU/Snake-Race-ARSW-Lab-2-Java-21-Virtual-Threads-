# Snake Race — ARSW Lab #2 (Java 21, Virtual Threads)

**Escuela Colombiana de Ingeniería – Arquitecturas de Software**  
Laboratorio de programación concurrente: condiciones de carrera, sincronización y colecciones seguras.

---

## Requisitos

- **JDK 21** (Temurin recomendado)
- **Maven 3.9+**
- SO: Windows, macOS o Linux

---

## Cómo ejecutar

```bash
mvn clean verify
mvn -q -DskipTests exec:java -Dsnakes=4
```

- `-Dsnakes=N` → inicia el juego con **N** serpientes (por defecto 2).
- **Controles**:
  - **Flechas**: serpiente **0** (Jugador 1).
  - **WASD**: serpiente **1** (si existe).
  - **Espacio** o botón **Action**: Pausar / Reanudar.

---

## Reglas del juego (resumen)

- **N serpientes** corren de forma autónoma (cada una en su propio hilo).
- **Ratones**: al comer uno, la serpiente **crece** y aparece un **nuevo obstáculo**.
- **Obstáculos**: si la cabeza entra en un obstáculo hay **rebote**.
- **Teletransportadores** (flechas rojas): entrar por uno te **saca por su par**.
- **Rayos (Turbo)**: al pisarlos, la serpiente obtiene **velocidad aumentada** temporal.
- Movimiento con **wrap-around** (el tablero “se repite” en los bordes).

---

## Arquitectura (carpetas)

```
co.eci.snake
├─ app/                 # Bootstrap de la aplicación (Main)
├─ core/                # Dominio: Board, Snake, Direction, Position
├─ core/engine/         # GameClock (ticks, Pausa/Reanudar)
├─ concurrency/         # SnakeRunner (lógica por serpiente con virtual threads)
└─ ui/legacy/           # UI estilo legado (Swing) con grilla y botón Action
```

---

# Actividades del laboratorio

## Parte I — (Calentamiento) `wait/notify` en un programa multi-hilo

1. Toma el programa [**PrimeFinder**](https://github.com/ARSW-ECI/wait-notify-excercise).
2. Modifícalo para que **cada _t_ milisegundos**:
   - Se **pausen** todos los hilos trabajadores.
   - Se **muestre** cuántos números primos se han encontrado.
   - El programa **espere ENTER** para **reanudar**.
3. La sincronización debe usar **`synchronized`**, **`wait()`**, **`notify()` / `notifyAll()`** sobre el **mismo monitor** (sin _busy-waiting_).
4. Entrega en el reporte de laboratorio **las observaciones y/o comentarios** explicando tu diseño de sincronización (qué lock, qué condición, cómo evitas _lost wakeups_).

> Objetivo didáctico: practicar suspensión/continuación **sin** espera activa y consolidar el modelo de monitores en Java.

---

## Parte II — SnakeRace concurrente (núcleo del laboratorio)

### 1) Análisis de concurrencia

- Explica **cómo** el código usa hilos para dar autonomía a cada serpiente.
- **Identifica** y documenta en **`el reporte de laboratorio`**:
  - Posibles **condiciones de carrera**.
  - **Colecciones** o estructuras **no seguras** en contexto concurrente.
  - Ocurrencias de **espera activa** (busy-wait) o de sincronización innecesaria.

### 2) Correcciones mínimas y regiones críticas

- **Elimina** esperas activas reemplazándolas por **señales** / **estados** o mecanismos de la librería de concurrencia.
- Protege **solo** las **regiones críticas estrictamente necesarias** (evita bloqueos amplios).
- Justifica en **`el reporte de laboratorio`** cada cambio: cuál era el riesgo y cómo lo resuelves.

### 3) Control de ejecución seguro (UI)

- Implementa la **UI** con **Iniciar / Pausar / Reanudar** (ya existe el botón _Action_ y el reloj `GameClock`).
- Al **Pausar**, muestra de forma **consistente** (sin _tearing_):
  - La **serpiente viva más larga**.
  - La **peor serpiente** (la que **primero murió**).
- Considera que la suspensión **no es instantánea**; coordina para que el estado mostrado no quede “a medias”.

### 4) Robustez bajo carga

- Ejecuta con **N alto** (`-Dsnakes=20` o más) y/o aumenta la velocidad.
- El juego **no debe romperse**: sin `ConcurrentModificationException`, sin lecturas inconsistentes, sin _deadlocks_.
- Si habilitas **teleports** y **turbo**, verifica que las reglas no introduzcan carreras.

> Entregables detallados más abajo.

---

## Entregables

1. **Código fuente** funcionando en **Java 21**.
2. Todo de manera clara en **`**el reporte de laboratorio**`** con:
   - Data races encontradas y su solución.
   - Colecciones mal usadas y cómo se protegieron (o sustituyeron).
   - Esperas activas eliminadas y mecanismo utilizado.
   - Regiones críticas definidas y justificación de su **alcance mínimo**.
3. UI con **Iniciar / Pausar / Reanudar** y estadísticas solicitadas al pausar.

---

## Criterios de evaluación (10)

- (3) **Concurrencia correcta**: sin data races; sincronización bien localizada.
- (2) **Pausa/Reanudar**: consistencia visual y de estado.
- (2) **Robustez**: corre **con N alto** y sin excepciones de concurrencia.
- (1.5) **Calidad**: estructura clara, nombres, comentarios; sin _code smells_ obvios.
- (1.5) **Documentación**: **`reporte de laboratorio`** claro, reproducible;

---

## Tips y configuración útil

- **Número de serpientes**: `-Dsnakes=N` al ejecutar.
- **Tamaño del tablero**: cambiar el constructor `new Board(width, height)`.
- **Teleports / Turbo**: editar `Board.java` (métodos de inicialización y reglas en `step(...)`).
- **Velocidad**: ajustar `GameClock` (tick) o el `sleep` del `SnakeRunner` (incluye modo turbo).

---

## Cómo correr pruebas

```bash
mvn clean verify
```

Incluye compilación y ejecución de pruebas JUnit. Si tienes análisis estático, ejecútalo en `verify` o `site` según tu `pom.xml`.

---

## Créditos

Este laboratorio es una adaptación modernizada del ejercicio **SnakeRace** de ARSW. El enunciado de actividades se conserva para mantener los objetivos pedagógicos del curso.

**Base construida por el Ing. Javier Toquica.**

---

**REPORTE DEL LABORATORIO (Respuestas y cambios realizados)**

Resumen de alcance: se implementaron correcciones de concurrencia y una mejora en la UI para mostrar estadísticas al pausar. Se omitió la Parte I (PrimeFinder) tal como solicitaste — no se realizó la búsqueda de primos aquí.

1) Análisis de concurrencia
- Cómo se usan hilos: cada serpiente corre en su propio task (virtual thread) mediante `Executors.newVirtualThreadPerTaskExecutor()` y la lógica de avance está en `SnakeRunner`.
- Posibles condiciones de carrera encontradas:
  - Acceso concurrente a la estructura interna de `Snake` (`body`): la UI (repaint) lee `body` vía `snapshot()` mientras `SnakeRunner` lo modifica con `advance(...)`. Esto puede provocar lecturas inconsistentes o excepciones de concurrencia.
  - Otras estructuras compartidas (`Board`) ya estaban protegidas con `synchronized` en métodos críticos (`step`, getters devuelven copias), y `GameClock` usa `AtomicReference` para el estado.

2) Correcciones realizadas y justificación
- Protección mínima de `Snake.body`: los métodos que acceden o modifican `body` se sincronizaron (ahora `turn`, `head`, `snapshot`, `advance` son `synchronized`). Razón: reducir la región crítica al propio `Snake` en lugar de bloquear globalmente `Board` o la UI. Esto evita iteración concurrente mientras se realiza `new ArrayDeque<>(body)` en `snapshot()`.
- No se cambiaron las políticas de `Board.step()` — ya era `synchronized` y continúa protegiendo mutaciones del tablero e interacciones globales (mice, obstacles, teleports).

3) Esperas activas y alternativas
- Búsqueda: no se encontraron esperas activas (busy-wait) en el código base; `SnakeRunner` usa `Thread.sleep(...)` para temporizar movimientos (no busy-wait), y `GameClock` usa `ScheduledExecutorService`.
- Por tanto, no fue necesario eliminar loops con espera activa; sí se aplicó protección de datos donde era necesaria.

4) UI: Pausa / Reanudar y consistencia visual
- Comportamiento añadido: al pausar (botón Action o SPACE) el `GameClock` se pausa y se muestra un dialog con dos estadísticas consistentes calculadas a partir de snapshots sincronizados:
  - `Serpiente viva más larga`: índice y longitud.
  - `Peor serpiente (más corta)`: índice y longitud.
- Nota importante: el proyecto base no incluye un mecanismo de "muerte" de serpientes (no hay eliminación al chocar con otras serpientes). Por eso se interpreta "peor serpiente (la que primero murió)" como la serpiente con menor longitud al momento de la pausa; esto se documenta en el reporte.

5) Robustez bajo carga
- Pruebas manuales sugeridas:
  - Ejecutar: `mvn -q -DskipTests exec:java -Dsnakes=20` y observar que no aparecen `ConcurrentModificationException` ni `DeadLock` en ejecuciones prolongadas.
- Cómo lo hice seguro: la protección a nivel de `Snake` evita iteraciones concurrentes sobre `body`; `Board` ya protegía su estado interno.

6) Cómo reproducir y verificar cambios
- Compilar y ejecutar:

```bash
mvn clean verify
mvn -q -DskipTests exec:java -Dsnakes=4
```

- Para carga alta:

```bash
mvn -q -DskipTests exec:java -Dsnakes=20
```

7) Resumen de archivos modificados
- `src/main/java/co/eci/snake/core/Snake.java`: sincronización mínima de `turn`, `head`, `snapshot`, `advance`.
- `src/main/java/co/eci/snake/ui/legacy/SnakeApp.java`: al pausar se calculan y muestran estadísticas consistentes basadas en snapshots.

8) Observaciones finales y mejoras futuras
- Si deseas un comportamiento de muerte/abandono real (serpientes que mueren al chocar), puedo implementarlo añadiendo una estructura global de ocupación de celdas o extendiendo `Board` para conocer todas las serpientes y detectar colisiones atómicas. Esto requeriría cambios adicionales en `Board` y coordinación fina entre hilos.

Si quieres que implemente la mecánica de muerte o que modifique la definición de "peor serpiente" para ajustarla a otro criterio, dime y lo hago a continuación.
