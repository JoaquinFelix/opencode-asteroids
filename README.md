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
