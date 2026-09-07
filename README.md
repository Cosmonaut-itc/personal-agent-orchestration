# Personal Agent Orchestration

Configuración documental para que Codex y Claude Code compartan contrato de
orquestación, selección de trabajadores y cierre de encargos, aunque cada
harness use un transporte distinto.

## Estructura

| Ruta | Propósito |
|------|-----------|
| [`docs/orchestration.md`](docs/orchestration.md) | Fuente canónica del contrato común: roles, descendientes y cierre. |
| [`docs/agent-routing.md`](docs/agent-routing.md) | Tabla de modelos, esfuerzo, escalado, review y UI/UX. |
| [`docs/native-subagents.md`](docs/native-subagents.md) | Contexto, configuración y seguimiento de subagentes nativos en Codex y Claude Code. |
| [`configs/codex/`](configs/codex/AGENTS.md) | Entry point global de Codex, con el contrato inlineado, y guía Codex → Claude. |
| [`configs/claude/`](configs/claude/CLAUDE.md) | Entry point global de Claude Code, con el contrato inlineado, y guía Claude Code → GPT. |

## Contrato común

El [contrato](docs/orchestration.md) distingue al **orquestador**, responsable
del resultado final, del **trabajador**, que resuelve el encargo, y del
**adaptador**, que transporta una ruta prescrita. Cada delegación pasa por
**Despachar → Recoger → Vetar → Cerrar**.

Los entrypoints globales llevan ese contrato inlineado, de modo que rige desde
el primer turno sin una lectura previa. `docs/orchestration.md` es la fuente
canónica: al editarlo, replica el cambio en ambos entrypoints.

El [routing](docs/agent-routing.md) es la única fuente para elegir modelo,
esfuerzo y pases de review. Los entrypoints apuntan a esa política; las guías
describen cómo ejecutarla en cada entorno.

## Rutas de ejecución

| Ruta | Mecanismo | Guía |
|------|-----------|------|
| Codex → GPT | Subagentes nativos de Codex. | [`native-subagents.md`](docs/native-subagents.md), rama Codex |
| Claude Code → Claude | Subagentes nativos de Claude Code. | [`native-subagents.md`](docs/native-subagents.md), rama Claude Code |
| Codex → Claude | Proceso externo `claude -p`, seguido hasta su terminación. | [`claude-workers.md`](configs/codex/docs/claude-workers.md) |
| Claude Code → GPT | Runtime del plugin openai/codex-plugin-cc, adaptador opcional o CLI de Codex. | [`codex-delegation.md`](configs/claude/docs/codex-delegation.md) |

Un trabajador externo por CLI tiene un ciclo de vida distinto del subagente
nativo. Cada guía explica qué contexto y controles ofrece.

El MCP `cheap-coder` (`stealth/ox-alpha`) está **deprecado por ahora**: el
routing prohíbe despachar por sus tools. Para retirarlo del equipo:

```bash
claude mcp remove cheap-coder -s user
# y elimina el bloque [mcp_servers.cheap-coder] de ~/.codex/config.toml
```

## Alcance

Este repositorio define el contrato de orquestación y el routing comunes a
los proyectos. Las secuencias de implementación, revisión y tracker pertenecen
a cada repositorio y se declaran en su propio `AGENTS.md`, `CLAUDE.md` o docs
locales.

## Instalación manual

Revisa el diff y respalda cualquier archivo de destino existente. Después,
desde la raíz de este repositorio, copia de forma interactiva:

```bash
mkdir -p ~/.agents/docs ~/.codex/docs ~/.claude/docs

cp -i docs/orchestration.md ~/.agents/docs/orchestration.md
cp -i docs/agent-routing.md ~/.agents/docs/agent-routing.md
cp -i docs/native-subagents.md ~/.agents/docs/native-subagents.md
rm -f ~/.agents/docs/cheap-coder-workers.md

cp -i configs/codex/AGENTS.md ~/.codex/AGENTS.md
cp -i configs/codex/docs/claude-workers.md ~/.codex/docs/claude-workers.md

cp -i configs/claude/CLAUDE.md ~/.claude/CLAUDE.md
cp -i configs/claude/docs/codex-delegation.md ~/.claude/docs/codex-delegation.md
```

Instala también las guías referenciadas: cambiar sólo los entrypoints dejaría
apuntadores incompletos. Acepta cada reemplazo cuando el diff corresponda a la
configuración que quieres activar. Editar este repositorio no cambia los
archivos ya instalados en tu equipo.
