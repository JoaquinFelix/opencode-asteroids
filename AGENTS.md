# AGENTS.md

## Estructura

Juego HTML5 canvas puro. Cuatro archivos versionados y nada más: `index.html`,
`game.js`, `favicon.svg`, `README.md`. Sin `package.json`, sin bundler, sin
dependencias, sin `.gitignore`, sin build, test, lint, typecheck, codegen ni CI.
No inventes scripts ni tooling.

- `index.html`: punto de entrada. CSS inline en un `<style>` y un único
  `<script src="game.js">` **clásico** (sin `type="module"`) al cierre de `<body>`.
- `game.js`: todo el juego en un archivo con `'use strict'` al tope (game.js:1).
  Clases `Bullet`, `Asteroid`, `Ship`, `Pickup`, `Particle`; estado global suelto
  (`ship`, `bullets`, `asteroids`, `particles`, `pickups`, `score`, `lives`,
  `level`, `state`); funciones `initGame`, `nextLevel`, `spawnPickup`,
  `updatePickups`, `explode`, `killShip`, `update(dt)`, `draw()` y `loop(ts)`.
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
`SPEED = 520` en el constructor de `Bullet` (game.js:49);
`ROT`/`THRUST`/`DRAG` dentro de `Ship.update` (game.js:171-173);
`NOSE = 21` y `TRIPLE_SPREAD = 0.15` en `tryShoot` (game.js:194-195);
`SAFE_DIST = 130` con `randomSafePoint()` en Utils (game.js:33-42), compartido
por `spawnAsteroids` y `spawnPickup`.
Tunear = editar donde está; si centralizás, mantené los nombres.

Único grupo de constantes aislado: `POWERUPS` (game.js:137-140), arriba de
`class Ship`, con una fila por power-up (`time`/`color`/`glyph`/`label`/`flame`).
Es el único sitio donde vive la metadata compartida: `Pickup.draw`, `Ship.draw`,
la colisión en `update()` y el HUD leen la misma fila. `BOOST_MULT` (game.js:134)
no está en la tabla porque es mecánica específica de Velocidad y solo lo lee
`Ship.update`. Agregar un power-up = una fila en la tabla más su mecánica.

`RADII`/`SPEEDS`/`POINTS` (game.js:73-75) se indexan por tamaño 1..3 y el
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
(game.js:426-439): no hagas `splice` ni borres mientras iterás.

`Pickup` es la excepción a "todo lleva `dead`": vive en `pickups`, un objeto
con un slot por tipo (`pickups.speed`, `pickups.triple`), no en un array. Se
agota con `pickups[kind] = null` (game.js:453-459) y `null` significa "recogido,
todavía no disponible". `spawnPickup(kind)` (game.js:338) lo recria cuando expira
ese boost. Un slot en `null` no bloquea al otro: los dos power-ups pueden estar
en pantalla a la vez, y `initGame()` (game.js:359-360) los rehace juntos.

## Power-ups

- Dos contadores en la nave, `ship.speedTimer` y `ship.tripleTimer`, y **ninguno
  se declara en `Ship.reset()`** (game.js:150-161): `reset()` corre al morir
  (game.js:403) y en `nextLevel()` (game.js:367), así que los boosts sobreviven
  a ambos. Si los movés ahí a secas, el efecto se pierde al reaparecer.
- Los contadores solo bajan en `Ship.update` (game.js:168-169), que solo corre
  en la rama `playing` (game.js:412): durante los 2 s de `dead` se congelan,
  igual que `invincible`.
- El efecto solo lee su propio contador: `mult` en `Ship.update` (speed),
  `tryShoot`/`activePowerup` en `Ship` (triple). No cruces mecánicas.
- El pickup de cada tipo reaparece por su propia línea (game.js:419-420), atada
  a su propio timer. Esas líneas van **antes** de la colisión del mismo frame
  (game.js:453-459); no las muevas después sin revisar que no re-spawneen con
  el boost activo.
- El efecto cruza muerte y cambio de nivel porque `initGame()` lo crea
  (game.js:359-360) y `nextLevel()` no toca ni los timers ni los slots.

## Wrap, dt y bootstrap

- `wrap()` (game.js:27) hace el espacio toroidal y lo usan `Bullet`, `Asteroid`,
  `Ship` y `Pickup`. `Particle` **no** lo usa a propósito (explosiones cortas
  que salen del canvas). No lo "arregles".
- `loop` clampa `dt` a 0.05 (game.js:545) para que un cambio de pestaña no
  provoque tunneling. Mantenelo.
- Al final del archivo corren `initGame()` y `requestAnimationFrame(loop)`
  (game.js:552-553): importar `game.js` en un test arranca el juego y el loop.

## Convenciones

- Identificadores y código en inglés; comentarios y README en español.
- Los textos de UI mezclan inglés (`SCORE`, `GAME OVER`) y español (`NIVEL`,
  `PUNTAJE`, `ESPACIO PARA REINICIAR`). No los normalices sin preguntar.

## Verificación

- Único chequeo automático posible: `node --check game.js` (solo sintaxis).
- Después de eso, revisión visual en el navegador. No declares "corregido"
  sin haberlo ejecutado.
