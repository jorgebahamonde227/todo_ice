# Jerarquia de Pantallas y Componentes - Gestor de Tareas ICE

Documento de estructura funcional y visual basado en `MVP_ALCANCE_FUNCIONAL.md`, `GUIA_CONTROL_DESARROLLO_REACT.md` y el mockup `pantallas-gestor-tareas-ice-mui.png`.

La app se plantea como una unica vista principal con dialogos modales. Esta decision mantiene el MVP simple para el curso, evita rutas innecesarias y encaja con Material UI.

## Mapa de pantallas del MVP

```text
App
├── Dashboard principal
│   ├── AppHeader / Navbar
│   ├── Acciones principales
│   ├── TaskFilters
│   ├── TaskTable
│   └── EmptyState
├── TaskFormDialog
│   ├── Modo crear
│   └── Modo editar
├── PriorityModal
│   ├── Estado cargando IA
│   ├── Estado sugerencia valida
│   └── Estado error IA / respuesta invalida
└── ConfirmDialog
    └── Confirmar eliminacion
```

## Jerarquia tecnica recomendada

```text
src/
├── app/
│   ├── App.tsx
│   └── theme.ts
├── components/
│   ├── layout/
│   │   └── AppHeader.tsx
│   └── tasks/
│       ├── EmptyState.tsx
│       ├── IceScoreChips.tsx
│       ├── PriorityModal.tsx
│       ├── TaskFilters.tsx
│       ├── TaskFormDialog.tsx
│       ├── TaskTable.tsx
│       └── TaskTableRow.tsx
├── context/
│   └── TaskContext.tsx
├── hooks/
│   └── useTasks.ts
├── services/
│   ├── aiService.ts
│   └── storageService.ts
├── types/
│   └── task.ts
├── utils/
│   └── ice.ts
└── main.tsx
```

## Componentes principales

| Componente | Rol UX | Componentes MUI sugeridos |
| --- | --- | --- |
| `App` | Componer tema, provider, layout principal y dialogos abiertos | `ThemeProvider`, `CssBaseline`, `Container`, `Stack`, `Box` |
| `AppHeader` | Identificar la app y ofrecer accion principal | `AppBar`, `Toolbar`, `Typography`, `Button` |
| `TaskFilters` | Cambiar entre todas, pendientes y completadas | `ToggleButtonGroup`, `ToggleButton`, `Stack` |
| `TaskTable` | Mostrar tareas filtradas y ordenadas por prioridad ICE | `Table`, `TableHead`, `TableBody`, `TableRow`, `TableCell`, `Paper` |
| `TaskTableRow` | Representar una tarea y sus acciones | `TableRow`, `TableCell`, `Checkbox`, `IconButton`, `Tooltip`, `Chip` |
| `TaskFormDialog` | Crear o editar una tarea con valores ICE manuales o sugeridos | `Dialog`, `DialogTitle`, `DialogContent`, `TextField`, `Slider`, `Button`, `Alert`, `CircularProgress` |
| `PriorityModal` | Confirmar o ajustar la sugerencia IA antes de guardar | `Dialog`, `Chip`, `Alert`, `Stack`, `Button`, `CircularProgress` |
| `IceScoreChips` | Visualizar Impacto, Confianza, Facilidad e ICE total | `Chip`, `Tooltip`, `Box` |
| `EmptyState` | Guiar cuando no existen tareas | `Paper`, `Typography`, `Button` |
| `ConfirmDialog` | Evitar borrados accidentales | `Dialog`, `DialogActions`, `Button` |
| `SnackbarFeedback` | Confirmar guardado, eliminacion o errores generales | `Snackbar`, `Alert` |

## Layout visual segun el mockup PNG

```text
┌────────────────────────────────────────────────────────────┐
│ AppBar: Gestor de Tareas ICE                 Nueva tarea   │
├────────────────────────────────────────────────────────────┤
│ Container                                                  │
│                                                            │
│  ToggleButtonGroup: Todas | Pendientes | Completadas       │
│                                                            │
│  Paper / TaskTable                                         │
│  ┌──────────────┬──────────┬──────────────┬─────────────┐  │
│  │ Tarea        │ ICE      │ Estado       │ Acciones    │  │
│  ├──────────────┼──────────┼──────────────┼─────────────┤  │
│  │ Titulo       │ Chips    │ Checkbox     │ Edit/Delete │  │
│  │ Descripcion  │ I C E    │ Pendiente    │ Recalcular  │  │
│  └──────────────┴──────────┴──────────────┴─────────────┘  │
│                                                            │
└────────────────────────────────────────────────────────────┘

Dialog: TaskFormDialog
┌────────────────────────────────────┐
│ Nueva tarea / Editar tarea          │
│ TextField: Titulo                   │
│ TextField: Descripcion              │
│ Slider/TextField: Impacto 1-10      │
│ Slider/TextField: Confianza 1-10    │
│ Slider/TextField: Facilidad 1-10    │
│ Alert error IA, si aplica           │
│ Calcular ICE con IA   Guardar       │
└────────────────────────────────────┘

Dialog: PriorityModal
┌────────────────────────────────────┐
│ Sugerencia ICE de IA                │
│ Chips: Impacto, Confianza, Facilidad│
│ ICE total destacado                 │
│ Alert: reasoning breve              │
│ Ajustar valores   Confirmar tarea   │
└────────────────────────────────────┘
```

