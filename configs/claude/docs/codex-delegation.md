# Trabajadores Codex desde Claude Code

Esta guía cubre la mecánica Claude → Codex. La política, los roles y el cierre
viven en `~/.agents/docs/orchestration.md`.

Antes de elegir el trabajador, lee `~/.agents/docs/agent-routing.md`; después
usa este archivo sólo para transportarlo hacia Codex.

El puente vigente es el plugin habilitado
[openai/codex-plugin-cc](https://github.com/openai/codex-plugin-cc), que usa el
Codex CLI y su app server locales. Antes del primer despacho de una sesión,
ejecuta `/codex:setup` para comprobar instalación y autenticación.

## Transporte principal

Para un handoff que Claude debe revisar dentro de su flujo de orquestación,
invoca el subagente nativo `codex:codex-rescue` mediante la herramienta Agent.
Es un forwarder delgado: transporta la tarea al runtime Codex y devuelve su
stdout. Claude conserva el resultado, lo veta conforme al contrato y sólo
después integra o cierra.

El shortcut `/codex:rescue` usa el mismo subagente, pero su interfaz devuelve
la salida de Codex verbatim. Úsalo cuando ese handoff textual sea el resultado
esperado; el veto del orquestador ocurre antes de integrar o afirmar cierre.

Comandos instalados:

| Necesidad | Comando |
|-----------|---------|
| Tarea o diagnóstico | `/codex:rescue [--wait\|--background] [--fresh\|--resume] [--model <model>] [--effort <effort>] <task>` |
| Review del estado git | `/codex:review [--wait\|--background] [--base <ref>] [--scope auto\|working-tree\|branch]` |
| Review de enfoque/diseño | `/codex:adversarial-review [mismos flags] [focus ...]` |
| Seguimiento | `/codex:status [job-id] [--wait] [--timeout-ms <ms>] [--all]` |
| Resultado o cancelación | `/codex:result [job-id]` · `/codex:cancel [job-id]` |
| Continuidad en Codex | `/codex:transfer [--source <claude-jsonl>]` |

Los comandos de review y resultado también presentan la salida Codex sin
vetarla. Claude lee esa salida, contrasta cada hallazgo con el diff y cierra
según `~/.agents/docs/orchestration.md`.

## Tarea autocontenida

Codex no ve la conversación Claude. Construye el prompt con todos los campos
de **Despachar** definidos en `~/.agents/docs/orchestration.md`. Declara que
Codex actúa como trabajador: resuelve y reporta directamente, sin invocar
Claude ni abrir un puente de vuelta.

Elige read-only para investigación, análisis y review. Usa escritura sólo
cuando el encargo la autorice; el plugin rescue es write-capable por default,
así que expresa read-only de forma explícita cuando corresponda. Para escritura,
prefiere un worktree aislado; si debe compartir checkout, captura primero
`git status` y el diff existente para proteger trabajo ajeno.

## Fallback directo

Cuando el plugin no cubra el caso o esté indisponible, usa el CLI verificado:

```bash
codex exec \
  -C <repo> \
  -s read-only \
  --ephemeral \
  -m <model> \
  -c 'model_reasoning_effort="high"' \
  "TASK"
```

Cambia a `-s workspace-write` sólo para una tarea de escritura autorizada.
Usa `--json`, `-o <file>` o `--output-schema <file>` cuando el consumidor
necesite salida estructurada. Para seguimiento con contexto, usa
`codex exec resume --last "FOLLOW-UP"`.

Selecciona modelo y esfuerzo según `~/.agents/docs/agent-routing.md`. Pasa esos
flags explícitamente cuando la rúbrica lo requiera; en los demás casos puede
gobernar la configuración Codex vigente.

## Validación y límites

- Claude espera el estado terminal, inspecciona stdout/resultado y verifica el
  diff y los gates aplicables. Un payload correcto no prueba por sí solo que la
  acción ocurrió.
- Un job en background se cierra con `status` + `result`, o se detiene con
  `cancel`; no se abandona como sustituto de cierre.
- El fallback directo no aporta lifecycle integrado: Claude impone un tiempo
  máximo, conserva stdout/stderr y termina el proceso si excede el límite.
- El plugin comparte instalación, auth, checkout y configuración del Codex CLI
  local. Sus jobs no son subagentes Claude de propósito general: el rescue es
  deliberadamente un transporte delgado.

Referencias primarias:

- [Codex CLI](https://developers.openai.com/codex/cli/)
- [Codex app server](https://developers.openai.com/codex/app-server/)
- [Referencia de `codex exec`](https://developers.openai.com/codex/cli/reference/)
