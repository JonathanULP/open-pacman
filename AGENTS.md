# AGENTS.md

Vanilla JS/HTML/CSS Pac-Man clone. No build system, no framework, no tests, no lint. Purpose is learning Spec-Driven Development (see README).

## Run / verify
- No dev server or tooling. Run by opening `src/index.html` in a browser; verify manually in `#game` canvas.

## Architecture gotchas
- **Script load order in `src/index.html` is a hard dependency.** Files share state via `window.*` globals (no ES modules): `js/maze.js` → `js/game.js` → `js/render.js` → `js/main.js`. Add any new JS file with a `<script>` tag in the correct order after its dependencies. `render.js` uses `DIRS` from `game.js`; `main.js` calls `createGame`/`update` from `game.js` and `draw` from `render.js`.
- `MAZE` in `maze.js` is the pristine matrix, never mutated. `createGame()` (game.js) deep-copies it into `game.grid`; eating dots writes to `game.grid` so restarts work. Render reads `game.grid`, not `MAZE`.
- Grid legend: `1` wall, `2` dot, `3` pen door, `0` empty. In `isWall()` (game.js) doors block pacman but not ghosts.
- Tile geometry is coupled in three places: maze is 28×31, `TILE = 20` in `render.js`, canvas is hardcoded `560×620` in `index.html`. Change all three together.
- Comments and UI text are written in Spanish; keep new comments/docs/specs in Spanish (code identifiers stay in English).

## Spec-driven workflow
- New features go through the `/spec` skill (pinned in `skills-lock.json`, source at `.agents/skills/spec/SKILL.md`): it writes to `specs/NN-slug.md` (zero-padded, sequential).
- No `specs/` folder or `specs/.spec-config.yml` yet — create on first use; config seeds `AutoCreateBranch: true`, but this repo is **not a git repo**, so skip/disable branch creation.
- Follow the template `.agents/skills/spec/template.md`; spec state defaults to `Draft` until the user approves.