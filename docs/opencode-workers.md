# Despachar trabajadores mediante OpenCode Go

Referencia técnica consultada: 2026-09-10. Instalación: `~/.agents/docs/opencode-workers.md`.
Estado de las pruebas y alcance autorizado: véase «Estado de validación».

Lee esta guía antes de despachar desde Codex o Claude Code a un cheap coder
de Go. La selección pertenece a `agent-routing.md`; esta guía define el
transporte. El orquestador conserva la aceptación del resultado.

## Ruta elegida

Usar **OpenCode CLI como trabajador externo**. OpenCode proporciona el bucle
de herramientas, archivos y sesiones. Una petición directa al endpoint del
modelo sólo genera una respuesta; integrar ese endpoint exigiría construir
también ese bucle. Para muchos encargos, la alternativa es el servidor HTTP
de OpenCode. El SDK oficial envuelve esa API.

La ruta CLI evita un servicio persistente para despachos individuales; HTTP
facilita identificar una sesión antes de enviar el encargo y cancelarla por
ID. Comenzar con CLI y conservar la alternativa HTTP documentada.

## Preparar

1. Comprobar ejecutable y versión con `command -v opencode` y
   `opencode --version`. Verificar las opciones de la versión instalada con
   `opencode run --help`. Instalar/configurar sólo dentro del alcance aprobado.
2. Conectar el proveedor Go por su flujo de autenticación y comprobar
   `opencode models opencode-go --verbose`. Registrar IDs y variantes; nunca
   incluir credenciales en prompts, repositorio o evidencia.
3. Validar los perfiles con `opencode agent list`, la configuración efectiva
   y el catálogo. El modelo y perfil solicitados deben existir antes de lanzar.
4. Asignar directorio real, archivos permitidos y presupuesto. Dar al
   trabajador un encargo autocontenido con contexto, criterios de aceptación,
   comprobaciones y formato de resultado. No hereda el chat de origen.

