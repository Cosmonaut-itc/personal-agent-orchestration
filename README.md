# Personal Agent Orchestration

Configuración documental para que Codex y Claude Code compartan el mismo
contrato de orquestación, selección de agentes y cierre de trabajo delegado,
aunque cada harness use un transporte distinto.

## Estructura

| Ruta | Propósito |
|------|-----------|
| [`docs/orchestration.md`](docs/orchestration.md) | Contrato común, roles y regla antibucle. |
| [`docs/agent-routing.md`](docs/agent-routing.md) | Fuente canónica para elegir agente, modelo, esfuerzo y pases de review. |
| [`docs/cheap-coder-workers.md`](docs/cheap-coder-workers.md) | Guía de transporte del MCP `cheap-coder` (workers baratos qwen3-coder-next). |
| [`configs/codex/`](configs/codex/AGENTS.md) | Entry point global de Codex y puente Codex → Claude/Opus. |
| [`configs/claude/`](configs/claude/CLAUDE.md) | Entry point global de Claude Code y puente Claude → Codex. |

## Contrato común

El [contrato](docs/orchestration.md) distingue al **orquestador**, que conserva
ownership y veta el resultado final, del **trabajador**, que resuelve un
encargo autocontenido y devuelve evidencia. Cada delegación pasa por cuatro
fases: **Despachar → Recoger → Vetar → Cerrar**. Un trabajador no vuelve a
delegar al harness que lo lanzó.

La [rúbrica de routing](docs/agent-routing.md) es la única fuente para decidir
modelo, esfuerzo, UI/UX y review. Los entrypoints y puentes sólo describen cómo
transportar la tarea; no redefinen esa selección.

## Puentes

- **Codex → Claude/Opus:**
  [`claude-workers.md`](configs/codex/docs/claude-workers.md) documenta el
  trabajador externo no interactivo con `claude -p`, permisos por tarea,
  salida estructurada y validación posterior por Codex.
- **Claude → Codex:**
  [`codex-delegation.md`](configs/claude/docs/codex-delegation.md) documenta el
  puente `openai/codex-plugin-cc`, el subagente `codex:codex-rescue`, sus
  comandos de seguimiento y el fallback directo al Codex CLI.

## Alcance

Este repositorio define únicamente el contrato de orquestación y la rúbrica de
routing, que son comunes a cualquier proyecto. Las secuencias de
implementación, revisión y tracker pertenecen a cada repositorio y se declaran
en su propio `AGENTS.md`, `CLAUDE.md` o docs locales; mantenerlas aquí
duplicaría decisiones que sólo el proyecto puede tomar.

## Instalación manual

Revisa el diff y respalda cualquier archivo de destino existente. Después,
desde la raíz de este repositorio, copia de forma interactiva para evitar
sobrescrituras accidentales:

```bash
mkdir -p ~/.agents/docs ~/.codex/docs ~/.claude/docs

cp -i docs/orchestration.md ~/.agents/docs/orchestration.md
cp -i docs/agent-routing.md ~/.agents/docs/agent-routing.md
cp -i docs/cheap-coder-workers.md ~/.agents/docs/cheap-coder-workers.md

cp -i configs/codex/AGENTS.md ~/.codex/AGENTS.md
cp -i configs/codex/docs/claude-workers.md ~/.codex/docs/claude-workers.md

cp -i configs/claude/CLAUDE.md ~/.claude/CLAUDE.md
cp -i configs/claude/docs/codex-delegation.md ~/.claude/docs/codex-delegation.md
```

Lee primero los archivos de destino que ya existan y acepta cada reemplazo
sólo cuando el diff corresponda a la configuración que quieres activar.
