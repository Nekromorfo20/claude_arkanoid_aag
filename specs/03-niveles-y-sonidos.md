# Spec 03 — Niveles y sonidos

**Estado:** Aprobado
**Depende de:** SPEC 01, SPEC 02
**Fecha:** 2026-10-16

## Objetivo

Agregar progresión de 5 niveles con layouts y velocidad de pelota crecientes, y reproducir los efectos de sonido existentes (`ball-bounce.mp3`, `break-sound.mp3`) en los rebotes de pelota y roturas de bloque.

## Scope

**Incluye:**
- 5 niveles predefinidos, cada uno con su propio grid de bloques (layout) y su propio multiplicador de velocidad de pelota.
- Al romper todos los bloques del nivel actual (mismo trigger que ya usa Spec 01/02 para "regenerar grid", incluyendo la espera a que terminen las explosiones activas de Spec 02), se avanza al siguiente nivel: se carga su grid y se aplica su velocidad.
- Al completar el nivel 5, se vuelve al nivel 1 (loop infinito), sin pantalla de victoria, consistente con la filosofía de Spec 01 de no detener nunca el juego.
- La velocidad de la pelota aumenta mediante un multiplicador fijo por nivel sobre la velocidad base: `velocidad = velocidad_base * (1 + 0.15 * (nivel - 1))`.
- Al perder una vida (Spec 01) y resetear la pelota, la velocidad aplicada es la del nivel actual (no se reinicia a la velocidad base del nivel 1).
- Contador de "Nivel: N" visible en pantalla, junto a los contadores existentes de vidas y puntaje.
- Reproducción de `assets/sounds/ball-bounce.mp3` en cada rebote de la pelota contra paredes (izquierda, derecha, arriba) y contra la paddle.
- Reproducción de `assets/sounds/break-sound.mp3` en el momento del impacto que rompe un bloque (mismo instante en que se suma el `score`, igual que en Spec 02).
- Reproducción de sonidos mediante `new Audio(src)` por cada disparo (instancia nueva cada vez), para permitir solapamientos sin cortar reproducciones en curso.
- Volumen fijo (por defecto del navegador), sin control de mute.

**No incluye (fuera de scope de esta spec):**
- Bloques con más de un golpe de resistencia (`hits` por bloque).
- Pantallas de Game Over o Victoria.
- Botón o control de mute/volumen.
- Persistencia de nivel o puntaje entre sesiones.
- Power-ups.
- Sonido para pérdida de vida o cambio de nivel (solo rebote y rotura de bloque quedan en scope).

## Modelo de datos

Se introducen las siguientes estructuras nuevas en `game.js`:

- `LEVELS`: array fijo de 5 definiciones de nivel, cada una `{ grid: [[color|null, ...], ...], speedMultiplier: number }`.
  - `grid`: matriz de filas x columnas (mismas dimensiones que el grid de Spec 01, ej. 8x5) donde cada celda es un nombre de color válido del spritesheet (`gray, red, yellow, cyan, magenta, hotpink, green`) o `null` para un hueco sin bloque.
  - `speedMultiplier`: número usado para escalar la velocidad base de la pelota en ese nivel (nivel 1 = `1.0`, y los siguientes siguen la fórmula `1 + 0.15 * (nivel - 1)`).
- `currentLevel`: número entero, inicia en 1 (índice de nivel actual, 1-based, dentro de `LEVELS`).
- `SOUNDS`: objeto con las rutas de los efectos, `{ ballBounce: 'assets/sounds/ball-bounce.mp3', brickBreak: 'assets/sounds/break-sound.mp3' }`, usado como fuente para instanciar `new Audio(...)` en cada reproducción.

El array `bricks` (Spec 01) y `explosions` (Spec 02) no cambian de forma; `bricks` se regenera a partir de `LEVELS[currentLevel - 1].grid` en lugar de un único grid fijo.

## Plan de implementación

1. Refactorizar la generación del grid de bloques para que lea desde una estructura `LEVELS[0]` (el nivel 1 actual) en vez de un grid hardcodeado único, manteniendo el comportamiento visual idéntico al de Spec 01/02. Verificable: el juego se ve y funciona igual que antes del cambio.
2. Definir `LEVELS` con 5 niveles: 5 grids distintos (layouts con variación de colores y huecos) y sus `speedMultiplier` según la fórmula `1 + 0.15 * (nivel - 1)`. Verificable: inspeccionando el array en consola se ven los 5 niveles con sus datos correctos.
3. Agregar `currentLevel` (inicia en 1) y aplicar `LEVELS[currentLevel - 1].speedMultiplier` al calcular la velocidad de la pelota, tanto al iniciar el juego como al resetear la pelota tras perder una vida. Verificable: la pelota se mueve visiblemente más rápido si se fuerza `currentLevel` a un valor alto.
4. Modificar la condición de regeneración de grid (que en Spec 02 espera `bricks` todos rotos y `explosions.length === 0`) para que, en lugar de regenerar el mismo grid, incremente `currentLevel` (volviendo a 1 si supera 5) y cargue `LEVELS[currentLevel - 1].grid`. Verificable: al romper todos los bloques de un nivel, aparece el grid del siguiente nivel y la pelota se mueve más rápido.
5. Renderizar en pantalla el texto "Nivel: N" junto a los contadores de vidas y puntaje ya existentes. Verificable: el valor se actualiza visualmente al cambiar de nivel.
6. Crear el objeto `SOUNDS` con las rutas de los dos archivos de audio existentes en `assets/sounds/`. Verificable: no hay errores de consola al cargar `game.js`.
7. Reproducir `new Audio(SOUNDS.ballBounce).play()` en cada rebote de la pelota contra paredes y contra la paddle (puntos ya identificados en Spec 01, pasos 5 y 6). Verificable: se escucha el sonido en cada rebote, incluso si ocurren varios rebotes seguidos.
8. Reproducir `new Audio(SOUNDS.brickBreak).play()` en el mismo punto donde se suma el `score` al romper un bloque (Spec 01 paso 10 / Spec 02 paso 1). Verificable: se escucha el sonido al romper cada bloque, incluso si se rompen varios bloques en el mismo frame.

