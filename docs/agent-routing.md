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
