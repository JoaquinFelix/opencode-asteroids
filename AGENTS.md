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
  (`ship`, `bullets`, `asteroids`, `particles`, `pickup`, `score`, `lives`,
  `level`, `state`); funciones `initGame`, `nextLevel`, `spawnPickup`,
  `explode`, `killShip`, `update(dt)`, `draw()` y `loop(ts)`.
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
`ROT`/`THRUST`/`DRAG` dentro de `Ship.update` (game.js:165-167);
`NOSE = 21` en `tryShoot` (game.js:188);
`SAFE_DIST = 130` con `randomSafePoint()` en Utils (game.js:33-42), compartido
por `spawnAsteroids` y `spawnPickup`.
Tunear = editar donde está; si centralizás, mantené los nombres.

Único grupo de constantes aislado: `BOOST_TIME`/`BOOST_MULT`/`BOOST_COLOR`
(game.js:134-136), arriba de `class Ship`. Van juntas porque las tres
constantes las leen tres sitios distintos (`Ship.update`, `Ship.draw`, la
colisión en `update()`); no es un config general.

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
(game.js:392-406): no hagas `splice` ni borres mientras iterás.

`Pickup` es la excepción a "todo lleva `dead`": es un solo objeto, no un array.
Se agota con `pickup = null` (game.js:419-423) y `pickup === null` significa
"recogido, todavía no disponible". `spawnPickup()` lo recria cuando expira el
boost.

## Boost "Velocidad"

- El contador es `ship.speedTimer` y **no se declara en `Ship.reset()`**
  (game.js:140-143): `reset()` corre al morir (game.js:371) y en `nextLevel()`
  (game.js:335), así que el boost sobrevive a ambos. Si lo movés ahí a secas,
  el efecto se pierde al reaparecer.
- El contador solo baja en `Ship.update`, que solo corre en la rama `playing`
  (game.js:380): durante los 2 s de `dead` se congela, igual que `invincible`.
- El pickup reaparece por `if (!pickup && ship.speedTimer <= 0) spawnPickup()`
  (game.js:387). Ese chequeo va **antes** de la colisión del mismo frame; no lo
  muevas después sin revisar que no re-spawnee con el boost activo.
- No es un power-up que pueda estar en pantalla dos veces: `initGame()` lo crea
  (game.js:328) y `nextLevel()` no lo toca, así que cruza cambios de nivel.

## Wrap, dt y bootstrap

- `wrap()` (game.js:27) hace el espacio toroidal y lo usan `Bullet`, `Asteroid`,
  `Ship` y `Pickup`. `Particle` **no** lo usa a propósito (explosiones cortas
  que salen del canvas). No lo "arregles".
- `loop` clampa `dt` a 0.05 (game.js:502) para que un cambio de pestaña no
  provoque tunneling. Mantenelo.
- Al final del archivo corren `initGame()` y `requestAnimationFrame(loop)`
  (game.js:509-510): importar `game.js` en un test arranca el juego y el loop.

## Convenciones

- Identificadores y código en inglés; comentarios y README en español.
- Los textos de UI mezclan inglés (`SCORE`, `GAME OVER`) y español (`NIVEL`,
  `PUNTAJE`, `ESPACIO PARA REINICIAR`). No los normalices sin preguntar.

## Verificación

- Único chequeo automático posible: `node --check game.js` (solo sintaxis).
- Después de eso, revisión visual en el navegador. No declares "corregido"
  sin haberlo ejecutado.
