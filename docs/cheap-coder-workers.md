# Workers baratos: cheap-coder

Guía de transporte del servidor MCP `cheap-coder` (repo
`~/VSCODE/REPOS/cheap-coder-mcp`): qwen3-coder-next vía OpenRouter con pin a
Parasail, registrado en Claude Code (scope user) y en Codex. La rúbrica de
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

| Encargo | Tiempo | Costo | Tool calls | Salida |
|---------|--------|-------|------------|--------|
| Fix mecánico, 1 archivo | 28 s | ~$0.02 | 9 | correcto a la primera |
| Fix acotado, varios archivos chicos | 197 s | $0.13 | 36 | correcto; sin commit y con un summary no pedido |
| Fix multi-archivo con matices | agotó los 600 s de pared | $0.77 | 115 | ~90%, con 4 defectos rescatados por el orquestador |

Con caching una tarea del primer tipo baja a ~$0.004–0.01. La degradación es
por **tiempo**: el costo sigue siendo trivial en absoluto, pero la ambigüedad
y la coherencia entre archivos alargan la corrida hasta el timeout. Ese
presupuesto de pared es la palanca de control — dale uno corto para que
escale temprano y retomes tú, en vez de pagar 600 s por un 90%. Como fuente
barata de telemetría para benchmarks rinde bien; como worker de confianza,
todavía no.

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
premium del trato. Los defectos que sobreviven son de forma, no de lógica:
rutas mal prefijadas, nombres fuera de la convención del repo, aserciones
faltantes en tests que declara escritos y docs actualizadas a medias; revisa
eso primero. Puede además dejar el cambio sin commitear y sumar un summary
que nadie pidió, así que el diff es la verdad, no la branch ni el reporte.
Integra solo lo validado (merge/cherry-pick de la branch o aplicando el diff)
y al cerrar elimina worktree y branch
(`git worktree remove <path> && git branch -D cheap-coder/task-<id>`).

## Gotchas

- El bash del worker ve la API key de OpenRouter (pi no filtra el env):
  delega solo sobre repos confiables y tareas sin secretos en juego.
- En Codex, `tool_timeout_sec` debe superar el presupuesto de pared de la
  tarea (instalado: 900 s); el default de 60 s cortaría cualquier tarea real.
- `telemetry.provider` reporta `openrouter`, no el upstream real (Parasail);
  verifica el caching por `cacheReadTokens`/`costUsd`, no por ese campo.
