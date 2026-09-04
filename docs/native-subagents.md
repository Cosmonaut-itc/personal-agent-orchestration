# Subagentes nativos

Lee únicamente la rama del entorno que ejecuta el encargo. Decide por las
herramientas expuestas en la sesión, no por el nombre comercial de la app.

## Codex

Los nombres siguientes no están verificados desde el CLI: confirma el esquema
de herramientas expuesto en la sesión antes de usarlos, y comprueba qué
defaults y perfiles de agente alteran lo que pides.

Las herramientas para crear tareas del usuario, bifurcar conversaciones o
moverlas entre equipos gestionan otro ciclo de vida; úsalas cuando el usuario
pida esas operaciones.

En la superficie que exponga `collaboration.spawn_agent`:

- Usa `fork_turns: "none"` y un encargo autocontenido para seleccionar
  `model` y `reasoning_effort` explícitos. También puede existir un fork
  parcial; consulta el esquema antes de usarlo.
- Un fork completo hereda modelo y esfuerzo y no admite esos overrides en
  esta superficie.
- Conserva el identificador devuelto. `send_message` orienta a un agente;
  `followup_task` puede reactivar uno inactivo. `wait_agent` avisa de eventos:
  recoge también el mensaje o resultado asociado.
- `interrupt_agent` interrumpe un turno; comprueba después el estado. Si no
  existe una operación de eliminación, conserva el agente terminado sin
  atribuirle un cierre físico inexistente.

Los trabajadores comparten el directorio por defecto. Define la propiedad de
las escrituras antes de paralelizar.

Referencia: [Subagents](https://learn.chatgpt.com/docs/agent-configuration/subagents).

## Claude Code

Usa `Agent` con una definición que fije `model` y `effort`. Comprueba overrides
de entorno, especialmente `CLAUDE_CODE_EFFORT_LEVEL`, antes de confiar en el
frontmatter. Los perfiles sin esfuerzo pueden heredarlo.

Un subagente ordinario empieza con contexto nuevo; un fork hereda la
conversación y su configuración efectiva. Conserva el ID para continuar y
recoger el resultado. Para escritura aislada, usa `isolation: worktree`
cuando exista.

Subagentes, sesiones de fondo y equipos tienen ciclos distintos. Para una
sesión de fondo, sigue por ID los controles actuales de identificación, logs,
parada y retirada.

Referencias: [Subagents](https://code.claude.com/docs/en/sub-agents),
[esfuerzo](https://code.claude.com/docs/en/model-config#effort-level),
[sesiones de fondo](https://code.claude.com/docs/en/agent-view).
