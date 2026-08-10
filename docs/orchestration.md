# Contrato de orquestación

Fuente canónica para Codex y Claude Code. Los archivos globales de cada
harness sólo deciden cómo transportar una delegación; este archivo define el
contrato común.

## Rol

- **Orquestador:** recibió la tarea principal del usuario. Conserva ownership,
  decide si delegar y veta el resultado final.
- **Trabajador:** recibió un encargo de otro agente. Resuelve ese encargo y
  devuelve evidencia al orquestador.
- **Antibucle:** el trabajador no invoca al harness que lo lanzó ni abre un
  puente de vuelta. Toda delegación entre harnesses permanece en el
  orquestador.

## Contrato

Una delegación termina sólo cuando el orquestador completa estas cuatro fases:

1. **Despachar:** enviar una tarea autocontenida con objetivo, contexto
   imprescindible, alcance de archivos/sistemas, restricciones, criterios de
   aceptación y formato de respuesta.
2. **Recoger:** esperar el estado terminal y capturar resultado, evidencia,
   errores y cambios producidos.
3. **Vetar:** leer el resultado y el diff completo cuando exista; contrastar
   afirmaciones, pruebas y criterios de aceptación. Reorientar o repetir el
   trabajo que no dé la talla.
4. **Cerrar:** integrar sólo el resultado validado, informar el estado real y
   cerrar o detener el trabajador.

## Routing

Antes de elegir agente o modelo, esfuerzo, secuencia de review o tratamiento
UI/UX, lee `~/.agents/docs/agent-routing.md` y aplica completa su tabla y sus
reglas. Usa después la guía operativa del harness que realiza el transporte.
