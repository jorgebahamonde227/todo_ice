# Diagramas de Flujo - Gestor de Tareas ICE

Este documento describe los flujos funcionales del MVP segun `MVP_ALCANCE_FUNCIONAL.md`, la guia tecnica `GUIA_CONTROL_DESARROLLO_REACT.md` y el mockup visual `pantallas-gestor-tareas-ice-mui.png`.

El producto se plantea como una aplicacion React de una sola pantalla principal, apoyada en componentes de Material UI y dialogos modales para crear, editar y confirmar sugerencias ICE.

## Diagrama: Crear tarea con ICE sugerido por IA

```mermaid
flowchart TD
  Start([Usuario entra a la app])
  Start --> App["App carga ThemeProvider y TaskProvider"]
  App --> Load["TaskContext carga tareas desde localStorage"]
  Load --> Dashboard["Dashboard principal con AppHeader, TaskFilters y TaskTable"]
  Dashboard --> NewTask["Usuario pulsa Nueva tarea"]
  NewTask --> Form["Abrir TaskFormDialog en modo crear"]
  Form --> Fill["Usuario introduce titulo y descripcion"]
  Fill --> ValidateAI{Descripcion valida para IA?}
  ValidateAI -- No --> FormAlert["Mostrar Alert: la descripcion es obligatoria"]
  FormAlert --> Fill
  ValidateAI -- Si --> AskAI["Usuario pulsa Calcular ICE con IA"]
  AskAI --> Loading["Mostrar CircularProgress y deshabilitar acciones sensibles"]
  Loading --> AIService["aiService envia descripcion a la API de IA"]
  AIService --> ValidResponse{Respuesta JSON valida?}
  ValidResponse -- Si --> Suggestion["Abrir PriorityModal con impact, confidence, ease, reasoning e ICE calculado"]
  ValidResponse -- No --> AIError["Mostrar Alert: error IA o respuesta invalida"]
  AIError --> ManualScores["Usuario introduce Impacto, Confianza y Facilidad manualmente"]
  Suggestion --> UserDecision{Usuario decide}
  UserDecision -- Confirmar tarea --> ApplySuggestion["Aplicar valores sugeridos al formulario"]
  UserDecision -- Ajustar valores --> Adjust["Usuario modifica scores 1-10"]
  UserDecision -- Cancelar --> Form
  Adjust --> Recalculate["Recalcular ICE con calculateIceScore"]
  ApplySuggestion --> SaveValidation{Formulario valido?}
  Recalculate --> SaveValidation
  ManualScores --> SaveValidation
  SaveValidation -- No --> ValidationAlert["Mostrar errores de titulo, descripcion o scores"]
  ValidationAlert --> Form
  SaveValidation -- Si --> Create["Dispatch task/created"]
  Create --> Persist["storageService guarda tareas en localStorage"]
  Persist --> Sort["TaskTable muestra tareas filtradas y ordenadas por ICE descendente"]
  Sort --> Snackbar["Snackbar: tarea guardada"]
  Snackbar --> Dashboard
```

## Diagrama: Crear tarea manual sin IA

```mermaid
flowchart TD
  Dashboard["Dashboard principal"] --> NewTask["Nueva tarea"]
  NewTask --> Form["TaskFormDialog"]
  Form --> RequiredFields["Completar titulo y descripcion"]
  RequiredFields --> ManualICE["Introducir Impacto, Confianza y Facilidad de 1 a 10"]
  ManualICE --> Calc["Calcular ICE = promedio de I + C + E"]
  Calc --> Save{Datos validos?}
  Save -- No --> Errors["Mostrar Alert con errores de validacion"]
  Errors --> Form
  Save -- Si --> Create["Crear tarea pendiente"]
  Create --> Persist["Persistir en localStorage"]
  Persist --> List["Volver a TaskTable ordenada por mayor ICE"]
```

## Diagrama: Navegacion del usuario en el MVP

```mermaid
flowchart LR
  App["App"] --> Header["AppHeader / Navbar MUI"]
  App --> Main["Dashboard principal"]

  Header --> Title["Nombre de la app"]
  Header --> NewButton["Button: Nueva tarea"]

  Main --> Filters["TaskFilters: Todas, Pendientes, Completadas"]
  Main --> ListState{Hay tareas?}

  ListState -- No --> Empty["EmptyState: Aun no hay tareas"]
  ListState -- Si --> Table["TaskTable"]

  Table --> Row["TaskTableRow / TaskCard"]
  Row --> Ice["IceScoreChips"]
  Row --> Toggle["Completar / Reabrir"]
  Row --> Edit["Editar"]
  Row --> Delete["Eliminar"]

  NewButton --> CreateDialog["TaskFormDialog modo crear"]
  Edit --> EditDialog["TaskFormDialog modo editar"]
  CreateDialog --> Priority["PriorityModal sugerencia IA"]
  EditDialog --> Priority

  Toggle --> Table
  Delete --> Confirm["ConfirmDialog"]
  Confirm --> Table
  Priority --> CreateDialog
  Priority --> EditDialog
  CreateDialog --> Table
  EditDialog --> Table
```

## Diagrama: Estados de interfaz

```mermaid
stateDiagram-v2
  [*] --> LoadingStorage
  LoadingStorage --> EmptyList: sin tareas
  LoadingStorage --> TaskList: tareas cargadas

  EmptyList --> Creating: Nueva tarea
  TaskList --> Creating: Nueva tarea
  TaskList --> Editing: Editar tarea

  Creating --> CalculatingAI: Calcular ICE con IA
  Editing --> CalculatingAI: Recalcular ICE con IA
  CalculatingAI --> AISuggestion: respuesta valida
  CalculatingAI --> AIError: error API o JSON invalido

  AISuggestion --> Creating: ajustar o confirmar valores
  AISuggestion --> Editing: ajustar o confirmar valores
  AIError --> Creating: entrada manual
  AIError --> Editing: entrada manual

  Creating --> TaskList: guardar tarea valida
  Editing --> TaskList: guardar cambios validos
  TaskList --> EmptyList: eliminar ultima tarea
```

## Notas de alcance

- No hay rutas, autenticacion, backend, paginacion, analitica ni settings en el MVP.
- La navegacion real ocurre mediante estado local/global, filtros y dialogos MUI.
- La IA solo sugiere valores ICE; el usuario siempre puede confirmar, ajustar o introducir valores manuales.
- El score se calcula como promedio: `(Impacto + Confianza + Facilidad) / 3`, con un decimal.
- Las tareas se muestran por defecto ordenadas de mayor a menor `iceScore`.
