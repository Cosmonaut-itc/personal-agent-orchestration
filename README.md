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
| [`docs/opencode-workers.md`](docs/opencode-workers.md) | Despacho programático de cheap coders Go: CLI/HTTP, modelo, controles y cierre. |
| [`configs/codex/`](configs/codex/AGENTS.md) | Entry point global de Codex, con el contrato inlineado, y guía Codex → Claude. |
| [`configs/claude/`](configs/claude/CLAUDE.md) | Entry point global de Claude Code, con el contrato inlineado, y guía Claude Code → GPT. |
| [`configs/opencode/`](configs/opencode/opencode.json) | Perfiles DeepSeek V4.1 Flash, MiniMax M3 y Kimi K3, con instrucciones compartidas. |

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
| Codex o Claude Code → OpenCode Go | `opencode run` con perfil/modelo explícitos; servidor HTTP para despacho asíncrono. | [`opencode-workers.md`](docs/opencode-workers.md) |

Un trabajador externo por CLI tiene un ciclo de vida distinto del subagente
nativo. Cada guía explica qué contexto y controles ofrece.

El MCP antiguo `cheap-coder` (`stealth/ox-alpha`) sigue **deprecado**. Los
perfiles Go usan OpenCode y no reactivan ese MCP. Para retirar el antiguo:

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
cp -i docs/opencode-workers.md ~/.agents/docs/opencode-workers.md
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

### Perfiles OpenCode Go

Los perfiles requieren OpenCode y el proveedor Go conectado. Conserva el
archivo de configuración existente: combina la clave `agent` y las variantes
de `provider.opencode-go.models.kimi-k3` de
[`configs/opencode/opencode.json`](configs/opencode/opencode.json) con la
configuración global o del proyecto, y copia `worker-prompt.md` junto al JSON
para resolver su referencia relativa. Revisa colisiones de nombres antes de
combinar. No reemplaces proveedores, MCP ni otras preferencias existentes.

Los perfiles parten de lectura y búsqueda; los permisos de implementación
se fijan por encargo mediante overrides de ejecución. Kimi K3 está habilitado
con low de inicio y high al escalar. Go recibió y aceptó ambos valores en
pruebas reales; el dueño autorizó su uso aceptando que el nivel interno
aplicado no tiene confirmación observable. Sigue la
[guía de transporte](docs/opencode-workers.md) para consultar la evidencia,
las comprobaciones pendientes y los controles de cada despacho.
