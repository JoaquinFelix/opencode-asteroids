# Asteroids

Clon del clásico arcade **Asteroids** implementado en canvas HTML5 puro, sin dependencias ni bundler.

## Descripción

Nave espacial en un campo de asteroides con envolvimiento de bordes (el espacio es toroidal). Destruye asteroides para sumar puntos: los grandes se parten en medianos, los medianos en pequeños.

## Tecnologías

- **HTML5 Canvas** — renderizado 2D
- **JavaScript (ES6+)** — lógica del juego en un solo archivo `game.js`
- Sin frameworks, sin bundler, sin dependencias

## Cómo correr

Abre `index.html` directamente en el navegador (doble clic), o usa un servidor local:

```bash
npx serve .
```

Luego visita `http://localhost:3000`.

## Controles

| Tecla     | Acción     |
| --------- | ---------- |
| `←` `→`   | Rotar nave |
| `↑`       | Propulsar  |
| `Espacio` | Disparar   |
| `S`       | Skin siguiente (`Shift+S`: anterior) |

## Skins

Cinco skins de nave, todas disponibles desde el inicio. Se cambian con `S`
(siguiente) o `Shift+S` (anterior), en cualquier momento, incluso en el game
over. El nombre de la skin activa aparece en el HUD, centrado debajo de
`NIVEL`, con el color del casco.

| Skin | Casco | Silueta |
| ---- | ----- | ------- |
| Clásica | Blanco | Triángulo con muesca trasera (la original) |
| Interceptor | Verde menta | Delta ancha con punta alargada |
| Halcón | Amarillo | Alas en gancho con doble muesca |
| Sombra | Violeta | Doble diente con punta trasera puntiaguda |
| Brasa | Naranja | Cuña ancha con muesca profunda |

Cada skin cambia a la vez el color del casco, las balas, las partículas de
explosión, la llama del propulsor y los íconos de vidas del HUD. La elección no
se guarda: al recargar la página vuelve a la Clásica. Con el power-up `V`
activo, el casco y la llama se tiñen de cian (las balas y las partículas
conservan el color de la skin).

## Power-ups

| Power-up      | Efecto                                                        |
| ------------- | ------------------------------------------------------------- |
| `V` Velocidad | Duplica el empuje y la rotación de la nave durante **5 minutos** |

Aparece como un ícono `V` cian que deriva por el campo. Al recogerlo arranca la
cuenta regresiva, visible en el HUD (abajo a la izquierda, en cian). La nave y
la llama se ponen cian mientras dura el efecto.

Cuando expira, el ícono vuelve a aparecer en un punto seguro (fuera del área de
reaparición). El contador sigue corriendo si la nave muere o se pasa de nivel:
el efecto no se pierde ni se reinicia.

## Puntuación

| Asteroide | Puntos |
| --------- | ------ |
| Grande    | 20     |
| Mediano   | 50     |
| Pequeño   | 100    |

## Características

- 3 vidas con invencibilidad temporal al reaparecer (parpadeo)
- Asteroides se parten en fragmentos más pequeños al ser destruidos
- Partículas de explosión al destruir asteroides
- Power-up `V` que duplica el empuje y la rotación durante 5 minutos
- 5 skins de nave seleccionables con `S`, que cambian silueta, colores, balas y partículas