## Responsabilidades por componente

### `App`

- Renderiza la estructura principal.
- Mantiene que dialogo esta abierto: crear, editar, confirmar IA o eliminar.
- Consume `useTasks` para leer tareas, filtro y acciones.
- No llama directamente a `localStorage` ni a la API de IA.

### `AppHeader`

- Usa `AppBar` y `Toolbar`.
- Muestra `Gestor de Tareas ICE`.
- Incluye el boton `Nueva tarea`.
- Puede mostrar un contador simple de tareas pendientes si no complica el MVP.

### `TaskFilters`

- Recibe `filter` y `onFilterChange`.
- Valores permitidos: `all`, `pending`, `completed`.
- No ordena ni modifica tareas.

### `TaskTable`

- Recibe tareas ya filtradas y ordenadas, o aplica una funcion derivada clara.
- Orden por defecto: `iceScore` descendente.
- Muestra estado vacio mediante `EmptyState` cuando no hay tareas.
- Evita mutar el array original.

### `TaskTableRow`

- Muestra titulo y descripcion resumida.
- Usa `IceScoreChips` para I, C, E e ICE.
- Permite completar/reabrir, editar y eliminar.
- Puede incluir accion `Recalcular ICE con IA` si se quiere repetir la sugerencia desde una tarea existente.

### `TaskFormDialog`

- Contiene el estado local del formulario.
- Valida titulo obligatorio.
- Valida descripcion obligatoria antes de calcular ICE con IA.
- Valida que Impacto, Confianza y Facilidad sean enteros entre 1 y 10.
- Permite entrada manual aunque falle la IA.
- Al guardar, envia datos ya validados al contexto.

### `PriorityModal`

- Presenta la sugerencia de IA como decision asistida, no automatica.
- Muestra `impact`, `confidence`, `ease`, `iceScore` y `reasoning`.
- Permite confirmar o volver a ajustar valores.
- Muestra `CircularProgress` mientras la IA calcula.
- Muestra `Alert` si la API falla o devuelve JSON invalido.

### `IceScoreChips`

- Centraliza la visualizacion de scores.
- Usa colores consistentes:
  - 1 a 4: bajo, `error`.
  - 5 a 7: medio, `warning`.
  - 8 a 10: alto, `success`.
- El ICE total se muestra con un decimal.

## Estados UX obligatorios

| Estado | Componente responsable | Comportamiento esperado |
| --- | --- | --- |
| Lista vacia | `TaskTable` / `EmptyState` | Mostrar mensaje y CTA `Nueva tarea` |
| Calculando IA | `TaskFormDialog` / `PriorityModal` | Mostrar `CircularProgress` y evitar doble envio |
| Error API IA | `TaskFormDialog` / `PriorityModal` | Mostrar `Alert` y permitir valores manuales |
| Respuesta IA invalida | `TaskFormDialog` / `PriorityModal` | Rechazar valores fuera de 1-10 y explicar el problema |
| Guardado correcto | `SnackbarFeedback` | Confirmar creacion o actualizacion |
| Eliminacion | `ConfirmDialog` + `SnackbarFeedback` | Confirmar antes de borrar y mostrar feedback |

## Modelo visual de prioridad

| Rango | Significado | Color MUI |
| --- | --- | --- |
| 1.0 - 4.0 | Baja prioridad | `error` |
| 4.1 - 7.0 | Prioridad media | `warning` |
| 7.1 - 10.0 | Alta prioridad | `success` |

## Fuera de alcance para este MVP

- Rutas con React Router.
- Pantallas de Analytics.
- Pantallas de Settings.
- Autenticacion.
- Backend.
- Paginacion.
- Multiusuario.
- Tags, subtareas, adjuntos o fechas limite.

Estas piezas pueden aparecer en una evolucion futura, pero no deben condicionar la estructura inicial del MVP.
