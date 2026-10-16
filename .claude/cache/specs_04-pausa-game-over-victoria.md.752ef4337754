# Spec 04 — Pausa, Game Over y Victoria

**Estado:** Draft
**Depende de:** SPEC 01, SPEC 02, SPEC 03
**Fecha:** 2026-10-16

## Objetivo

Agregar un sistema de estados de juego (`playing`, `paused`, `gameover`, `victory`) que permita pausar/reanudar con la tecla P, termine el juego al perder la última vida en lugar de reiniciar vidas, y muestre una pantalla de victoria al completar el nivel 5 en lugar de hacer loop infinito, ambas pantallas con opción de reinicio total vía la tecla Enter.

## Scope

**Incluye:**
- Un estado global de juego (`gameState`) con cuatro valores posibles: `'playing'`, `'paused'`, `'gameover'`, `'victory'`.
- **Pausa:** la tecla `P` alterna entre `'playing'` y `'paused'`. Mientras está en `'paused'`:
  - `update()` no se ejecuta (pelota, paddle, explosiones y colisiones quedan congeladas).
  - El `mousemove` sobre el canvas no mueve la paddle (el listener verifica el estado antes de actuar).
  - Se dibuja un overlay semitransparente sobre el tablero (que sigue visible de fondo, congelado) con el texto `PAUSA`.
  - `P` solo tiene efecto cuando `gameState` es `'playing'` o `'paused'` (no hace nada en `'gameover'` ni `'victory'`).
- **Game Over:** se elimina el reinicio automático de vidas a 3 (Spec 01). Al llegar `lives` a 0, `gameState` pasa a `'gameover'` y el loop dejar de actualizar el juego (mismo congelamiento que en pausa). Se muestra una pantalla con el texto `GAME OVER`, el puntaje final (`Puntaje: N`) y la instrucción `Presioná Enter para reiniciar`.
- **Victoria:** se elimina el loop infinito de niveles (Spec 03). Al completar el nivel 5 (todos los bloques rotos y `explosions.length === 0`, mismo trigger que ya usa Spec 03 para avanzar de nivel), en lugar de volver al nivel 1, `gameState` pasa a `'victory'`. Se muestra una pantalla con el texto `VICTORIA`, el puntaje final (`Puntaje: N`) y la instrucción `Presioná Enter para reiniciar`.
- **Reinicio:** con `gameState` en `'gameover'` o `'victory'`, presionar `Enter` reinicia todo el estado del juego a sus valores iniciales (`lives = 3`, `score = 0`, `currentLevel = 1`, `bricks` regenerados desde `LEVELS[0]`, `explosions = []`, pelota y paddle en posición inicial) y vuelve `gameState` a `'playing'`.
- Sin sonido nuevo al entrar a Game Over o Victoria (silencio, sin agregar archivos de audio).

**No incluye (fuera de scope de esta spec):**
- Sonidos para pausa, Game Over o Victoria.
- Animaciones de transición entre estados (el cambio de pantalla es instantáneo).
- Persistencia de puntaje o high scores entre sesiones/partidas.
- Botón visual de pausa/reinicio (solo teclado: `P` y `Enter`).
- Cualquier cambio a la mecánica de niveles, sonidos de juego, o explosión de bloques ya definida en Spec 02/03.
- Dificultad o ajustes de balance (número de vidas, número de niveles, fórmula de velocidad).

## Modelo de datos

Se introduce una variable global nueva en `game.js`:

- `gameState`: string, uno de `'playing'`, `'paused'`, `'gameover'`, `'victory'`. Inicia en `'playing'`.

No se agregan estructuras nuevas más allá de esto. `lives`, `score`, `currentLevel`, `bricks`, `explosions`, `ball` y `paddle` (de Spec 01/02/03) se reutilizan tal cual; esta spec solo cambia qué ocurre cuando `lives` llega a 0 y cuando se completa el nivel 5, y agrega una función de reinicio total que reutiliza la lógica de inicialización ya existente (`createBricks`, `resetBall`, reseteo de `paddle.x`).

