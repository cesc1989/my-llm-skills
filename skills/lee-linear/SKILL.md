---
name: lee-linear
description: Usar cuando el usuario quiere que revises un ticket/issue en Linear. El usuario te pasará un ID de ticket o debes pedirselo. Dispara cuando el usuario diga "Lee Linear [ID]"
---

# Lee Linear

Los casos están en Linear y tienen un identificador para que lo ubiques mediante el MCP. Revisa todo lo necesario.

## Consideraciones

- No tienes acceso a la consola de Rails en producción así que da los comandos y el usuario te da las salidas.
- Solo dale un par de comandos/consultas de verificación a la vez.
- Si vas a construir un comando de un modelo, asegurate de revisar el schema y el modelo para que no le des campos que no existen o campos que son funciones.
