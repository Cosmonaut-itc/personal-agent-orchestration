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
| ox-alpha      | 10    | 9            | 6     |

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

## Worker gratis: ox-alpha (cheap-coder)

El servidor MCP `cheap-coder` expone `stealth/ox-alpha` como trabajador dentro
de esta rúbrica, bajo el mismo contrato Despachar → Recoger → Vetar → Cerrar.
Es gratis por token, así que lo único que gasta una tarea suya es **tiempo de
pared**: por la regla de la tabla, con inteligencia a la par de gpt-5.6-sol y
costo 0, es el primer candidato para todo lo que quepa en su transporte.

Ese transporte es el límite real, no la capacidad: solo se le alcanza por las
tres tools, cada una con su rol, su presupuesto y su worktree aislado —
exploración de repos (`cheap_coder_explore`), cambios que se despachan como
encargo autocontenido (`cheap_coder_implement`) y tests o reproducción de bugs
(`cheap_coder_test`). Lo que exige conversación, juicio de producto, UI/UX o
un review round se queda en los modelos premium de la tabla; el review final
lo manda siempre la secuencia gpt-5.6-sol + veto, nunca un worker.

Aún **no tiene perfil observado**: el modelo cambió el 2026-08-21 y la
telemetría anterior era de qwen3-coder-next. Hasta que haya corridas propias,
trátalo como capacidad no medida — despacha con criterio de éxito verificable
y presupuesto de pared corto, para que escale temprano y retomes tú en vez de
esperar 600 s por un parcial. El código sensible (auth, schema, migraciones)
sigue pidiendo el mismo pase premium que exige la política del proyecto,
delegado o no.

Todo lo que un worker barato produce se veta leyendo su diff completo antes
de integrar. Antes de despachar una tarea al MCP, lee
`~/.agents/docs/cheap-coder-workers.md` — contrato de input, Envelope,
presupuestos y cierre del worktree.
