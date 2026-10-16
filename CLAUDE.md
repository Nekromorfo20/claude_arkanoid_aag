# CLAUDE.md

Este archivo brinda guía a Claude Code (claude.ai/code) al trabajar con código en este repositorio.

## Estado del proyecto

El juego está **implementado y jugable**: `index.html`, `style.css` y `game.js` (un único archivo monolítico con todo el motor del juego) ya existen en la raíz del repo. Todo lo construido hasta ahora llegó a través del flujo `/spec` → `/spec-impl` descrito abajo — no asumas que hay código fuera de ese flujo, y antes de tocar una feature existente revisá su spec correspondiente en `specs/` para entender el comportamiento esperado.

## Objetivo

Construir un juego de Arkanoid usando HTML, CSS y JavaScript puro — **cero dependencias** (sin frameworks, sin herramientas de build, sin paquetes npm). El juego debe poder jugarse directamente en un navegador.

## Workflow: desarrollo guiado por specs

Este repo usa dos skills personalizadas de slash-command (provenientes de `Klerith/fernando-skills`, registradas en `skills-lock.json`) que definen cómo se planifican y construyen las features. Seguí este workflow en lugar de saltar directo al código:

- **`/spec <descripción>`** — diseñador de specs guiado. Hace preguntas de aclaración por fases y luego escribe un archivo de spec numerado en `specs/NN-slug.md` (ej. `specs/05-slug.md`, siguiendo el número consecutivo más alto en `specs/`). Las specs empiezan en estado `Draft`. Reglas completas: `.agents/skills/spec/SKILL.md`, template: `.agents/skills/spec/template.md`.
- **`/spec-impl <NN-spec-name>`** — implementa una spec aprobada. **Se niega a ejecutar si el estado de la spec no es `Aprobado`** (el humano debe cambiarlo manualmente después de revisarla). Al aprobarse, crea una rama git `spec-NN-slug`, muestra el resumen de la spec, e implementa el plan **paso a paso, pausando para revisión después de cada paso**. Nunca hace commit automáticamente.

Cuando te pidan agregar una feature a este juego, preferí pasar primero por `/spec` en lugar de escribir código directamente, salvo que el usuario pida explícitamente un cambio rápido/descartable.

El comportamiento de creación de ramas de `/spec-impl` se controla desde `specs/.spec-config.yml` (`AutoCreateBranch: true|false`), que `/spec` crea automáticamente la primera vez que hace falta.

Cada spec declara explícitamente de qué specs anteriores depende (`**Depende de:**`) — antes de implementar o modificar una feature, revisá esa cadena de dependencias para no romper comportamiento ya aprobado en specs previas.

### Specs implementadas

Todas en estado `Aprobado` y ya mergeadas en `game.js`:

| Spec | Feature |
|---|---|
| `specs/01-mvp-arkanoid.md` | MVP jugable: paddle (mouse/teclado), pelota con rebote de ángulo variable, grid fijo de bloques, 3 vidas, puntaje. |
| `specs/02-explosion-de-bloques.md` | Animación de explosión de 4 frames al romper un bloque (`drawFrame`/`EXPLOSION_FRAMES`), en vez de desaparición instantánea. |
| `specs/03-niveles-y-sonidos.md` | Progresión de 5 niveles (layout y velocidad propios) y efectos de sonido (`ball-bounce.mp3`, `break-sound.mp3`) en rebotes y roturas. |
| `specs/04-pausa-game-over-victoria.md` | Estado global de juego (`playing`/`paused`/`gameover`/`victory`): pausa con tecla `P`, Game Over al perder la última vida, pantalla de victoria al completar el nivel 5, reinicio con `Enter`. |

## Assets ya disponibles

- `assets/spritesheet-breakout.png` — la imagen del sprite sheet (paddle, pelota, bloques, animaciones de explosión).
- `assets/spritesheet.js` — helper de carga/dibujo del sprite sheet, ya implementado y listo para usar:
  - `loadSpritesheet(cb)` — carga el PNG una sola vez (lo cachea en un canvas offscreen) e invoca `cb` cuando está listo; se puede llamar varias veces sin problema.
  - `drawSprite(ctx, name, x, y, w, h)` — dibuja un sprite por nombre (`'paddle'`, `'ball'`, o `'block_<color>'` ej. `'block_red'`) sobre un contexto de canvas en la posición/tamaño dado.
  - `drawFrame(ctx, frame, x, y, w, h)` — dibuja un objeto de frame crudo (usado para los frames de animación de explosión definidos en `EXPLOSION_FRAMES`).
  - Colores de bloques disponibles: `gray, red, yellow, cyan, magenta, hotpink, green`. Existen animaciones de explosión por color (4 frames cada una, `EXPLOSION_DURATION = 150`ms).
- `assets/sounds/ball-bounce.mp3`, `assets/sounds/break-sound.mp3` — efectos de sonido para rebotes de paddle/pared y rotura de bloques.

Al implementar el juego, reutilizá `spritesheet.js` tal cual en lugar de re-derivar las coordenadas de los sprites — ya tiene codificados los offsets correctos `sx/sy/sw/sh` dentro de la hoja.
