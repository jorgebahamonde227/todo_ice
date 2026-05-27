# Tareas de Implementacion - Gestor de Tareas ICE

Plan ordenado para construir el MVP en React + TypeScript + Material UI, siguiendo `MVP_ALCANCE_FUNCIONAL.md`, `GUIA_CONTROL_DESARROLLO_REACT.md`, `docs/flow_diagrams.md` y `docs/jerarquia_componentes.md`.

## 1. Preparar base del proyecto React + Material UI

**Objetivo:** dejar el proyecto listo para desarrollar con React, TypeScript estricto y Material UI.

**Incluye:**

- Crear o validar proyecto Vite con React 18+ y TypeScript.
- Instalar dependencias aprobadas: `@mui/material`, `@emotion/react`, `@emotion/styled`.
- Instalar `@mui/icons-material` solo si se decide usar iconos MUI.
- Crear `src/app/theme.ts` con tema centralizado.
- Envolver la app con `ThemeProvider` y `CssBaseline`.
- Crear estructura inicial de carpetas:
  - `app/`
  - `components/layout/`
  - `components/tasks/`
  - `context/`
  - `hooks/`
  - `services/`
  - `types/`
  - `utils/`

**Criterios de aceptacion:**

- La app arranca sin errores.
- TypeScript compila.
- Material UI esta disponible y aplicado desde un tema central.

## 2. Definir modelo de dominio y utilidades ICE

**Objetivo:** establecer los tipos y reglas puras que usara toda la aplicacion.

**Incluye:**

- Crear `src/types/task.ts`.
- Definir `Task`, `TaskFilter` y tipos auxiliares para formularios o respuestas IA.
- Crear `src/utils/ice.ts`.
- Implementar `calculateIceScore(impact, confidence, ease)`.
- Implementar `isValidScore(value)`.
- Definir constantes para rangos validos, por ejemplo `MIN_SCORE = 1` y `MAX_SCORE = 10`.

**Criterios de aceptacion:**

- El score ICE se calcula como promedio de Impacto, Confianza y Facilidad.
- El resultado se muestra con un decimal.
- No hay reglas ICE duplicadas fuera de `utils/ice.ts`.

## 3. Implementar estado global con Context + useReducer y persistencia local

**Objetivo:** centralizar las tareas, filtros y acciones de dominio.

**Incluye:**

- Crear `src/context/TaskContext.tsx`.
- Implementar reducer con acciones:
  - `tasks/loaded`
  - `task/created`
  - `task/updated`
  - `task/deleted`
  - `task/toggled`
  - `filter/changed`
  - `error/set`
  - `error/cleared`
- Crear `src/hooks/useTasks.ts`.
- Crear `src/services/storageService.ts`.
- Leer tareas desde `localStorage` al iniciar.
- Guardar tareas tras crear, editar, eliminar o completar.
- Recuperarse de JSON invalido devolviendo lista vacia.

**Criterios de aceptacion:**

- Las tareas sobreviven al refrescar el navegador.
- El reducer es puro.
- Ningun componente accede directamente a `localStorage`.

## 4. Construir layout principal y navegacion de una sola pantalla

**Objetivo:** montar la estructura visual base del mockup con Material UI.

**Incluye:**

- Crear `AppHeader` con `AppBar`, `Toolbar`, `Typography` y boton `Nueva tarea`.
- Crear layout en `App.tsx` usando `Container`, `Stack`, `Box` y/o `Paper`.
- Crear `TaskFilters` con `ToggleButtonGroup`.
- Crear `EmptyState` para lista vacia.
- Controlar desde `App` la apertura de dialogos de crear, editar y eliminar.

**Criterios de aceptacion:**

- La app muestra una unica pantalla principal.
- El boton `Nueva tarea` abre el dialogo de formulario.
- El estado vacio tiene una llamada clara para crear la primera tarea.
- No se introducen rutas ni pantallas fuera de alcance.

## 5. Implementar formulario de crear/editar tarea

**Objetivo:** permitir alta y edicion de tareas con valores ICE manuales.

**Incluye:**

- Crear `TaskFormDialog`.
- Usar `Dialog`, `DialogTitle`, `DialogContent`, `TextField`, `Slider`, `Button`, `Alert` y `CircularProgress` si aplica.
- Campos:
  - Titulo.
  - Descripcion.
  - Impacto.
  - Confianza.
  - Facilidad.
  - Estado completada, solo si se edita o si se decide mostrarlo.
- Validar titulo obligatorio.
- Validar scores enteros de 1 a 10.
- Recalcular ICE al modificar Impacto, Confianza o Facilidad.
- Soportar modo crear y modo editar con el mismo componente.

**Criterios de aceptacion:**

- El usuario puede crear una tarea manualmente.
- El usuario puede editar titulo, descripcion, scores y estado.
- Los errores aparecen como estados de UI, no solo en consola.
- Al guardar, la tarea aparece en la lista con su ICE calculado.

## 6. Implementar listado, ordenacion y acciones de tareas

**Objetivo:** mostrar tareas priorizadas y permitir gestionarlas desde la tabla.

**Incluye:**

- Crear `TaskTable`.
- Crear `TaskTableRow`.
- Crear `IceScoreChips`.
- Mostrar:
  - Titulo.
  - Descripcion resumida.
  - Impacto.
  - Confianza.
  - Facilidad.
  - ICE total.
  - Estado pendiente/completada.
  - Acciones editar, eliminar y completar/reabrir.
- Ordenar por defecto de mayor a menor `iceScore`.
- Filtrar por todas, pendientes y completadas.
- Crear `ConfirmDialog` para eliminar.
- Crear feedback con `Snackbar` o `Alert`.

**Criterios de aceptacion:**

- Las tareas aparecen ordenadas por prioridad ICE.
- El usuario puede completar, reabrir, editar y eliminar.
- Los filtros cambian la lista visible sin perder datos.
- La ultima tarea eliminada devuelve la interfaz al estado vacio.

## 7. Integrar calculo ICE con IA y pulir estados de interfaz

**Objetivo:** conectar la sugerencia IA y cerrar el flujo completo del MVP.

**Incluye:**

- Crear `src/services/aiService.ts`.
- Leer la clave desde `VITE_GEMINI_API_KEY`.
- Enviar la descripcion de la tarea a la API de IA.
- Solicitar respuesta JSON con:
  - `impact`
  - `confidence`
  - `ease`
  - `reasoning`
- Validar que los scores son numeros enteros entre 1 y 10.
- Crear o completar `PriorityModal`.
- Mostrar loading mientras calcula.
- Mostrar error si falla la API o la respuesta es invalida.
- Permitir entrada manual si la IA falla.
- Permitir confirmar o ajustar valores antes de guardar.
- Revisar responsive basico en desktop y movil.

**Criterios de aceptacion:**

- El usuario puede escribir una descripcion y pedir `Calcular ICE con IA`.
- La sugerencia IA rellena Impacto, Confianza y Facilidad.
- El usuario puede confirmar o ajustar antes de guardar.
- La app no queda bloqueada si la IA falla.
- El flujo completo coincide con los diagramas de `docs/flow_diagrams.md`.

## Orden recomendado de verificacion

1. La app compila y renderiza el layout principal.
2. Crear una tarea manual.
3. Editar la tarea.
4. Completar y reabrir la tarea.
5. Eliminar la tarea.
6. Confirmar persistencia con `localStorage`.
7. Probar filtros.
8. Probar calculo ICE con IA.
9. Probar error de IA y entrada manual.
10. Revisar que no hay funcionalidades fuera de alcance.
