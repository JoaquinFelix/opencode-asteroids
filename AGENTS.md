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
  `level`, `state`, `skinIndex`); funciones `initGame`, `nextLevel`,
  `spawnPickup`, `updatePickups`, `explode`, `killShip`, `update(dt)`, `draw()`
  y `loop(ts)`.
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
`SPEED = 520` en el constructor de `Bullet` (game.js:68);
`ROT`/`THRUST`/`DRAG` dentro de `Ship.update` (game.js:221-223);
`NOSE = currentSkin().nose` y `TRIPLE_SPREAD = 0.15` en `tryShoot`
(game.js:244-245);
`SAFE_DIST = 130` con `randomSafePoint()` en Utils (game.js:33-42), compartido
por `spawnAsteroids` y `spawnPickup`.
Tunear = editar donde está; si centralizás, mantené los nombres.

Dos grupos aislados:

- `BOOST_MULT` (game.js:149) y `POWERUPS` (game.js:152-155), arriba de `class
  Ship`. `POWERUPS` es el único sitio donde vive la metadata de los power-ups
  (`time`/`color`/`glyph`/`label`/`flame`): la leen `Pickup.draw`, `Ship.draw`
  y `activePowerup`, la colisión en `update()` (game.js:505-512) y el HUD
  (game.js:554-555). `BOOST_MULT` no está en la tabla porque es mecánica
  específica de Velocidad y solo lo lee `Ship.update` (game.js:224). Agregar un
  power-up = una fila en la tabla más su mecánica.
- `SKINS` (game.js:161-187), tabla de apariencias de la nave: `name`, `shape`
  (pares `[x,y]` relativos), `nose`, `nozzle` (`[x, ancho]` de la tobera) y los
  colores `hull`/`bullet`/`spark`/`flame`. La leen `Ship.draw`, `tryShoot`,
  `Bullet.draw`, `Particle.draw` y `drawLifeIcon` vía `currentSkin()`
  (game.js:190). `skinIndex` (game.js:189) va junto a la tabla y **no** se
  resetea en `initGame()`: la elección dura toda la sesión. La cambia el bloque
  de `S`/`Shift+S` al tope de `update()` (game.js:437-440). Agregar una skin =
  agregar una entrada; si cambiás siluetas, dejá `nose` en el vértice más a la
  derecha. Los auxiliares `rgba()` y `strokeShape()` viven en Utils
  (game.js:46-61) y comparten color de casco, balas, partículas e íconos.

`RADII`/`SPEEDS`/`POINTS` (game.js:92-94) se indexan por tamaño 1..3 y el
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
(game.js:479-491): no hagas `splice` ni borres mientras iterás.

`Pickup` es la excepción a "todo lleva `dead`": vive en `pickups`, un objeto
con un slot por tipo (`pickups.speed`, `pickups.triple`), no en un array. Se
agota con `pickups[kind] = null` (game.js:504-512) y `null` significa "recogido,
todavía no disponible". `spawnPickup(kind)` (game.js:385) lo recria cuando expira
ese boost. Un slot en `null` no bloquea al otro: los dos power-ups pueden estar
en pantalla a la vez, y `initGame()` (game.js:406-407) los rehace juntos.

## Power-ups

- Dos contadores en la nave, `ship.speedTimer` y `ship.tripleTimer`, y **ninguno
  se declara en `Ship.reset()`** (game.js:200-211): `reset()` corre al morir
  (game.js:456) y en `nextLevel()` (game.js:414), así que los boosts sobreviven
  a ambos. Si los movés ahí a secas, el efecto se pierde al reaparecer.
- Los contadores solo bajan en `Ship.update` (game.js:218-219), que solo corre
  en la rama `playing` (game.js:460): durante los 2 s de `dead` se congelan,
  igual que `invincible`.
- El efecto solo lee su propio contador: `mult` en `Ship.update` (game.js:224,
  speed), `tryShoot`/`activePowerup` en `Ship` (game.js:241-261, triple). No
  cruces mecánicas.
- El pickup de cada tipo reaparece por su propia línea (game.js:472-473), atada
  a su propio timer. Esas líneas van **antes** de la colisión del mismo frame
  (game.js:504-512); no las muevas después sin revisar que no re-spawneen con
  el boost activo.
- El efecto cruza muerte y cambio de nivel porque `initGame()` lo crea
  (game.js:406-407) y `nextLevel()` no toca ni los timers ni los slots.
- Con un power-up activo, `Ship.draw` tiñe casco con `powerup.color` y llama con
  `powerup.flame` (game.js:273, 288); balas y partículas conservan el color de
  la skin.

## Wrap, dt y bootstrap

- `wrap()` (game.js:27) hace el espacio toroidal y lo usan `Bullet`, `Asteroid`,
  `Ship`, `Pickup` y también el índice de `skinIndex` al rotar de skin (no es
  un wrap espacial). `Particle` **no** lo usa a propósito (explosiones cortas
  que salen del canvas). No lo "arregles".
- `loop` clampa `dt` a 0.05 (game.js:600) para que un cambio de pestaña no
  provoque tunneling. Mantenelo.
- Al final del archivo corren `initGame()` y `requestAnimationFrame(loop)`
  (game.js:607-608): importar `game.js` en un test arranca el juego y el loop.

## Convenciones

- Identificadores y código en inglés; comentarios y README en español.
- Los textos de UI mezclan inglés (`SCORE`, `GAME OVER`, `SKIN`) y español
  (`NIVEL`, `PUNTAJE`, `ESPACIO PARA REINICIAR`, nombres de skins). No los
  normalices sin preguntar.

## Verificación

- Único chequeo automático posible: `node --check game.js` (solo sintaxis).
- Después de eso, revisión visual en el navegador. No declares "corregido"
  sin haberlo ejecutado.