## Plan de implementación

1. Agregar la variable `gameState` inicializada en `'playing'`, y un listener de `keydown` para la tecla `P` que alterne `gameState` entre `'playing'` y `'paused'` solo si el valor actual es uno de esos dos. Verificable: no hay errores de consola y el valor de `gameState` cambia correctamente (chequeable con un log temporal o el debugger).
2. En `update()`, agregar un guard al inicio que retorne inmediatamente si `gameState !== 'playing'`. Verificable: al pausar, la pelota, la paddle y las explosiones dejan de moverse visualmente.
3. En el listener de `mousemove` del canvas, agregar la misma condición (`gameState === 'playing'`) antes de mover la paddle. Verificable: mover el mouse sobre el canvas mientras está en pausa no mueve la paddle.
4. En `render()`, si `gameState === 'paused'`, dibujar un overlay semitransparente (rectángulo con alpha sobre todo el canvas) y el texto `PAUSA` centrado, después de dibujar el resto del tablero. Verificable: al pausar se ve el tablero congelado de fondo con el overlay y el texto superpuestos.
5. Modificar `updateBall()`: donde hoy se hace `if (lives <= 0) lives = 3;` tras decrementar `lives`, cambiarlo para que, si `lives <= 0`, en lugar de reiniciar vidas, setee `gameState = 'gameover'` y no llame a `resetBall()`. Verificable: al perder la última vida, el juego se congela y `gameState` pasa a `'gameover'`.
6. Modificar `updateExplosions()`: donde hoy se hace `currentLevel = currentLevel >= LEVELS.length ? 1 : currentLevel + 1;`, cambiarlo para que, si `currentLevel >= LEVELS.length`, en lugar de volver a 1 y regenerar bricks, setee `gameState = 'victory'` (sin regenerar bricks ni incrementar nivel). Si `currentLevel < LEVELS.length`, el comportamiento de avanzar de nivel se mantiene igual que en Spec 03. Verificable: al completar el nivel 5, el juego se congela y `gameState` pasa a `'victory'`.
7. En `render()`, si `gameState === 'gameover'` o `gameState === 'victory'`, dibujar una pantalla (fondo sólido o semitransparente sobre el canvas) con el texto correspondiente (`GAME OVER` o `VICTORIA`), el puntaje final (`Puntaje: ${score}`) y el texto `Presioná Enter para reiniciar`, en lugar de (o además de) el tablero. Verificable: cada pantalla muestra el mensaje correcto y el puntaje final alcanzado.
8. Crear una función `restartGame()` que resetee `lives = 3`, `score = 0`, `currentLevel = 1`, llame a `createBricks()`, vacíe `explosions`, llame a `resetBall()`, reposicione `paddle.x` a su valor inicial centrado, y finalmente setee `gameState = 'playing'`. Verificable: invocar la función manualmente desde consola deja el juego en el mismo estado que al cargar la página por primera vez.
9. Agregar un listener de `keydown` para la tecla `Enter` que llame a `restartGame()` únicamente si `gameState === 'gameover'` o `gameState === 'victory'`. Verificable: presionar Enter en esas pantallas reinicia el juego completo y vuelve a verse jugable desde el nivel 1 con 3 vidas y puntaje 0.

## Criterios de aceptación

