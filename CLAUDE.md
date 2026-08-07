# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Cómo ejecutar el juego

Sin proceso de build ni dependencias. Dos opciones:

```bash
# Abrir directamente en Windows
start index.html

# Servidor local (recomendado para evitar restricciones de CORS)
python3 -m http.server 8000
# luego abrir http://localhost:8000
```

## Arquitectura

Proyecto de tres archivos sin dependencias externas:

- **`index.html`** — DOM: dos `<canvas>` (`#board` 300×600 px y `#next-canvas` 120×120 px), panel lateral con HUD (score/lines/level) y overlay reutilizable para PAUSA y GAME OVER.
- **`style.css`** — Tema dark/retro. El overlay usa `backdrop-filter: blur` y se muestra/oculta con la clase `.hidden`.
- **`game.js`** — Toda la lógica del juego en un único archivo de ~300 líneas con `'use strict'`.

### Flujo de ejecución (`game.js`)

```
init()
  ├─ createBoard()            → matriz ROWS×COLS con ceros
  ├─ next = randomPiece()
  ├─ spawn()                  → mueve next → current, genera nueva next
  └─ requestAnimationFrame(loop)

loop(timestamp)
  ├─ acumula dropAccum += dt
  ├─ si dropAccum ≥ dropInterval → baja la pieza o llama lockPiece()
  ├─ draw()
  └─ requestAnimationFrame(loop)

lockPiece() → merge() → clearLines() → spawn()
  si spawn() colisiona al aparecer → endGame()
```

### Estado del juego (variables globales en `game.js`)

`board`, `current`, `next`, `score`, `lines`, `level`, `paused`, `gameOver`, `lastTime`, `dropAccum`, `dropInterval`, `animId`

### Constantes ajustables

| Constante      | Valor por defecto | Nota                                                         |
| -------------- | ----------------- | ------------------------------------------------------------ |
| `COLS`         | `10`              | Si se cambia, actualizar `width` del `<canvas id="board">`  |
| `ROWS`         | `20`              | Si se cambia, actualizar `height` del `<canvas id="board">` |
| `BLOCK`        | `30`              | Tamaño en px por celda; afecta canvas dimensions            |
| `LINE_SCORES`  | `[0,100,300,500,800]` | Puntos por 1–4 líneas eliminadas (antes de × nivel)    |

La velocidad de caída se recalcula en `clearLines()`: `dropInterval = Math.max(100, 1000 - (level - 1) * 90)` ms.

### Invariantes clave

- El canvas `#board` debe medir exactamente `COLS × BLOCK` × `ROWS × BLOCK` píxeles.
- Las piezas están indexadas 1–7; el índice 0 en el tablero y en `COLORS`/`PIECES` significa celda vacía.
- `ghostY()` no modifica el estado; solo proyecta la posición final hacia abajo.
- `tryRotate()` implementa wall kicks intentando desplazamientos `[0, -1, 1, -2, 2]` en X antes de descartar el giro.