Comandos de catálogo, ejecución, reanudación y exportación se documentan en
la [referencia CLI](https://opencode.ai/docs/cli/).

| Perfil | Modelo explícito |
|---|---|
| `cheap-coder-deepseek` | `opencode-go/deepseek-v4.1-flash` |
| `cheap-coder-minimax` | `opencode-go/minimax-m3` |
| `cheap-coder-kimi` | `opencode-go/kimi-k3` |

El [catálogo público](https://opencode.ai/zen/go/v1/models) consultado publica
los tres IDs y también `deepseek-flash`. Preferir el ID versionado de DeepSeek;
confirmar su resolución efectiva antes de usarlo. Publicado no significa
autenticado ni probado en esta cuenta.

## Perfil y controles efectivos

Los perfiles y `worker-prompt.md` están en `configs/opencode/` del repositorio.
Conserva el prompt junto al JSON al instalar. El perfil Kimi está habilitado
con `disable: false`, low de inicio y high al escalar. El dueño autorizó su
uso el 2026-09-10 tras verificar envío y aceptación de ambas variantes por Go,
aceptando que el esfuerzo interno aplicado no tiene confirmación observable.
Esta limitación no bloquea el despacho; evalúa los resultados de cada encargo.

Los perfiles usan `mode: all`: pueden actuar como agentes directos del
runtime OpenCode y como subagentes cuando corresponda. Son trabajadores
delegados desde el punto de vista de Codex/Claude Code. **`mode: subagent`
no sirve para el lanzamiento directo con `opencode run --agent`**: el código
examinado avisa y recurre al agente por defecto. También lo hace si el nombre
no existe. Verificar antes del despacho y tratar cualquier sustitución como
fallo de transporte. [Código CLI](https://github.com/anomalyco/opencode/blob/dev/packages/opencode/src/cli/cmd/run.ts).

Configurar permisos concretos para el encargo y denegar `task` si no se
autorizaron descendientes. En un encargo de lectura, denegar edición y shell
de escritura; en uno de implementación, autorizar sólo superficie y comandos
necesarios. El directorio de trabajo y un worktree no constituyen un sandbox.
El alcance también debe cubrir procesos lanzados desde shell y MCP heredados.

Los perfiles base permiten lectura y búsqueda; deniegan otras herramientas.
Para autorizar una implementación, combina en `OPENCODE_CONFIG_CONTENT` las
reglas del perfil seleccionado con rutas y comandos concretos. Ejemplo para
un encargo que autoriza sólo editar `src/formatter.ts` y ejecutar su prueba:

```json
{
  "agent": {
    "cheap-coder-deepseek": {
      "permission": {
        "edit": {
          "*": "deny",
          "/ruta/absoluta/worktree/src/formatter.ts": "allow"
        },
        "bash": {
          "*": "deny",
          "npm test -- formatter.test.ts": "allow"
        }
      }
    }
  }
}
```

Sustituye el ejemplo por el alcance y la comprobación reales del repositorio.
El programa de pruebas y sus dependencias pueden escribir archivos: permitir
el comando exige considerar esos efectos. Inspecciona las reglas combinadas
antes de enviar el encargo; no uses este ejemplo como permiso universal.

Presupuesto inicial: `steps: 20` y plazo externo de 10 minutos por despacho,
ajustables al preparar el encargo. `steps` limita iteraciones, no tokens ni
dólares. Un agotamiento devuelve trabajo parcial para veto, no éxito.
[Agentes y permisos](https://opencode.ai/docs/agents/).

La configuración se combina con otras capas. Los overrides de ejecución
`OPENCODE_CONFIG_CONTENT` tienen precedencia; `OPENCODE_CONFIG` por sí solo
puede ser sobrescrito por configuración del proyecto. Inspeccionar el
resultado efectivo y mantener los cambios del despacho fuera de la
configuración global. [Precedencia](https://opencode.ai/docs/config/).

| Modelo | Control y evidencia |
|---|---|
| DeepSeek V4.1 Flash | `--variant low` de inicio; `high` si hace falta. Verificar que se transmite `reasoning_effort`; DeepSeek mapea medium a high. |
| MiniMax M3 | `--variant thinking` para coding; `none` sólo para encargos apropiados. OpenCode traduce thinking a modo adaptativo, no a un nivel high. |
| Kimi K3 | `reasoningEffort: "low"` en el perfil; variantes explícitas low/high. Usar `--variant low` de inicio y high al escalar; max/xhigh deshabilitados. Envío y aceptación verificados en Go; nivel interno aplicado no observable. |

Fuentes: [DeepSeek](https://api-docs.deepseek.com/guides/thinking_mode/),
[transformaciones de OpenCode](https://github.com/anomalyco/opencode/blob/dev/packages/opencode/src/provider/transform.ts),
[controles Kimi K3](https://www.kimi.com/code/docs/en/kimi-code/models.html).
K3 documenta low/high/max en su API; este contrato limita su uso a low/high.
El transporte Go se comprobó para low/high con el alcance descrito abajo. La excepción
de modo adaptativo corresponde a MiniMax, no a K3.

El código examinado no crea variantes automáticas para Kimi sobre el SDK
compatible con OpenAI. Por ello la configuración incluye variantes propias
low/high en `provider.opencode-go.models.kimi-k3.variants`: instálalas junto
al perfil. El modelo del catálogo sigue siendo `opencode-go/kimi-k3`.
[Variantes personalizadas](https://opencode.ai/docs/models/#custom-variants).

El SDK compatible convierte `reasoningEffort` a `reasoning_effort` y el proxy
Go preserva el cuerpo. El perfil conserva `thinking: {type: "enabled"}`.
La inspección estática y las capturas de ejecución acreditan el transporte
de opciones. Las respuestas observadas no confirman su semántica interna.
[Normalización](https://github.com/anomalyco/opencode/blob/dev/packages/core/src/v1/config/agent.ts),
[SDK compatible 2.0.41](https://unpkg.com/@ai-sdk/openai-compatible@2.0.41/dist/index.mjs),
[cuerpo de petición Go](https://github.com/anomalyco/opencode/blob/dev/packages/console/app/src/routes/zen/util/requestBody.ts).
El perfil habitual queda habilitado por autorización del dueño con la
limitación indicada; conserva modelo y variante en la evidencia del encargo.

Entre los tres trabajadores seleccionados, K3 ocupa el puesto de mayor
capacidad. Moonshot lo presenta como su modelo de programación más capaz,
con contexto de hasta 1M en su servicio;
verifica el límite expuesto por Go antes de asignar contexto. Su mayor tarifa
y menor cuota justifican reservarlo para encargos que superen la capacidad
adecuada de los trabajadores económicos. Inteligencia/gusto en el selector
siguen siendo estimaciones operativas, no una comparación local medida.
[Modelo K3](https://www.kimi.com/code/docs/en/kimi-code/models.html).

## Despachar por CLI

Ejemplo con perfil ya validado, rutas sustituidas por las reales
y directorio de evidencia existente:

```sh
opencode run \
  --dir /ruta/absoluta/worktree \
  --agent cheap-coder-deepseek \
  --model opencode-go/deepseek-v4.1-flash \
  --variant low \
  --format json \
  < /ruta/absoluta/evidencia/encargo.md \
  > /ruta/absoluta/evidencia/eventos.jsonl \
  2> /ruta/absoluta/evidencia/stderr.log
```

Para un encargo K3, conserva el mismo procedimiento y usa
`--agent cheap-coder-kimi --model opencode-go/kimi-k3 --variant low`; requiere
perfil y variantes instalados. Al reanudar, conserva
el ID de sesión y fija nuevamente modelo y esfuerzo; no cambies una sesión
K2.7 a K3 para empezar el encargo nuevo.

El lanzador programático debe ejecutar el binario con un **array de argumentos**,
`cwd` explícito y el archivo de encargo por stdin. Conservar PID o ID del
proceso del harness y su código de salida. No interpolar el prompt como código
de shell. OpenCode acepta stdin; JSON emite registros con `sessionID`.
Los eventos observados en el código incluyen `text`, `tool_use`, `step_finish`
y `error`; no asumir un evento ficticio de tarea aceptada.
[Implementación de run](https://github.com/anomalyco/opencode/blob/dev/packages/opencode/src/cli/cmd/run.ts).

En ejecución no interactiva, los permisos pendientes pueden ser rechazados.
Preparar los permisos autorizados antes de lanzar; un rechazo se conserva
como evidencia. No resolverlo habilitando aprobación indiscriminada.

## Recoger, continuar y detener

- Seguir el proceso hasta terminación. Conservar stdout, stderr, código de
  salida y `sessionID`, incluyendo errores y comprobaciones fallidas.
- Exportar con `opencode export <sessionID>` y verificar en los mensajes el
  proveedor/modelo/agente usado. Conservar la configuración efectiva redactada
  para acreditar controles solicitados; si no hay lectura del modo realmente
  aplicado, reportar esa limitación.
- Reanudar el mismo encargo con `opencode run --session <sessionID>`, repitiendo
  directorio, perfil, modelo, controles y `--format json`; enviar la corrección
  por stdin y guardar nueva evidencia. Evitar `--continue`, que selecciona una
  sesión por recencia.
- Para ejecución CLI independiente, el lanzador debe disponer de un grupo de
  procesos propio: al cancelar, detenerlo y verificar la salida y los procesos
  hijos. Si se usa servidor, cancelar la sesión por la API antes de cerrar el
  cliente; matar un cliente conectado no demuestra que el servidor se detuvo.
- Revisar diff completo y criterios. Exit code 0, texto final o fin de un paso
  no acreditan aceptación. El orquestador decide aceptar, corregir o rechazar;
  conserva evidencia antes de retirar recursos.

## Alternativa HTTP para despacho asíncrono

Ejecutar `opencode serve --hostname 127.0.0.1 --port <puerto>` desde el
directorio asignado; identificar el proceso y protegerlo con autenticación.
Consultar `/doc` de esa versión para validar el esquema, incluida la variante.

| Operación | Petición |
|---|---|
| Crear e identificar encargo | `POST /session` con título; guardar `id`. |
| Despachar | `POST /session/:id/prompt_async` con agente, modelo y partes. |
| Recoger | `GET /session/status`, `GET /session/:id/message` y eventos SSE. |
| Continuar | Otro prompt a la misma sesión, con selección explícita. |
| Cancelar | `POST /session/:id/abort`; comprobar estado y procesos iniciados. |

El cuerpo del mensaje identifica el modelo así:

```json
{
  "agent": "cheap-coder-deepseek",
  "model": {
    "providerID": "opencode-go",
    "modelID": "deepseek-v4.1-flash"
  },
  "variant": "low",
  "parts": [{ "type": "text", "text": "Encargo autocontenido aprobado" }]
}
```

Un HTTP 204 sólo acredita envío. La recogida debe distinguir respuesta final,
estado idle, errores y aceptación. Usar un servidor por directorio cuando se
busque separación sencilla; no mutar `/config` de un servidor compartido para
cambiar el modelo entre encargos. [API](https://opencode.ai/docs/server/),
[SDK](https://opencode.ai/docs/sdk/).

## Estado de validación

Pruebas ejecutadas el 2026-09-10 con OpenCode 1.18.30 y la cuenta Go conectada:

- DeepSeek V4.1 Flash, MiniMax M3 y Kimi K3 leyeron un archivo y resolvieron
  correctamente el cálculo solicitado; las sesiones exportadas confirmaron
  agente, proveedor, modelo y variante.
- Kimi K3 respondió correctamente al mismo problema con low y high. La captura
  de la llamada nativa confirmó `reasoning_effort: low` y `high`, HTTP 200
  y modelo de respuesta `kimi-k3`. La revisión independiente aceptó ese alcance.
- Las respuestas no ofrecieron una lectura explícita del esfuerzo interno
  aplicado. Diferencias de tokens o duración no demuestran ese nivel. El dueño
  aceptó la limitación y autorizó habilitar Kimi para evaluar encargos reales.

El envío y la aceptación están verificados; la falta de lectura del nivel
interno no bloquea el uso autorizado de Kimi. Calibra resultados observados
sin convertir esta aceptación operativa en una confirmación del proveedor.

Reanudación, cancelación y rechazo de perfiles inválidos siguen sin prueba
específica de aceptación. Al usar esas operaciones, aplica los controles de
esta guía y comprueba el resultado. Los perfiles están en `configs/opencode/`.
Esta guía no implementa un lanzador: el runtime que despacha debe controlar
procesos, perfil efectivo y plazo desde el primer encargo.

## Tarifas y cuota

Referencia del 2026-09-10: Go cuesta **10 USD/mes por suscripción**. Tarifas
publicadas en USD por millón de tokens; son valores de consumo incluido.

| Modelo | Entrada | Salida | Lectura de caché | Cuota mensual equivalente usando ese modelo |
|---|---:|---:|---:|---:|
| DeepSeek V4.1 Flash, valle / pico | 0.15 / 0.30 | 0.60 / 1.20 | 0.003 / 0.006 | 15 base; 60 durante promoción |
| MiniMax M3 | 0.30 | 1.20 | 0.06 | 60 |
| Kimi K3 | 3.00 | 15.00 | 0.30 | 15 |

Las ventanas son 20% cada cinco horas, 50% semanal y 100% mensual. DeepSeek
aplica pico lunes–viernes 01:00–04:00 y 06:00–10:00 UTC; el resto es valle.
Revisa las [tarifas vigentes](https://opencode.ai/docs/go/) antes de estimar.

La [portada](https://opencode.ai/go) anunció 4× temporal para V4.1 Flash sin
vencimiento exacto publicado. Confirma su vigencia antes de contabilizarla.
Los modelos comparten los contadores de Go, con multiplicadores por modelo;
las cuotas de la tabla no se suman. [Contabilización](https://github.com/anomalyco/opencode/blob/43e89ea1673937c325edb922eaa1a768aa0e5bf4/packages/console/app/src/routes/zen/util/handler.ts#L1184).

El coste aceptado incluye reintentos y revisión. Las solicitudes estimadas no
equivalen a tareas terminadas. Al agotar Go, informa el límite y aplica el
escalado del routing con el presupuesto disponible; activar saldo Zen adicional
debe estar dentro de la autorización vigente.
