# Guia de control para desarrollo ReactJS

## Proposito

Este documento define las normas de desarrollo para el proyecto **Smart Task Manager (ICE)**. Su objetivo es mantener un codigo React limpio, simple y facil de seguir durante el curso, evitando ampliar el numero de librerias salvo que aporten valor claro.

Las decisiones tecnicas de este documento complementan el alcance funcional definido en `MVP_ALCANCE_FUNCIONAL.md`.

## Stack aprobado

- React 18+
- TypeScript con `strict` activado
- Material UI como libreria principal de controles y layout
- React Context + `useReducer` para estado global de tareas
- `fetch` nativo para llamadas HTTP
- `localStorage` para persistencia en navegador
- Vite como herramienta de desarrollo si se crea el proyecto desde cero

No usar Tailwind, Bootstrap, Zustand, Redux, Axios ni librerias de formularios para este MVP salvo decision justificada. La prioridad es aprender React y mantener una base pequena.

## Principios de clean code

- Cada componente debe tener una responsabilidad principal.
- Preferir nombres descriptivos: `TaskForm`, `TaskTable`, `calculateIceScore`, `saveTasks`.
- Evitar componentes grandes. Si un componente supera unas 150 lineas o mezcla UI, estado y logica de negocio, dividirlo.
- No duplicar calculos. El score ICE debe calcularse siempre con una funcion comun.
- Evitar valores magicos. Usar constantes para claves de `localStorage`, rangos de score y opciones de filtro.
- Mantener los efectos (`useEffect`) pequenos y orientados a sincronizacion, no a logica compleja.
- No comentar lo obvio. Comentar solo reglas de negocio o decisiones que no sean evidentes.
- Tipar datos de dominio con interfaces o types compartidos.
- Tratar los errores como estados esperados de la UI, no solo como `console.error`.

## Estructura de carpetas

```text
src/
├── app/
│   ├── App.tsx
│   └── theme.ts
├── components/
│   ├── layout/
│   │   └── AppHeader.tsx
│   └── tasks/
│       ├── IceScoreChips.tsx
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

Reglas:

- `components/` contiene solo componentes de UI.
- `context/` contiene el estado global y acciones del dominio.
- `hooks/` expone accesos simples a estado o comportamiento reutilizable.
- `services/` contiene integraciones externas: IA y almacenamiento.
- `types/` contiene modelos compartidos.
- `utils/` contiene funciones puras sin dependencia de React.

## Division en componentes

### `App.tsx`

Responsable de componer la pantalla principal:

- Tema de Material UI.
- Provider de tareas.
- Cabecera.
- Acciones principales.
- Tabla/listado de tareas.
- Dialogo de crear/editar.

No debe contener llamadas directas a `localStorage` ni a la API de IA.

### `AppHeader.tsx`

Cabecera visual de la aplicacion usando `AppBar`, `Toolbar` y `Typography` de MUI.

### `TaskFilters.tsx`

Controla el filtro visible:

- Todas.
- Pendientes.
- Completadas.

Usar `ToggleButtonGroup`, `Tabs` o `ButtonGroup` de MUI. El componente recibe el filtro actual y un callback.

### `TaskTable.tsx`

Lista tareas usando componentes de tabla de MUI:

- `Table`
- `TableHead`
- `TableBody`
- `TableRow`
- `TableCell`

Debe recibir tareas ya filtradas y ordenadas cuando sea posible. Si ordena internamente, la regla debe estar clara y centralizada.

### `TaskTableRow.tsx`

Representa una tarea individual:

- Titulo.
- Descripcion resumida.
- Scores ICE.
- Estado completado.
- Acciones de editar, eliminar y recalcular.

Usar `IconButton` con iconos de MUI o `@mui/icons-material` si ya esta instalado. Si no esta instalado, usar botones de texto de MUI para no agregar dependencias innecesarias.

### `TaskFormDialog.tsx`

Formulario para crear o editar tareas usando:

- `Dialog`
- `TextField`
- `Button`
- `Stack`
- `Alert`
- `CircularProgress`

Reglas:

- Validar titulo obligatorio.
- Validar descripcion obligatoria antes de calcular ICE.
- Permitir entrada manual de scores si falla la IA.
- Mantener el estado local del formulario dentro del componente.
- Enviar al contexto solo datos ya validados.

### `IceScoreChips.tsx`

Componente pequeno para visualizar:

- Impact.
- Confidence.
- Ease.
- ICE total.

Usar `Chip`, `Tooltip` o `Box` de MUI. El color puede depender del rango:

- 1 a 4: bajo.
- 5 a 7: medio.
- 8 a 10: alto.

## Gestion del estado

Para este MVP se usara **React Context + `useReducer`**. Es suficiente para una aplicacion pequena, evita anadir librerias y permite practicar patrones basicos de React.

### Estado global

```ts
export type TaskFilter = 'all' | 'pending' | 'completed';

export interface TaskState {
  tasks: Task[];
  filter: TaskFilter;
  loadingTaskId?: string;
  error?: string;
}
```

### Acciones recomendadas

```ts
type TaskAction =
  | { type: 'tasks/loaded'; payload: Task[] }
  | { type: 'task/created'; payload: Task }
  | { type: 'task/updated'; payload: Task }
  | { type: 'task/deleted'; payload: string }
  | { type: 'task/toggled'; payload: string }
  | { type: 'filter/changed'; payload: TaskFilter }
  | { type: 'error/set'; payload: string }
  | { type: 'error/cleared' };