## Criterios de aceptación

- [ ] El juego inicia en el nivel 1, mostrando su grid y velocidad correspondiente.
- [ ] Al romper todos los bloques del nivel actual (y tras finalizar las explosiones activas), se carga el grid del siguiente nivel y la pelota adopta la velocidad de ese nivel.
- [ ] Al completar el nivel 5, el juego vuelve al nivel 1 y continúa sin pantallas ni interrupciones.
- [ ] La velocidad de la pelota es visiblemente mayor en niveles más altos, siguiendo el multiplicador `1 + 0.15 * (nivel - 1)`.
- [ ] Al perder una vida, la pelota se resetea con la velocidad del nivel actual (no la velocidad base del nivel 1).
- [ ] El HUD muestra "Nivel: N" junto a vidas y puntaje, actualizado en tiempo real.
- [ ] Se escucha `ball-bounce.mp3` en cada rebote contra paredes y contra la paddle, incluso en rebotes consecutivos rápidos.
- [ ] Se escucha `break-sound.mp3` en cada rotura de bloque, incluso si varios bloques se rompen en el mismo frame.
- [ ] No hay errores de consola relacionados a carga o reproducción de audio.
- [ ] No hay dependencias externas (sin frameworks, sin build tools, sin paquetes npm).

## Decisiones tomadas y descartadas

- **Spec combinada de niveles y sonidos** en lugar de dos specs separadas: ambas features son acotadas y no se pisan entre sí; se prioriza evitar el overhead de coordinar dos specs/implementaciones por separado — decisión explícita del usuario.
- **Layout + velocidad por nivel**, sin bloques de múltiples golpes: se mantiene el campo `alive` booleano de Spec 01/02 sin modificar su forma; la resistencia variable de bloques queda para una spec futura si se necesita más dificultad.
- **Loop infinito al completar el nivel 5** en lugar de pantalla de victoria: consistente con la decisión ya tomada en Spec 01 de no detener nunca el juego ni mostrar pantallas de fin.
- **Multiplicador fijo por fórmula** (`1 + 0.15 * (nivel - 1)`) en lugar de tabla explícita de velocidades: mantiene `LEVELS` simple y la dificultad ajustable modificando un solo número.
- **La velocidad post-pérdida-de-vida usa el nivel actual**, no la velocidad base: evita que perder vidas en niveles avanzados "facilite" el juego bajando la velocidad, manteniendo la dificultad coherente con el nivel en curso.
- **`new Audio(src)` por cada reproducción** en lugar de una sola instancia reutilizada: permite que sonidos de rebotes/roturas simultáneos o consecutivos no se corten entre sí (ej. varios bloques rotos en el mismo frame por Spec 02).
- **Sin control de mute/volumen**: mantiene el alcance acotado, igual que el resto del MVP; se deja para una spec futura de UI/opciones si se necesita.
- **5 niveles hardcodeados en `game.js`**, sin generación procedural: suficiente para validar la progresión sin agregar complejidad de generación de niveles.

## Riesgos identificados

- **Fuga de instancias de `Audio`**: crear una nueva instancia por cada reproducción (rebotes y roturas frecuentes) podría acumular objetos en memoria si el navegador no los libera tras terminar de reproducir. Mitigación: los navegadores modernos recolectan instancias de `Audio` sin referencias activas una vez finalizada la reproducción; con el volumen de eventos de este juego (rebotes y bloques por partida) el impacto es despreciable.
- **Autoplay bloqueado por el navegador**: algunos navegadores bloquean la reproducción de audio antes de una interacción explícita del usuario. Mitigación: el primer input del jugador (mover la paddle, tocar una tecla) ya ocurre antes del primer rebote/rotura, por lo que el contexto de audio debería estar habilitado para cuando se dispare el primer sonido.
- **Grid de nivel con dimensiones distintas al de Spec 01/02**: si algún `LEVELS[n].grid` tuviera un número de filas/columnas distinto, podría romper el cálculo de posiciones `x, y` del grid. Mitigación: los 5 grids definidos respetan las mismas dimensiones (filas x columnas) que el grid original de Spec 01.
