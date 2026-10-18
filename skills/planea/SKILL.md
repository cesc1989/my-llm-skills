---
name: planea-trabajo-por-etapas
description: Usar cuando el usuario quiera planear sobre una característica o implementación. Dispara con "planea", "arma un plan", "haz un plan".
---

# Planea trabajo por etapas

## Visión General

Escribe un plan de implementación para un ingeniero que no conoce este repositorio ni especificación. Asume que el ingenerio puede escribir código en el lenguaje del repositorio una vez que sepan la interfaz y test exacto y que tomaran una decisión razonable cuando el plan proponga una opción abierta.

Lo que el ingeniero no puede saber es qué archivos, qué nombres, qué valores de la especificación, qué pruebas (tests) verifican cada tarea. Documentalo. Sé conciso.

1. Usa la skill `/revisa` para encontrar detalles a fondo del proyecto en curso.
2. Arma un plan por etapas de tal forma que se puedan ejecutar manualmente por el ingeniero.
  2.1 Cada etapa se compone de tareas que se pueden completar de manera independiente entre sí.
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
  6.1 Guardalo en la carpeta `./docs/plans/YYYY-MM-DD-<feature-name>.md`
7. Resume al ingeniero el plan e indica donde puede encontrarlo.

## Tamaño correcto de una tarea

Una tarea es la unidad más pequeña que conlleva su propio ciclo de pruebas y es valedera para pasar por una revisión. Al delimitar una tarea, incluye en ella la preparación, la configuración, la creación de la estructura base y la documentación que requiera su entregable; Bifurca solo cuando el revisor pueda significativamente rechazar una tarea mientras aprueba la anterior o la siguiente. Cada tarea termina con una entretable que se puede testear independientemente.

## Cabecera del Documento del Plan

```markdown
# [Nombre de la Funcionalidad] Plan de Implementación

> **Para trabajadores agénticos:** SUB-HABILIDAD REQUERIDA: Usa superpowers:subagent-driven-development (recomendado) o superpowers:executing-plans para implementar este plan tarea por tarea. Los pasos usan casillas de verificación (`- [ ]`) para el seguimiento.

**Objetivo:** [Una oración describiendo qué construye esto]

**Arquitectura:** [2-3 oraciones sobre el enfoque]

**Stack Tecnológico:** [Tecnologías/bibliotecas clave]

**Especificación:** [ruta al documento de especificación/diseño que implementa este plan — el plan argumenta desde la especificación, así que la especificación viaja con él; los ejecutores leen ambos]

## Restricciones Globales

[Los requisitos a nivel de proyecto de la especificación — versiones mínimas, límites de dependencias, reglas de nomenclatura y texto, requisitos de plataforma — una línea cada uno, con valores exactos copiados textualmente de la especificación. Los requisitos de cada tarea incluyen implícitamente esta sección.]

## Enfoque de Revisión

[Las cinco clases de entrada o modos de fallo que la especificación implica pero que las pruebas de ninguna tarea ejercitan y que son más propensos a afectar a una persona usando este software — una línea cada uno, nombrando la entrada o condición y el comportamiento que una persona razonable esperaría, los más probables primero. La especificación es un documento de visión: dice qué debe hacer el software, no todo lo que encontrará, y su silencio sobre una entrada no es permiso para que esa entrada rompa el programa. Escribe la lista aquí, una vez, con la especificación frente a ti. Luego, para cada línea, agrega la prueba que la fija a la tarea que posee el código, en el estilo de pasos propio de esa tarea.]

---
```