```

### Reglas de estado

- El reducer debe ser puro: sin llamadas a API, sin `localStorage`, sin fechas generadas dentro.
- Las llamadas a IA y almacenamiento se hacen en funciones del provider o en servicios.
- Guardar en `localStorage` despues de crear, editar, borrar o completar tareas.
- Ordenar por ICE descendente en una funcion derivada, no mutando el array original.
- No guardar datos duplicados si se pueden calcular facilmente, salvo `iceScore`, que se conserva por simplicidad educativa.

## Modelo de datos

```ts
export interface Task {
  id: string;
  title: string;
  description: string;
  impact: number;
  confidence: number;
  ease: number;
  iceScore: number;
  completed: boolean;
  createdAt: string;
  updatedAt: string;
}
```

Fechas como `string` ISO para simplificar la persistencia en `localStorage`.

## Servicios

### `aiService.ts`

Responsabilidades:

- Enviar la descripcion de la tarea a la API externa.
- Solicitar una respuesta JSON.
- Validar que `impact`, `confidence` y `ease` son numeros entre 1 y 10.
- Devolver un objeto tipado.
- Lanzar un error controlado si la respuesta no es valida.

No debe importar componentes React.

### `storageService.ts`

Responsabilidades:

- Leer tareas desde `localStorage`.
- Guardar tareas en `localStorage`.
- Recuperarse de JSON invalido devolviendo una lista vacia.

Clave recomendada:

```ts
export const TASKS_STORAGE_KEY = 'smart-task-manager.tasks';
```

## Funciones puras

### `utils/ice.ts`

```ts
export function calculateIceScore(impact: number, confidence: number, ease: number): number {
  return Number(((impact + confidence + ease) / 3).toFixed(1));
}

export function isValidScore(value: number): boolean {
  return Number.isInteger(value) && value >= 1 && value <= 10;
}
```

Estas funciones deben usarse en formularios, servicios y pruebas manuales para evitar reglas duplicadas.

## Normas TypeScript

- No usar `any` salvo caso excepcional y documentado.
- Preferir `unknown` para datos externos antes de validar.
- Tipar props con `interface`.
- Usar union types para filtros, estados y acciones.
- Evitar enums si un union type de strings es suficiente.
- Usar nombres de archivo en minusculas para tipos y servicios: `task.ts`, `aiService.ts`.
- Usar nombres PascalCase para componentes: `TaskTable.tsx`.

## Normas React

- Componentes funcionales.
- Hooks solo en componentes o hooks personalizados.
- Props claras y pequenas.
- Evitar pasar objetos gigantes si el componente solo necesita dos campos.
- Usar `useMemo` solo cuando haya calculos derivados claros como filtrado y ordenacion de tareas.
- Usar `useCallback` solo si ayuda a evitar renders innecesarios en componentes memorizados.
- No optimizar antes de tener un problema visible.

## Normas Material UI

- Usar `ThemeProvider` y un `theme.ts` centralizado.
- Priorizar componentes MUI antes de crear controles desde cero.
- Usar `sx` para ajustes locales pequenos.
- Si un estilo se repite varias veces, crear componente o constante de estilo.
- Evitar mezclar MUI con frameworks CSS externos.
- Usar `Container`, `Stack`, `Box` y `Paper` para layout simple.
- Usar `Dialog` para crear/editar tareas.
- Usar `Snackbar` o `Alert` para feedback de errores.
- Mantener contraste suficiente y diseno responsive basico.

## Flujo recomendado de implementacion

1. Crear el proyecto React con TypeScript.
2. Instalar Material UI.
3. Definir tipos en `types/task.ts`.
4. Crear utilidades ICE en `utils/ice.ts`.
5. Crear `storageService.ts`.
6. Crear `TaskContext.tsx` con reducer.
7. Construir la UI con componentes MUI.
8. Integrar `aiService.ts`.
9. Anadir manejo de errores y entrada manual de scores.
10. Revisar criterios funcionales del MVP.

## Criterios de revision tecnica

- El proyecto compila sin errores TypeScript.
- No hay componentes con responsabilidades mezcladas.
- No hay reglas ICE duplicadas.
- La UI usa Material UI de forma consistente.
- El estado global esta en Context y no disperso entre componentes no relacionados.
- Los servicios no importan React.
- `localStorage` esta encapsulado.
- La aplicacion mantiene comportamiento correcto sin conexion a la API, permitiendo scores manuales.

## Dependencias sugeridas

```json
{
  "react": "^18.2.0",
  "react-dom": "^18.2.0",
  "typescript": "^5.0.0",
  "@mui/material": "^5.15.0",
  "@emotion/react": "^11.11.0",
  "@emotion/styled": "^11.11.0"
}
```

Dependencias opcionales:

- `@mui/icons-material`: solo si se quieren iconos MUI.
- `uuid`: solo si se prefiere no usar `crypto.randomUUID()`.

## Decision de simplicidad

Para este proyecto se prioriza:

- Menos librerias.
- Mas TypeScript claro.
- Componentes pequenos.
- Servicios aislados.
- Material UI como unico sistema visual.
- Estado global nativo con React.

Esto mantiene el foco en aprender React sin convertir el MVP en una arquitectura innecesariamente grande.
