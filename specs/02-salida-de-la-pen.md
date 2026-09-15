# SPEC 02 — Salida de la pen: rebote y ruta fija hacia el pasillo

> **Status:** Draft
> **Depends on:** SPEC 01
> **Date:** 2026-09-14
> **Objective:** Que los fantasmas no queden atrapados en la pen: rebotan de lado a lado mientras esperan su turno y, al pasar su delay, salen por la puerta subiendo en línea recta hasta el pasillo antes de perseguir.

## Scope

**In:**

- Rebote horizontal (ping-pong) dentro de la pen para los 4 fantasmas desde el inicio de la partida, confinado a las columnas 11–16 y filas 13–15.
- El `delay` deja de congelar al fantasma y pasa a controlar cuándo empieza a salir.
- Modo salida: el fantasma se dirige con la greedy existente hacia un objetivo fijo `GHOST_EXIT = { x: 13, y: 11 }` (pasillo sobre la puerta); al alinearse ahí pasa a persecución normal.
- La puerta (celda `3`) bloquea a todo fantasma que no esté en salida: un fantasma ya fuera no vuelve a entrar a la pen.
- `resetPositions()` resetea `mode` y `bounceDir`: al perder vida se repite el rebote y el escalonado 0/3/6/9 s.

**Out of scope (para futuras specs):**

- Power pellets / estado asustado (frightened).
- Animación de reintroducción a la pen tras la muerte (sigue reiniciando el escalonado sin pausa de vuelta).
- Velocidades distintas por fantasma, scatter, fruta o niveles.

## Data model

```js
// maze.js — constante nueva junto a GHOST_STARTS
const GHOST_EXIT = { x: 13, y: 11 }; // pasillo justo encima de la puerta
const PEN_X = { min: 11, max: 16 };   // rebote confinado a estas columnas
```

```js
// game.js — estado de cada fantasma en createGame()
{
  x, y, dir: 'up', speed: GHOST_SPEED,
  kind: 'blinky', delay: 0,
  mode: 'wait',          // 'wait' | 'leaving' | 'chase'
  bounceDir: 'right',    // horizontal mientras rebota
}
```

Convenciones:

- `mode` guía todo el movimiento: `wait` rebota en la pen, `leaving` sube a `GHOST_EXIT`, `chase` usa `ghostTarget` por persona (sin cambios).
- `bounceDir` arranca hacia el centro del pen (columnas 13/14): `'right'` para `x <= 13`, `'left'` para `x >= 14`.
- Un fantasma cambia a `leaving` en cuanto `game.time >= g.delay`, desde la columna donde esté rebotando; la greedy hacia `GHOST_EXIT` funciona desde cualquier celda del pen.
- Al llegar alineado a `GHOST_EXIT`, pasa a `chase`.
- La puerta es transitable solo cuando `g.mode === 'leaving'`; en `wait`/`chase` es muro.

## Implementation plan

1. `src/js/maze.js`: añadir `GHOST_EXIT` y `PEN_X` junto a `GHOST_STARTS`. Manual: el juego sigue funcionando igual (aún sin rebote).
2. `src/js/game.js` (`createGame`): añadir `mode: 'wait'` y `bounceDir` a cada fantasma. Manual: comportamiento idéntico hasta el resto de pasos.
3. `src/js/game.js`: nueva `bounceGhost(game, g)` que mueve en horizontal a `GHOST_SPEED` y revierte `bounceDir` al llegar a `PEN_X.min`/`max`. `update` llama a `bounceGhost` para cada fantasma en `mode === 'wait'` (ya no depende del delay) y cambia a `leaving` cuando `game.time >= g.delay`. Manual: los 4 rebotan desde el inicio sin salir.
4. `src/js/game.js`: regla de puerta en `isWall`/`canMove` — la celda `3` es muro para fantasmas salvo si `mode === 'leaving'`; y rama en `ghostTarget` que devuelve `GHOST_EXIT` cuando `mode === 'leaving'`; al alinearse en `GHOST_EXIT`, pasar a `chase`. Manual: Blinky sale al ~0 s, luego Pinky ~3 s, Inky ~6 s, Clyde ~9 s; ninguno vuelve a entrar.
5. `src/js/game.js` (`resetPositions`): restablecer `mode: 'wait'` y `bounceDir` inicial. Manual: al perder vida se repite el rebote y el escalonado.

## Acceptance criteria

- [ ] Al arrancar la partida, los 4 fantasmas rebotan de lado a lado entre las columnas 11 y 16, filas 13-15, sin cruzar la puerta nunca.
- [ ] Blinky sale a ~0 s, Pinky ~3 s, Inky ~6 s y Clyde ~9 s: suben por la puerta a la fila 11 y pasan a perseguir.
- [ ] Un fantasma en `chase` nunca vuelve a entrar a la pen por la puerta.
- [ ] Al perder una vida, los 4 vuelven a rebotar y el escalonado de salida se repite.
- [ ] La consola del navegador no muestra errores durante el juego.

## Decisions

- **Sí:** mantener los 4 dentro de la pen (decisión de SPEC 01). Esta spec solo arregla la salida.
- **Sí:** salida con ruta fija hacia arriba vía la greedy ya existente hacia `GHOST_EXIT`. Funciona desde cualquier columna del rebote y reutiliza `decideGhost`.
- **No:** teletransporte a `(13,11)` al pasar el delay. Se vería un "salto"; la ruta es más natural.
- **Sí:** rebote ping-pong horizontal desde el inicio de la partida. Cambia SPEC 01 (estáticos hasta su delay); `delay` ahora solo dispara la salida.
- **Sí:** puerta direccional — transitable solo en `mode === 'leaving'`. Bloquea reentrada sin tocar la salida.
- **No (diferido):** scatter (instinto por cuadrante), velocidades por fantasma, reintroducción animada a la pen.

## Risks

| Riesgo | Mitigación |
| --- | --- |
| El rebote llega a la puerta por estar en columnas 13-14 | El ping-pong es solo horizontal en filas 13-15; la puerta (fila 12) no se alcanza en `wait` |
| Un fantasma en `leaving` desde col 11/16 (extremos) | La greedy hacia `GHOST_EXIT` navega el rectángulo abierto de la pen; trazado manualmente |
| Ramas de `canMove` si se pasa el objeto en vez del actor | `pacman` sigue siendo string; los fantasmas pasan el objeto con `mode` |
| Un `dt` enorme tras pestaña inactiva libera fantasmas de golpe | El clamp `Math.min(dt, 0.1)` de `main.js` ya existe (SPEC 01) |

## What is **not** in this spec

- Power pellets / estado asustado (frightened).
- Scatter (instinto de cuadrante) y velocidades por fantasma.
- Animación de reintroducción a la pen tras la muerte.
- Fruta, niveles o cambios de velocidad por nivel.

Cada una, si llega, irá en su propia spec.