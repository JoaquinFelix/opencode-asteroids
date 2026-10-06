---
description: Crea un worktree en .worktrees/<nombre>
agent: build
---

!`git worktree add ".worktrees/$1"`

Con la salida anterior:

- Analiza el argumento (puede contener espacios) y deriva un nombre corto en kebab-case (minúsculas, sin espacios ni acentos) que represente el contexto.
- Si git falló o no se indicó nombre, reportá el error y explicá el uso: `/worktree <nombre-del-worktree>`. No hagas nada más.
- Si salió bien, confirmá al usuario el worktree creado en `.worktrees/<nombre-del-worktree>` y la rama creada (igual al nombre), verificando con `git worktree list`.
- no hagas commit ni push.
- Si los argumentos son muy largos, simplifícalos a un nombre significativo.
