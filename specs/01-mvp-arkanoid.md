# Spec 01 — MVP Arkanoid

**Estado:** Aprobado
**Depende de:** Ninguna
**Fecha:** 2026-10-08

## Objetivo

Crear un MVP jugable de Arkanoid en HTML/CSS/JS puro, con paddle controlable por mouse o teclado, pelota con rebote de ángulo variable, un grid fijo de bloques multicolor, 3 vidas con reinicio automático al agotarse, y un contador de puntaje, sin pantallas de Game Over ni de victoria.

## Scope

**Incluye:**
- Canvas HTML5 con el loop de juego (`requestAnimationFrame`).
- Paddle controlable con mouse (seguimiento horizontal) y con teclado (flechas izquierda/derecha o A/D).
- Pelota con movimiento continuo, rebote contra paredes (izquierda, derecha, arriba) y contra la paddle.
- Rebote en la paddle con ángulo variable según el punto de impacto (golpear cerca del borde desvía más la pelota).
- Un único grid fijo de bloques (ej. 8 columnas x 5 filas) usando los colores disponibles en el spritesheet (`gray, red, yellow, cyan, magenta, hotpink, green`).
- Sistema de 3 vidas: al caer la pelota por debajo de la paddle, se pierde una vida y la pelota se resetea en posición inicial sobre la paddle.
- Al llegar a 0 vidas: se reinician las vidas a 3 y el juego continúa (sin pantalla de Game Over, sin detener el juego).
- Al romper todos los bloques del grid: se regenera el mismo grid completo y el juego continúa (sin pantalla de victoria).
- Contador de puntaje visible en pantalla, que suma puntos fijos por cada bloque roto.
- Uso de `assets/spritesheet.js` (`loadSpritesheet`, `drawSprite`) para dibujar paddle, pelota y bloques.

**No incluye (fuera de scope de este MVP):**
- Sonidos (`ball-bounce.mp3`, `break-sound.mp3`) — se deja para una spec futura.
- Animación de explosión de bloques (`drawFrame`, `EXPLOSION_FRAMES`) — se deja para una spec futura.
- Múltiples niveles o progresión de dificultad.
- Power-ups.
- Pantallas de Game Over o de Victoria.
- Persistencia de puntaje (high scores) entre sesiones.
- Pausa del juego.

## Modelo de datos

No se introduce persistencia ni estructuras complejas. El estado del juego vive en memoria, en variables/objetos JS dentro de `game.js` durante la sesión de juego:

- `paddle`: `{ x, y, width, height }`
- `ball`: `{ x, y, radius, dx, dy }`
- `bricks`: array de objetos `{ x, y, width, height, color, alive }`, generado a partir de un grid fijo (columnas x filas) al iniciar y al regenerarse.
- `lives`: número entero, inicia en 3.
- `score`: número entero, inicia en 0.

## Plan de implementación

