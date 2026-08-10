# Trabajadores Claude desde Codex

Esta guía cubre la mecánica Codex → Claude. La política y el criterio de cierre
viven en `~/.agents/docs/orchestration.md`.

Antes de elegir el trabajador, lee `~/.agents/docs/agent-routing.md`; después
usa este archivo sólo para transportarlo hacia Claude.

`claude -p` ejecuta un proceso Claude Code no interactivo. Es un trabajador
externo: no hereda las herramientas integradas de subagentes de Codex ni
aparece como uno de sus threads.

## Disponibilidad

Antes de la primera invocación de una tarea, comprueba sin iniciar trabajo:

```bash
command -v claude
claude --version
claude --help
```

El patrón actual requiere `-p`, `--safe-mode`, `--permission-mode`, `--tools`,
`--output-format` y `--no-session-persistence`. `--safe-mode` conserva auth,
modelo, herramientas integradas y permisos, pero desactiva CLAUDE.md, skills,
plugins, hooks, MCP, comandos y agentes personalizados. Así el trabajador no
recarga la política del orquestador ni forma un ciclo.

## Patrón por defecto

Para investigación read-only del repositorio:

```bash
claude --safe-mode -p \
  --model opus \
  --effort high \
  --permission-mode dontAsk \
  --tools "Read,Grep,Glob" \
  --output-format json \
  --no-session-persistence \
  "TASK"
```

Los nombres de routing `sonnet-5` y `opus-5` se invocan aquí mediante los
aliases Claude Code `sonnet` y `opus`: la tabla decide el agente y esta guía
decide el flag operativo.

`TASK` debe ser autocontenida y decir que el proceso actúa como trabajador
final: resuelve directamente, devuelve evidencia y no delega. `--tools` omite
`Agent`, por lo que el trabajador no puede abrir otra delegación.

Cuando todo el material ya llega por stdin, usa `--tools ""` en el mismo
patrón. Codex entrega sólo datos confiables por stdin y ejecuta `-p` únicamente
desde un directorio confiable: el modo no interactivo omite el diálogo de
confianza del workspace.

## Escritura

Un trabajador con escritura requiere autorización explícita, un worktree
aislado y una superficie definida por la tarea. Selecciona sólo las
herramientas necesarias; usa `acceptEdits` para edición de archivos y reglas
`--allowedTools` precisas para comandos de shell. Conserva read-only como
default cuando la tarea es review, investigación o diseño.

La alternativa portable para automatización es `--bare`, recomendada por
Anthropic para scripts. `--bare` no usa OAuth ni el keychain: requiere
`ANTHROPIC_API_KEY`, `apiKeyHelper` o credenciales del proveedor configurado.

## Validación y cierre

Codex:

1. exige código de salida cero y JSON parseable;
2. valida `result` contra el objetivo y los criterios de aceptación;
3. inspecciona el diff y ejecuta los gates aplicables si hubo cambios;
4. reorienta o repite un resultado insuficiente y termina el proceso al cerrar.

Una respuesta textual correcta no prueba que una edición o un test ocurrió.
El controlador verifica esa evidencia en el sistema correspondiente.

## Límites reales

- No hay steering, mailbox ni lifecycle integrado entre Codex y el proceso
  Claude; el controlador debe manejar stdin/stdout, timeout y cancelación.
- `--agents` define subagentes internos de Claude y `--agent` cambia el agente
  principal. No son necesarios para un único trabajador Opus y amplían el
  árbol fuera de la observabilidad de Codex.
- Los permisos de Codex no se transfieren al proceso. La invocación Claude
  define los suyos de forma independiente para cada tarea.

Referencias primarias:

- [Claude Code no interactivo](https://code.claude.com/docs/en/headless)
- [Permisos de Claude Code](https://code.claude.com/docs/en/permissions)
- [Subagentes de Claude Code](https://code.claude.com/docs/en/sub-agents)
