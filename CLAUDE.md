# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Ejecutar el juego

Sin build ni dependencias. Dos opciones:

```bash
# Abrir directamente
start index.html

# Servidor local (recomendado para evitar restricciones CORS)
python3 -m http.server 8000
# → http://localhost:8000
```

No hay tests automatizados; verificar cambios manualmente en el navegador.

## Arquitectura

Tres archivos, toda la lógica en `game.js` (~305 líneas, ES6+ vanilla):

- **`index.html`** — dos `<canvas>`: `#board` (300×600 px, tablero principal) y `#next-canvas` (120×120 px, vista previa). Panel lateral con HUD (`#score`, `#lines`, `#level`). Overlay único para PAUSA y GAME OVER.
- **`style.css`** — dark/retro theme, sin clases dinámicas (el overlay usa `.hidden`).
- **`game.js`** — todo el estado y lógica del juego.

### Flujo principal en `game.js`

| Función | Rol |
|---|---|
| `init()` | Resetea estado, lanza el game loop |
| `loop(ts)` | `requestAnimationFrame`; acumula `dropAccum`, baja pieza o llama `lockPiece()` |
| `collide(shape, ox, oy)` | Detección de colisiones contra bordes y tablero |
| `tryRotate()` | Rotación + wall kicks (offsets `[0, -1, 1, -2, 2]`) |
| `lockPiece()` | `merge()` → `clearLines()` → `spawn()` |
| `spawn()` | Promueve `next` a `current`; si colisiona al aparecer → `endGame()` |
| `ghostY()` | Proyecta posición final hacia abajo |
| `drawBlock(ctx, x, y, colorIndex, size, alpha)` | Primitiva de render compartida entre tablero y vista previa |

### Estado global

```js
let board, current, next, score, lines, level, paused, gameOver, lastTime, dropAccum, dropInterval, animId;
```

`board` es una matriz `ROWS×COLS` donde cada celda vale `0` (vacía) o `1–7` (índice de color/tipo de pieza).

### Constantes ajustables

| Constante | Valor | Nota |
|---|---|---|
| `COLS` / `ROWS` | 10 / 20 | Cambiar también `width`/`height` del canvas en HTML (`COLS×BLOCK`, `ROWS×BLOCK`) |
| `BLOCK` | 30 px | Tamaño de celda |
| `LINE_SCORES` | `[0,100,300,500,800]` | Puntos por 1–4 líneas |
| `dropInterval` | 1000 ms inicial | Velocidad mínima: `max(100, 1000-(level-1)×90)` |
