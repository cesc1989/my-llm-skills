---
name: planea-trabajo-por-etapas
description: Usar cuando el usuario quiera planear trabajo sobre una característica o implementación del proyecto o lo que se indique. Dispara con "planea", "arma un plan", "haz un plan".
---

# Planea trabajo por etapas

1. Usa la skill `/revisa` para encontrar detalles a fondo del proyecto en curso.
2. Arma un plan por etapas de tal forma que se puedan ejecutar manualmente por el usuario.
3. Incluye código relevante cuando se cambien secciones existentes.
4. Si vas a escribir pruebas en Minitest o RSpec usa la forma donde todo va dentro del example. Para que sea autocontenido. Sigue el patrón [Four-phase test de Thoughtbot](https://thoughtbot.com/blog/four-phase-test).
5. Si incluyes código ERB o HTML, escríbelo poco apiñado para que quien lo revise lo lea fácil: línea en blanco entre elementos hermanos, sin línea en blanco justo después de abrir una etiqueta, el `>` de cierre en su propia línea cuando los atributos ocupan varias líneas, y el contenido de cada elemento en su propia línea, indentado. Por ejemplo:
   ```erb
   <div class="menu-link"
        data-controller="clipboard"
   >
     <div class="menu-link-title">
       <%= t(".title") %>
     </div>

     <strong>
       <%= path %>
     </strong>
   </div>
   ```
6. Crea el plan como un archivo .md en la carpeta `./docs`, `./plans` o la que esté disponible.
7. Resume al usuario el plan e indica donde puede encontrarlo.

Responde siempre en español y sé conciso.