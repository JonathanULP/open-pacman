# SPEC 01 — Cuatro fantasmas con IA propia

> **Status:** Approved
> **Depends on:** Ninguna (primera spec del repo)
> **Date:** 2026-09-14
> **Objective:** Añadir cuatro fantasmas, cada uno con su propia forma de perseguir a Pacman — Blinky, Pinky, Inky y Clyde — con salida de la pen escalonada por tiempo.

## Scope

**In:**

- Cuatro fantasmas con IA propia: Blinky (persigue la celda actual de Pacman), Pinky (apunta 4 celdas por delante de Pacman), Inky (emboscada usando la posición de Blinky) y Clyde (persigue de lejos, se retira de cerca).
- Sustituir los `kind` actuales `'hunter'` y `'random'` por `'blinky'`, `'pinky'`, `'inky'` y `'clyde'`.
- Los 4 inician dentro de la pen y salen escalonados por tiempo (0 s, 3 s, 6 s, 9 s).
- Render coloreado por persona (rojo, rosa, cian, naranja).
- El bucle pasa segundos reales (`dt`) a `update`.

**Out of scope (para futuras specs):**

- Power pellets / estado "asustado" (frightened). No existen power-ups hoy.
- Colisiones entre fantasmas; se siguen atravesando.
- Animación de rebote en la pen mientras esperan su turno (se quedan estáticos).
- Reintroducción a la pen tras una muerte; la muerte reinicia el escalonado.
- Velocidades distintas por fantasma y niveles.

## Data model

```js
// maze.js — 4 salidas, una por persona
const GHOST_STARTS = [
  { x: 13, y: 13, kind: 'blinky', delay: 0 },
  { x: 14, y: 13, kind: 'pinky', delay: 3 },
  { x: 13, y: 15, kind: 'inky', delay: 6 },
  { x: 14, y: 15, kind: 'clyde', delay: 9 },
];
```

```js
// game.js — estado de cada fantasma en createGame()
{ x, y, dir: 'up', speed: GHOST_SPEED, kind: 'blinky', delay: 0 }
// plus: game.time  (se acumula con dt; un fantasma se mueve solo si time >= delay)
```

Convenciones:

- `delay` en segundos. `game.time` acumula los `dt` que `main.js` pasa a `update(game, dt)`.
- Objetivo de persecución (tiles): Blinky = celda de Pacman; Pinky = celda de Pacman + 4 × su dirección (clamp a bordes del laberinto); Inky = `2 × (Pacman + 2 × dir) − Blinky`; Clyde = celda de Pacman si distancia Manhattan a él > 8, si no la esquina `{ x: 1, y: 28 }`.
- Decisión en cada cruce: entre las direcciones válidas no-reversa, la que minimiza la distancia Manhattan a su objetivo (reemplaza la rama `hunter`/`random` actual).
- `resetPositions()` pone `game.time = 0`: el escalonado se repite al perder vida.

## Implementation plan

1. `src/js/maze.js`: ampliar `GHOST_STARTS` a las 4 personas con `kind` y `delay`. Manual: siguen viéndose fantasmas (temporalmente con IA anterior).
2. `src/js/render.js`: colorear por `kind` (mapa `kind → color`) en lugar de por índice. Manual: 4 fantasmas rojo, rosa, cian y naranja.
3. `src/js/game.js` + `src/js/main.js`: añadir `game.time`, copiar `delay` en `createGame`, **actualizar `update(game)` → `update(game, dt)`** en ambos, y saltar el movimiento del fantasma hasta `game.time >= g.delay`; `resetPositions()` reinicia `game.time`. Acotar `dt` con `Math.min(dt, 0.1)`. Manual: al arrancar sale solo Blinky; cada ~3 s sale el siguiente.
4. `src/js/game.js`: en `decideGhost`, sustituir las ramas `hunter`/`random` por `ghostTarget(game, g)` por persona + elección greedy por Manhattan. Manual: observar los 4 comportamientos en juego.

## Acceptance criteria

- [ ] Al cargar la partida se ven 4 fantasmas con colores rojo, rosa, cian y naranja.
- [ ] Al empezar, solo Blinky se mueve; Pinky sale a los ~3 s, Inky a los ~6 s y Clyde a los ~9 s.
- [ ] En cada cruce, Blinky elige la opción que reduce la distancia Manhattan a la celda actual de Pacman.
- [ ] Con Pacman en línea recta, Pinky converge a 4 celdas por delante según su dirección.
- [ ] El objetivo de Inky verifica `2 × (Pacman + 2 × dir) − Blinky`.
- [ ] Clyde persigue mientras esté a más de 8 celdas (Manhattan) de Pacman y se dirige a la esquina (1, 28) a 8 o menos.
- [ ] Al perder una vida, los 4 vuelven a sus posiciones iniciales y el escalonado se repite.
- [ ] La consola del navegador no muestra errores durante el juego.

## Decisions

- **Sí:** personas clásicas del arcade. Estándar de Pac-Man, ideal para aprendizaje.
- **No:** conservar `'hunter'`/`'random'`. Son un boceto; las 4 personas los absorben.
- **Sí:** los 4 dentro de la pen (frente a Blinky fuera). Decisión del usuario; más simple.
- **Sí:** escalonado por tiempo 0/3/6/9 s. Decisión del usuario; el clásico "por dots" (30/60) queda fuera por añadir estado extra.
- **Sí:** distancia Manhattan para todos, incluyendo el umbral de Clyde (8). Consistente con el `hunter` actual y más simple que Euclidiana.
- **Sí:** clamp de objetivos fuera del laberinto a los bordes (caso Pinky/Inky).
- **Sí:** `update(game, dt)` con segundos reales; el escalonado necesita tiempo real.
- **No:** rebote animado en la pen ni reintroducción tras muerte. Van a otra spec si llegan.

## Risks

| Riesgo | Mitigación |
| --- | --- |
| Un `dt` enorme tras pestaña inactiva libera varios fantasmas de golpe | Acotar `Math.min(dt, 0.1)` en `main.js` |
| Cambio de firma de `update` si se actualiza solo un llamador | Se tocan `game.js` y `main.js` en el mismo paso |
| Pen congestionada al liberarse 2 fantasmas juntos | Los fantasmas se atraviesan entre sí (comportamiento actual); no hay atasco real |

## What is **not** in this spec

- Power pellets / estado asustado (frightened).
- Colisiones entre fantasmas.
- Animación de rebote en la pen.
- Reintroducción a la pen tras la muerte.
- Velocidades por fantasma, fruta o niveles.

Cada una, si llega, irá en su propia spec.