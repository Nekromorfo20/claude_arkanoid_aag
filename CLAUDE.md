# CLAUDE.md

Este archivo brinda guía a Claude Code (claude.ai/code) al trabajar con código en este repositorio.

## Estado del proyecto

Este es un proyecto **sin implementar**. Todavía no existe código del juego — solo hay un README, un pipeline de assets de sprites, y una configuración de workflow guiado por specs. No asumas que existen archivos HTML/CSS/JS del juego hasta que se creen mediante el flujo `/spec` → `/spec-impl` descrito abajo.

## Objetivo

Construir un juego de Arkanoid usando HTML, CSS y JavaScript puro — **cero dependencias** (sin frameworks, sin herramientas de build, sin paquetes npm). El juego debe poder jugarse directamente en un navegador.

## Workflow: desarrollo guiado por specs

Este repo usa dos skills personalizadas de slash-command (provenientes de `Klerith/fernando-skills`, registradas en `skills-lock.json`) que definen cómo se planifican y construyen las features. Seguí este workflow en lugar de saltar directo al código:

- **`/spec <descripción>`** — diseñador de specs guiado. Hace preguntas de aclaración por fases y luego escribe un archivo de spec numerado en `specs/NN-slug.md` (ej. `specs/01-mvp-arkanoid.md`). Las specs empiezan en estado `Draft`. Reglas completas: `.agents/skills/spec/SKILL.md`, template: `.agents/skills/spec/template.md`.
- **`/spec-impl <NN-spec-name>`** — implementa una spec aprobada. **Se niega a ejecutar si el estado de la spec no es `Approved`** (el humano debe cambiarlo manualmente después de revisarla). Al aprobarse, crea una rama git `spec-NN-slug`, muestra el resumen de la spec, e implementa el plan **paso a paso, pausando para revisión después de cada paso**. Nunca hace commit automáticamente.

Cuando te pidan agregar una feature a este juego, preferí pasar primero por `/spec` en lugar de escribir código directamente, salvo que el usuario pida explícitamente un cambio rápido/descartable.

El comportamiento de creación de ramas de `/spec-impl` se controla desde `specs/.spec-config.yml` (`AutoCreateBranch: true|false`), que `/spec` crea automáticamente la primera vez que hace falta.

## Assets ya disponibles

- `assets/spritesheet-breakout.png` — la imagen del sprite sheet (paddle, pelota, bloques, animaciones de explosión).
- `assets/spritesheet.js` — helper de carga/dibujo del sprite sheet, ya implementado y listo para usar:
  - `loadSpritesheet(cb)` — carga el PNG una sola vez (lo cachea en un canvas offscreen) e invoca `cb` cuando está listo; se puede llamar varias veces sin problema.
  - `drawSprite(ctx, name, x, y, w, h)` — dibuja un sprite por nombre (`'paddle'`, `'ball'`, o `'block_<color>'` ej. `'block_red'`) sobre un contexto de canvas en la posición/tamaño dado.
  - `drawFrame(ctx, frame, x, y, w, h)` — dibuja un objeto de frame crudo (usado para los frames de animación de explosión definidos en `EXPLOSION_FRAMES`).
  - Colores de bloques disponibles: `gray, red, yellow, cyan, magenta, hotpink, green`. Existen animaciones de explosión por color (4 frames cada una, `EXPLOSION_DURATION = 150`ms).
- `assets/sounds/ball-bounce.mp3`, `assets/sounds/break-sound.mp3` — efectos de sonido para rebotes de paddle/pared y rotura de bloques.

Al implementar el juego, reutilizá `spritesheet.js` tal cual en lugar de re-derivar las coordenadas de los sprites — ya tiene codificados los offsets correctos `sx/sy/sw/sh` dentro de la hoja.
