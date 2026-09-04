# Delegación y elección de modelo

Política común para seleccionar trabajadores y revisores.

## Modelos

Elige siempre dentro de esta tabla. Las puntuaciones son preferencias
operativas (10 = mejor) y se ajustan con resultados observados.

| Modelo        | costo | inteligencia | gusto | Cuándo |
|---------------|-------|--------------|-------|--------|
| gpt-6-astra   | 7     | 10           | 7     | Razonamiento y juicio difícil; primer pase de revisión. |
| gpt-5.6-sol   | 8     | 9            | 6     | Opción habitual para ejecución técnica. |
| gpt-5.6-terra | 9     | 8            | 5     | Alternativa a Sol cuando el proyecto o el transporte lo favorezcan. |
| gpt-5.6-luna  | 10    | 7            | 4     | Encargos claros y fáciles de verificar. |
| opus-5        | 4     | 8            | 8     | Gusto visual, decisiones de producto y contraste independiente de alto riesgo. |
| sonnet-5      | 5     | 5            | 7     | Trabajo en el entorno Claude que no requiera el juicio de Opus. |

El **presupuesto** real son los tokens Claude: GPT es gratis en la práctica.
Por defecto GPT cuando plausiblemente da la talla; gasta Claude donde el gusto,
el entorno o una perspectiva independiente pagan. En conflicto sobre algo que
se embarca: **inteligencia > gusto > costo**. La elección explícita del
usuario prevalece.

El MCP `cheap-coder` (`stealth/ox-alpha`) está **deprecado por ahora**: sus
tools pueden aparecer en la sesión, pero no están en la tabla.

## Esfuerzo

Selecciona modelo y esfuerzo por separado, usando sólo niveles admitidos por
cada modelo. El esfuerzo sigue la incertidumbre del encargo, no su tamaño.

| Esfuerzo | Criterio |
|----------|----------|
| low | Contexto suficiente para una solución directa y una comprobación sencilla. |
| medium | Hace falta interpretar, planificar o comparar alternativas; punto de partida habitual. |
| high | Persisten incertidumbres importantes o se necesita razonamiento profundo. |

El tope de cualquier despacho, sea quien sea el trabajador y aunque reanude,
es **high**.

Configura el esfuerzo explícitamente en el runtime; un default de
configuración no es un tope y pedirlo en el prompt no lo fija. Si se hereda,
comprueba el valor efectivo. Si la ruta no permite fijarlo, usa otra ruta o
resuelve directamente.

## Escalado

Tienes **permiso permanente** de escalar a un modelo o esfuerzo mejor cuando
el output no da la talla, sin preguntar. Escala ante contradicciones,
hipótesis pendientes o criterios de aceptación incumplidos por razonamiento;
entrega al nuevo intento la evidencia útil del anterior.

La falta de contexto, acceso o herramientas se resuelve aparte, no escalando.
Si ninguna opción de la tabla basta, devuelve el estado parcial y la decisión
pendiente.

## Revisión y UI/UX

Primer pase técnico: un revisor independiente con gpt-6-astra, esfuerzo según
los criterios anteriores. Un cambio de **alto riesgo** añade un pase opus-5
independiente: auth, permisos, migraciones de datos, releases y lo que el
proyecto clasifique así. Después el orquestador veta tanto el cambio como los
hallazgos.

UI/UX: primero un prototipo de fidelidad baja o media aprobado por el dueño.
Si faltan prototipo y guía, diseña opus-5. Implementa con los criterios
generales y verifica contra el diseño y sus estados de interacción.

## Transporte

Antes del primer despacho por una ruta en la sesión, lee su guía:

| Ruta | Guía |
|------|------|
| Codex → GPT (subagentes nativos) | `~/.agents/docs/native-subagents.md`, rama Codex |
| Claude Code → Claude (subagentes nativos) | `~/.agents/docs/native-subagents.md`, rama Claude Code |
| Codex → Claude (CLI externo) | `~/.codex/docs/claude-workers.md` |
| Claude Code → GPT (plugin o CLI) | `~/.claude/docs/codex-delegation.md` |

Las guías determinan mecanismos y parámetros; este documento determina la
selección. Comunica cualquier sustitución de modelo o esfuerzo.
