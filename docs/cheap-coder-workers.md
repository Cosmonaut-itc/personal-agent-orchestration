# Workers baratos: cheap-coder

Guía de transporte del servidor MCP `cheap-coder` (repo
`~/VSCODE/REPOS/cheap-coder-mcp`): `stealth/ox-alpha` vía OpenRouter con pin
al provider Stealth, registrado en Claude Code (scope user) y en Codex. La rúbrica de
`agent-routing.md` decide *cuándo* delegar aquí; este archivo define *cómo*.
Cada delegación sigue el contrato Despachar → Recoger → Vetar → Cerrar de
`orchestration.md`.

## Tools y roles

| Tool | Rol | Permisos | Presupuesto default |
|------|-----|----------|---------------------|
| `cheap_coder_explore` | scout | solo lectura (read/grep/find/ls) | 60 tool calls · 5 min |
| `cheap_coder_implement` | worker | escribe en worktree aislado | 120 tool calls · 10 min |
| `cheap_coder_test` | tester | ejecuta tests en worktree aislado | 80 tool calls · 10 min |

## Perfil observado

**Sin perfil todavía.** El worker cambió a `stealth/ox-alpha` el 2026-08-21 y
la telemetría acumulada hasta esa fecha es de qwen3-coder-next: no se
transfiere. Trátalo como capacidad no medida hasta que haya corridas propias
en `telemetry/usage.jsonl` — despacha primero encargos con criterio de éxito
verificable y veta el diff completo, que es lo que produce esa medición.

Lo que sí cambia por diseño: el modelo es **gratis** (0 por token en
OpenRouter), así que el único presupuesto que gasta una tarea es el **tiempo
de pared**. Ese es la palanca de control — dale uno corto para que escale
temprano y retomes tú, en vez de esperar 600 s por un parcial.

## Despachar

Input común a las tres tools:

- `task` (requerido): objetivo autocontenido, con criterios de aceptación.
- `repoPath` (requerido): ruta **absoluta** dentro del repo objetivo; el
  server resuelve la raíz git desde ahí.
- `paths` (opcional): rutas relativas de interés dentro del repo.
- `constraints` (opcional): restricciones adicionales del encargo.
- `budget` (opcional): `{ maxToolCalls?, maxWallTimeMs? }`; un override
  parcial conserva el default del rol en el subcampo omitido.

## Recoger

Las tres tools devuelven el mismo Envelope (`structuredContent`):

- `taskId` — correlaciona con la telemetría (`telemetry/usage.jsonl` en el
  repo del server).
- `status: "completed" | "escalated"`. **`escalated` no es un error**: la
  tarea superó presupuesto (`wall_timeout`, `tool_call_budget`) o capacidad
  (`worker_declared`); el parcial y el worktree sobreviven — retómala en un
  modelo premium con ese contexto. `isError` queda reservado a fallos de
  infraestructura o del worker.
- `report` — varía por rol. Confía solo en los campos **derivados** por el
  harness o git (`filesRead`, `changedFiles`, `diffSummary`); el resto lo
  declara el modelo y se contrasta en el veto.
- `worktree` — `null` en explore; en implement/test,
  `{ path, branch, diffSummary }`: worktree en
  `../.cheap-coder/worktrees/task-<id>`, branch `cheap-coder/task-<id>`.

## Vetar y cerrar

Lee el diff completo del worktree antes de integrar — siempre; es la mitad
premium del trato. Con el modelo anterior los defectos que sobrevivían eran
de forma, no de lógica: rutas mal prefijadas, nombres fuera de la convención
del repo, aserciones faltantes en tests que declaraba escritos y docs
actualizadas a medias. Sigue revisando eso primero, pero sin darlo por el
patrón del modelo actual, que aún no tiene perfil. Un worker puede además
dejar el cambio sin commitear y sumar un summary que nadie pidió, así que el
diff es la verdad, no la branch ni el reporte.
Integra solo lo validado (merge/cherry-pick de la branch o aplicando el diff)
y al cerrar elimina worktree y branch
(`git worktree remove <path> && git branch -D cheap-coder/task-<id>`).

## Gotchas

- El bash del worker ve la API key de OpenRouter (pi no filtra el env):
  delega solo sobre repos confiables y tareas sin secretos en juego.
- En Codex, `tool_timeout_sec` debe superar el presupuesto de pared de la
  tarea (instalado: 900 s); el default de 60 s cortaría cualquier tarea real.
- `telemetry.provider` reporta `openrouter`, no el upstream real (Stealth);
  verifica el caching por `cacheReadTokens`, no por ese campo. `costUsd` es 0
  mientras el modelo sea gratis, así que ya no sirve como señal de nada.