1. Crear `index.html` con el `<canvas>`, el link a `style.css`, y los `<script>` para `assets/spritesheet.js` y `game.js`. Verificable: la página carga y muestra un canvas en blanco sin errores de consola.
2. Crear `style.css` con estilos mínimos (centrado del canvas, fondo oscuro, tamaño fijo del canvas). Verificable: el canvas se ve centrado y con fondo distinguible.
3. En `game.js`, inicializar el loop de juego con `requestAnimationFrame`, cargar el spritesheet con `loadSpritesheet`, y dibujar la paddle estática en su posición inicial. Verificable: se ve la paddle dibujada con el sprite correcto.
4. Agregar control de la paddle por teclado (flechas/A-D) y por mouse (`mousemove` sobre el canvas), con límites para no salir del canvas. Verificable: la paddle se mueve con ambos métodos sin salirse de los bordes.
5. Agregar la pelota: dibujo, movimiento continuo, y rebote contra paredes izquierda/derecha/arriba. Verificable: la pelota rebota visualmente contra las tres paredes.
6. Implementar rebote de la pelota contra la paddle con ángulo variable según punto de impacto. Verificable: golpear cerca de los bordes de la paddle cambia notoriamente el ángulo de salida.
7. Implementar la pérdida de vida: detectar cuando la pelota cae debajo de la paddle, decrementar `lives`, resetear la posición de la pelota. Verificable: al dejar caer la pelota, se descuenta una vida y la pelota reaparece sobre la paddle.
8. Implementar el reinicio de vidas al llegar a 0: volver `lives` a 3 y continuar sin detener el juego. Verificable: tras perder 3 veces seguidas, el contador de vidas vuelve a 3 y el juego sigue jugable.
9. Generar el grid fijo de bloques multicolor usando `drawSprite` con los nombres `block_<color>`, y renderizarlo en pantalla. Verificable: se ven todas las filas/columnas de bloques con sus colores correspondientes.
10. Implementar colisión pelota-bloque: al impactar un bloque, marcarlo como no vivo (dejar de dibujarlo/colisionar), rebotar la pelota, y sumar puntos al `score`. Verificable: los bloques desaparecen al ser golpeados y el puntaje sube.
11. Implementar la regeneración del grid completo al romperse todos los bloques. Verificable: al romper el último bloque, el grid completo reaparece intacto y el juego continúa.
12. Renderizar en pantalla el contador de vidas y el contador de puntaje (texto simple sobre el canvas o HTML superpuesto). Verificable: ambos valores se actualizan en tiempo real durante el juego.

## Criterios de aceptación

- [ ] `index.html` carga sin errores de consola y muestra el canvas con paddle, pelota y bloques.
- [ ] La paddle se controla con flechas/A-D y con el mouse, sin salir de los límites del canvas.
- [ ] La pelota rebota correctamente contra las paredes izquierda, derecha y superior.
- [ ] La pelota rebota contra la paddle con ángulo variable según el punto de impacto.
- [ ] Al caer la pelota debajo de la paddle, se resta una vida y la pelota se resetea sobre la paddle.
- [ ] Al llegar a 0 vidas, el contador vuelve a 3 y el juego continúa sin pantallas ni detenerse.
- [ ] Los bloques se dibujan en un grid fijo con los colores del spritesheet y desaparecen al ser golpeados.
- [ ] El puntaje aumenta con cada bloque roto y se muestra en pantalla.
- [ ] Al romperse todos los bloques, el grid se regenera completo y el juego continúa.
- [ ] No hay dependencias externas (sin frameworks, sin build tools, sin paquetes npm).

## Decisiones tomadas y descartadas

- **Sin sonido ni animación de explosión en este MVP**: se decidió mantener el alcance mínimo funcional primero; `assets/sounds/*` y `drawFrame`/`EXPLOSION_FRAMES` quedan listos para una spec de "FX y pulido" futura.
- **Sin pantallas de Game Over/Victoria**: en lugar de detener el juego, tanto perder las 3 vidas como romper todos los bloques disparan un reinicio silencioso (de vidas o del grid), manteniendo el juego siempre jugable — decisión explícita del usuario para este MVP.
- **Ángulo de rebote variable en la paddle** sobre rebote simple: se prioriza fidelidad a la jugabilidad clásica de Arkanoid, donde el punto de impacto en la paddle es la herramienta principal de control del jugador.
- **Grid único y fijo de bloques** (sin niveles): se descarta progresión de niveles para mantener el MVP acotado; cualquier sistema de niveles queda para una spec futura.
- **Archivos separados** (`index.html` + `game.js` + `style.css`) sobre todo-en-uno: se prioriza legibilidad y mantenibilidad para iteraciones futuras (power-ups, niveles, FX).

## Riesgos identificados

- **Colisión pelota-bloque con múltiples bloques adyacentes**: si la pelota golpea dos bloques en el mismo frame, podría rebotar de forma inconsistente. Mitigación: resolver una sola colisión por frame (la más cercana) y re-evaluar en el siguiente frame.
- **Rebote en ángulo extremo en la paddle**: un ángulo demasiado horizontal podría hacer que la pelota quede "rebotando" sin ganar altura. Mitigación: acotar el ángulo máximo de salida a un rango razonable (ej. ±60°).
