# Delegación y elección de modelo

Fuente canónica para que Codex y Claude Code elijan agente, modelo y secuencia
de revisión con la misma rúbrica.

| modelo        | costo | inteligencia | gusto |
|---------------|-------|--------------|-------|
| gpt-5.6-sol   | 8     | 9            | 6     |
| gpt-5.6-terra | 9     | 8            | 5     |
| gpt-5.6-luna  | 10    | 7            | 4     |
| sonnet-5      | 5     | 5            | 7     |
| opus-5        | 4     | 8            | 8     |
| qwen3-coder-next | 10 | 3            | 2     |

Elige siempre dentro de esta tabla. El **presupuesto** real son los tokens
Claude (gpt-5.6 es gratis en la práctica): por defecto gpt-5.6 cuando
plausiblemente da la talla; gasta Claude donde el gusto o el juicio difícil
pagan. En conflicto sobre algo que se embarca: inteligencia > gusto > costo.
Tienes permiso permanente de escalar a un modelo mejor cuando el output no da
la talla, sin preguntar.

UI/UX: primero un prototipo de fidelidad baja/media aprobado por el dueño
(diseña opus-5 si no existe prototipo ni guía); implementa gpt-5.6.

Reviews: primero gpt-5.6, siempre en la variante `gpt-5.6-sol` con esfuerzo
`high`; después el orquestador lee el diff completo y veta tanto el cambio como
sus hallazgos. Un cambio de **alto riesgo** lleva además un pase opus-5
independiente: auth/permisos, migraciones de datos, releases y lo que el
archivo de instrucciones del proyecto marque como alto riesgo.

La tabla decide el agente; la guía de transporte del harness decide el alias,
flag o herramienta vigente para invocarlo.

## Worker barato: qwen3-coder-next (cheap-coder)

El servidor MCP `cheap-coder` expone qwen3-coder-next como trabajador dentro
de esta rúbrica, bajo el mismo contrato Despachar → Recoger → Vetar → Cerrar.
Su valor está en delegar **antes** de gastar tokens premium leyendo el repo.
Rutéale tareas bien especificadas, verificables mecánicamente y de bajo
riesgo: exploración de repos (`cheap_coder_explore`), cambios acotados que
siguen un patrón existente (`cheap_coder_implement`) y tests unitarios o
reproducción de bugs localizados (`cheap_coder_test`). Las decisiones de
arquitectura, el código sensible (auth, schema, migraciones), los bugs
ambiguos y el review final se quedan en los modelos premium de la tabla.

Todo lo que un worker barato produce se veta leyendo su diff completo antes
de integrar. Antes de despachar una tarea al MCP, lee
`~/.agents/docs/cheap-coder-workers.md` — contrato de input, Envelope,
presupuestos y cierre del worktree.
