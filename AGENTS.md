# AGENTS.md

## Estructura

Juego HTML5 canvas puro. Cuatro archivos versionados y nada más: `index.html`,
`game.js`, `favicon.svg`, `README.md`. Sin `package.json`, sin bundler, sin
dependencias, sin `.gitignore`, sin build, test, lint, typecheck, codegen ni CI.
No inventes scripts ni tooling.

- `index.html`: punto de entrada. CSS inline en un `<style>` y un único
  `<script src="game.js">` **clásico** (sin `type="module"`) al cierre de `<body>`.
- `game.js`: todo el juego en un archivo con `'use strict'` al tope (game.js:1).
  Clases `Bullet`, `Asteroid`, `Ship`, `Particle`; estado global suelto
  (`ship`, `bullets`, `asteroids`, `particles`, `score`, `lives`, `level`,
  `state`); funciones `initGame`, `nextLevel`, `explode`, `killShip`,
  `update(dt)`, `draw()` y `loop(ts)`.
- Cada `draw()` pinta directo sobre el `ctx` único del canvas. No hay capas,
  assets ni segundo contexto: la lógica y el render están entrelazados.
- Repo propio (`github.com/JoaquinFelix/opencode-asteroids`), un solo commit,
  sin CI ni releases. Los proyectos hermanos del curso son repos separados.

## Ejecutar

- Abre `index.html` con doble clic. Funciona por `file://` **justo porque** el
  script es clásico y no hay `fetch` ni módulos.
- Si lo conviertes a ES modules o agregás un `fetch`, `file://` deja de servir:
  pasá a servidor (`npx serve .` → puerto 3000).

## Geometría del canvas (duplicada)

800x600 está en dos lugares: los atributos del `<canvas>` (index.html:23) y
`const W`/`const H` (game.js:5-6). `game.js` **no** lee el canvas, así que
cambiá los dos o el render se descentra. No hay resize ni `devicePixelRatio`.

## Constantes de física: inline, sin bloque central

No existe `config` ni `constants`. El tuning vive donde se usa:
`SPEED = 520` en el constructor de `Bullet` (game.js:37);
`ROT`/`THRUST`/`DRAG` dentro de `Ship.update` (game.js:143-145);
`NOSE = 21` en `tryShoot` (game.js:165);
`SAFE_DIST = 130` en `spawnAsteroids` (game.js:245).
Tunear = editar donde está; si centralizás, mantené los nombres.

`RADII`/`SPEEDS`/`POINTS` (game.js:61-63) se indexan por tamaño 1..3 y el
índice 0 es relleno. Agregar un tamaño = tocar las tres tablas más `split()`.

## Input

Dos capas: `keys` (mantenida) y `justPressed` (flanco). `pressed(code)`
(game.js:20) es **destructiva**: consume el flag, así que la segunda llamada en
el mismo frame devuelve `false`. Llamala una sola vez por tecla por frame.
El autofire del navegador no dispara shots porque
`justPressed[e.code] = !keys[e.code]`.

## Ciclo de vida de entidades

Cada entidad lleva `dead` y los arrays se filtran al final de `update`. Las
colisiones bala/asteroide recolectan en `newAsteroids` y concatenan después
(game.js:324-336): no hagas `splice` ni borres mientras iterás.

## Wrap, dt y bootstrap

- `wrap()` (game.js:27) hace el espacio toroidal y lo usan `Bullet`, `Asteroid`
  y `Ship`. `Particle` **no** lo usa a propósito (explosiones cortas que salen
  del canvas). No lo "arregles".
- `loop` clampa `dt` a 0.05 (game.js:415) para que un cambio de pestaña no
  provoque tunneling. Mantenelo.
- Al final del archivo corren `initGame()` y `requestAnimationFrame(loop)`
  (game.js:422-423): importar `game.js` en un test arranca el juego y el loop.

## Convenciones

- Identificadores y código en inglés; comentarios y README en español.
- Los textos de UI mezclan inglés (`SCORE`, `GAME OVER`) y español (`NIVEL`,
  `PUNTAJE`, `ESPACIO PARA REINICIAR`). No los normalices sin preguntar.

## Verificación

- Único chequeo automático posible: `node --check game.js` (solo sintaxis).
- Después de eso, revisión visual en el navegador. No declares "corregido"
  sin haberlo ejecutado.
