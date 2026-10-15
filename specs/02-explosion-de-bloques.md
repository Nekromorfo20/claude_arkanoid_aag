# Spec 02 — Explosión de bloques

**Estado:** Aprobado
**Depende de:** SPEC 01
**Fecha:** 2026-10-15

## Objetivo

Reemplazar la desaparición instantánea de los bloques al ser golpeados por una animación de explosión de 4 frames usando `drawFrame`/`EXPLOSION_FRAMES` del spritesheet, manteniendo el bloque sin colisión desde el primer impacto.

## Scope

**Incluye:**
- Al golpear un bloque, en lugar de desaparecer instantáneamente, se reproduce su animación de explosión correspondiente a su color (`EXPLOSION_FRAMES[color]`, 4 frames, `EXPLOSION_DURATION = 150`ms total).
- El bloque deja de colisionar y de dibujarse como bloque sólido en el mismo instante del impacto (ya no bloquea ni rebota la pelota durante la animación).
- El puntaje se suma inmediatamente al momento del impacto, igual que en la Spec 01 — la animación no afecta el `score`.
- Una vez terminada la animación (pasados los 150ms), la explosión se retira por completo de la pantalla.
- La regeneración del grid completo (cuando se rompen todos los bloques) espera a que todas las explosiones activas terminen antes de recrear el grid.
- Reutilización de `drawFrame` y `EXPLOSION_FRAMES` ya existentes en `assets/spritesheet.js`, sin modificar ese archivo.

**No incluye (fuera de scope de esta spec):**
- Sonido de rotura (`break-sound.mp3`) — queda para una spec futura de audio/FX.
- Sonido de rebote (`ball-bounce.mp3`) — igualmente fuera de scope.
- Cualquier cambio a la lógica de puntaje, vidas, o colisión pelota-paddle/paredes.
- Power-ups, niveles, o cualquier otra mecánica nueva.

## Modelo de datos

Se introduce un nuevo array global en `game.js`:

- `explosions`: array de objetos `{ x, y, width, height, color, startTime }`, independiente del array `bricks`.
  - `x, y, width, height`: posición y tamaño del bloque que explotó (se reutilizan los valores del brick original para dibujar el frame en el mismo lugar).
  - `color`: color del bloque, usado para indexar `EXPLOSION_FRAMES[color]`.
  - `startTime`: timestamp (`performance.now()`) capturado en el momento del impacto, usado para calcular qué frame mostrar.

El array `bricks` (de Spec 01) no cambia de forma; al detectar colisión, el brick pasa a `alive = false` igual que antes (deja de colisionar y dibujarse como bloque), y se agrega una entrada nueva a `explosions`.

## Plan de implementación

1. Al detectar colisión pelota-bloque, además de marcar `brick.alive = false` y sumar el `score` (comportamiento ya existente de Spec 01), agregar una entrada a `explosions` con `{ x: brick.x, y: brick.y, width: brick.width, height: brick.height, color: brick.color, startTime: performance.now() }`. Verificable: al romper un bloque, se agrega una entrada al array `explosions` (chequeable con un log temporal o el debugger).
2. En `update()`, recorrer `explosions` y eliminar (filtrar) las que ya superaron `EXPLOSION_DURATION` (150ms) desde su `startTime`. Verificable: las entradas de `explosions` desaparecen del array pasados los 150ms.
3. En `render()`, para cada entrada activa en `explosions`, calcular el índice de frame como `Math.min(3, Math.floor((performance.now() - startTime) / (EXPLOSION_DURATION / 4)))` y dibujarlo con `drawFrame(ctx, EXPLOSION_FRAMES[color][frameIndex], x, y, width, height)`. Verificable: al romper un bloque se ve la secuencia de 4 frames de explosión en su lugar antes de desaparecer.
4. Ajustar la condición de regeneración del grid completo: en lugar de regenerar apenas todos los `bricks` tienen `alive = false`, esperar además a que `explosions.length === 0`. Verificable: al romper el último bloque, el grid no se regenera hasta que termina la animación de explosión de ese último bloque.

## Criterios de aceptación

- [X] Al golpear un bloque, este deja de colisionar con la pelota inmediatamente (la pelota no rebota contra un bloque "en explosión").
- [X] Al golpear un bloque, se reproducen visualmente los 4 frames de su animación de explosión (`EXPLOSION_FRAMES[color]`) en la posición del bloque.
- [X] La animación de un bloque dura aproximadamente 150ms (`EXPLOSION_DURATION`) y luego desaparece por completo de la pantalla.
- [X] El puntaje se suma en el momento del impacto, sin esperar a que termine la animación.
- [X] Si se rompe el último bloque del grid, el grid completo no se regenera hasta que la animación de explosión de ese bloque haya terminado.
- [X] No hay errores de consola ni bloques "fantasma" (bloques que ya no deberían colisionar pero siguen bloqueando la pelota).
- [X] No se modifica `assets/spritesheet.js`; se reutiliza tal cual (`drawFrame`, `EXPLOSION_FRAMES`, `EXPLOSION_DURATION`).

## Decisiones tomadas y descartadas

- **El bloque deja de colisionar de inmediato** (no al terminar la animación): se prioriza la fidelidad a la jugabilidad clásica de Arkanoid, donde el impacto es instantáneo y la explosión es puramente un efecto visual posterior.
- **El puntaje se suma en el momento del impacto**, no al terminar la animación: mantiene el comportamiento ya validado en Spec 01 y evita acoplar la lógica de puntaje a temporizadores visuales.
- **La regeneración del grid espera a que terminen las explosiones activas**: evita que un bloque nuevo aparezca en el mismo lugar donde todavía se está reproduciendo la animación de explosión del bloque anterior, lo cual se vería visualmente inconsistente.
- **Array `explosions` separado** en lugar de campos extra en cada `brick`: mantiene el array `bricks` simple (igual que en Spec 01) y separa claramente el estado "estructural" del grid del estado "efímero" de las animaciones en curso.
- **Sonido fuera de scope**: se mantiene la misma separación que ya estableció Spec 01 entre mecánica visual y audio, dejando `break-sound.mp3` para una spec futura de audio/FX.

## Riesgos identificados

- **Múltiples explosiones simultáneas con distinto `startTime`**: si varios bloques explotan en el mismo frame o en frames cercanos, el array `explosions` crece y se recorre cada frame en `update()` y `render()`. Mitigación: el array se mantiene acotado porque las entradas se eliminan automáticamente pasados los 150ms; con el tamaño del grid de Spec 01 (8x5) el costo es despreciable.
- **Desfasaje entre `performance.now()` y el loop de `requestAnimationFrame`**: en teoría un frame de animación podría saltarse si el navegador tiene un frame drop severo. Mitigación: el cálculo de índice de frame es por tiempo transcurrido (no por conteo de frames de render), por lo que un frame salteado no rompe la sincronización, solo podría saltar directamente al frame 3.