- [ ] Presionar `P` durante el juego activo pausa el juego: la pelota, la paddle y las explosiones dejan de moverse.
- [ ] Mientras está pausado, mover el mouse sobre el canvas no mueve la paddle.
- [ ] Mientras está pausado, se ve un overlay semitransparente con el texto `PAUSA`, con el tablero congelado visible de fondo.
- [ ] Presionar `P` de nuevo estando en pausa reanuda el juego exactamente donde quedó (misma posición y velocidad de la pelota).
- [ ] Al perder la última vida (`lives` llega a 0), el juego se detiene y se muestra la pantalla `GAME OVER` con el puntaje final.
- [ ] Al llegar a `GAME OVER`, las vidas ya no se reinician automáticamente a 3 (se reemplaza el comportamiento de Spec 01).
- [ ] Al romper todos los bloques del nivel 5 (y terminar sus explosiones activas), el juego se detiene y se muestra la pantalla `VICTORIA` con el puntaje final, en lugar de volver al nivel 1.
- [ ] Presionar `Enter` en la pantalla de `GAME OVER` o `VICTORIA` reinicia el juego completo: vidas en 3, puntaje en 0, nivel 1, grid regenerado, pelota y paddle en posición inicial, y el juego vuelve a ser jugable.
- [ ] La tecla `P` no tiene ningún efecto mientras `gameState` es `'gameover'` o `'victory'`.
- [ ] La tecla `Enter` no tiene ningún efecto mientras `gameState` es `'playing'` o `'paused'`.
- [ ] No hay errores de consola en ningún estado ni transición entre estados.
- [ ] No hay dependencias externas (sin frameworks, sin build tools, sin paquetes npm).

## Decisiones tomadas y descartadas

- **Se revierte la filosofía de "el juego nunca se detiene" de Spec 01/03**: ambas specs anteriores decidieron explícitamente evitar pantallas de fin de juego; esta spec las introduce por pedido explícito del usuario, reemplazando el reinicio automático de vidas y el loop infinito de niveles.
- **Cuatro estados discretos (`playing`/`paused`/`gameover`/`victory`) en una sola variable** en lugar de flags booleanos separados (`isPaused`, `isGameOver`, etc.): evita estados inconsistentes (ej. pausado y game-over a la vez) y simplifica los guards en `update()`/`render()`.
- **Pausa solo disponible durante `'playing'`**: no tiene sentido pausar una pantalla de fin de juego que ya está congelada; mantiene la máquina de estados simple.
- **Overlay semitransparente en pausa (se ve el tablero de fondo)** en lugar de una pantalla opaca: decisión explícita del usuario, da feedback visual de en qué momento exacto del juego se pausó.
- **Reinicio total con Enter** en lugar de recargar la página o un botón: dado que ya existe manejo de teclado en el juego (Spec 01), agregar un listener más es consistente con el resto de la arquitectura y evita depender de UI HTML adicional.
- **Mensaje de Game Over/Victoria incluye el puntaje final**: útil para que el jugador vea su resultado sin tener que recordarlo del HUD antes de que se congelara.
- **Sin sonido nuevo en las transiciones de fin de juego**: mantiene el scope acotado; se puede agregar en una spec futura de audio/FX si se desea.
- **El avance de nivel 1 a 4 no cambia respecto a Spec 03**: solo se modifica el caso límite de completar el nivel 5, que pasa de "volver al nivel 1" a "disparar victoria".

## Riesgos identificados

- **Listener de `Enter` también podría interferir con otros usos del teclado**: si en el futuro `Enter` se usa para otra acción durante el juego activo, podría chocar con el reinicio. Mitigación: el listener solo actúa cuando `gameState` es `'gameover'` o `'victory'`, por lo que no afecta el juego activo.
- **Congelar `update()` completamente en pausa podría dejar sonidos de rebote/rotura a mitad de reproducción si se pausa justo en el instante del impacto**: dado que los sonidos se disparan con `new Audio(...).play()` de forma disparar-y-olvidar (Spec 03), seguirán reproduciéndose hasta terminar aunque el juego se pause justo después. Mitigación: se acepta como comportamiento esperado, ya que no afecta la jugabilidad ni dura más de un par de segundos.
- **Pantallas de Game Over/Victoria dibujadas directamente sobre el canvas (sin HTML superpuesto)**: mantiene consistencia con el resto del HUD (Spec 01/03, que ya usa `ctx.fillText`), evitando mezclar renderizado de canvas con overlays HTML.
