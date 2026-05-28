---
applyTo:
  - "**/*.ts"
  - "**/*.tsx"
---

# Instrucciones para editar React + TypeScript

Estas instrucciones aplican a cambios en archivos React y TypeScript del proyecto Smart Task Manager (ICE). Usa como referencia completa `GUIA_CONTROL_DESARROLLO_REACT.md`, pero evita repetir reglas ya definidas en `copilot-instructions.md`.

## Stack y límites

- Usa React 18+ con TypeScript en modo `strict`.
- Usa Material UI como librería principal de UI.
- Usa React Context + `useReducer` para el estado global.
- Usa `fetch` nativo para HTTP.
- Usa `localStorage` solo a través de `storageService`.
- No introduzcas Tailwind, Bootstrap, Zustand, Redux, Axios, React Router ni librerías de formularios salvo decisión justificada.

## Organización por carpeta

- `src/app/`: composición principal, `ThemeProvider` y tema. No debe contener `localStorage` ni llamadas directas a APIs.
- `src/components/`: componentes de UI. No debe acceder directamente a APIs, `localStorage` ni `TaskContext`.
- `src/context/`: estado global, reducer y provider.
- `src/hooks/`: acceso reutilizable a estado o comportamiento, especialmente `useTasks()`.
- `src/services/`: integraciones externas como IA y almacenamiento. No debe importar React.
- `src/types/`: modelos compartidos. No debe contener lógica.
- `src/utils/`: funciones puras sin React ni side effects.

## Componentes React

- Mantén cada componente con una responsabilidad principal.
- Divide componentes que superen unas 150 líneas efectivas o mezclen UI, estado y lógica de negocio.
- Usa nombres descriptivos como `TaskFormDialog`, `TaskTable`, `TaskTableRow` o `IceScoreChips`.
- Define una interfaz de props al inicio del archivo.
- Usa componentes funcionales y hooks solo dentro de componentes o hooks personalizados.
- Usa `useTasks()` para acceder al estado de tareas; no uses `useContext(TaskContext)` directamente en componentes.
- No pases objetos grandes por props si el componente solo necesita algunos campos.
- Usa `useMemo` solo para cálculos derivados claros, como filtrado u ordenación de tareas.
- Usa `useCallback` solo cuando ayude a evitar renders innecesarios en componentes memorizados.

```typescript
interface TaskTableProps {
  tasks: Task[];
  onEdit: (task: Task) => void;
  onDelete: (taskId: string) => void;
}

export function TaskTable({ tasks, onEdit, onDelete }: TaskTableProps) {
  return (
    <Table>
      <TableBody>
        {tasks.map((task) => (
          <TaskTableRow
            key={task.id}
            task={task}
            onEdit={onEdit}
            onDelete={onDelete}
          />
        ))}
      </TableBody>
    </Table>
  );
}
```

## Estado, reducer y efectos

- El reducer debe ser puro: sin `fetch`, sin `localStorage`, sin generación de fechas y sin side effects.
- Genera `createdAt` y `updatedAt` en el provider o en una capa de acción, no dentro del reducer.
- Carga y guarda tareas desde el provider usando `storageService`.
- Mantén los `useEffect` pequeños y orientados a sincronización.
- Incluye todas las dependencias de cada `useEffect`.
- Ordena y filtra mediante funciones derivadas sin mutar el array original.

```typescript
useEffect(() => {
  saveTasks(state.tasks);
}, [state.tasks]);
```

## Modelo de datos

Mantén `Task` y `TaskFilter` como fuente de verdad en `src/types/task.ts`.

```typescript
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

export type TaskFilter = "all" | "pending" | "completed";
```

- `title` es obligatorio.
- `description` es obligatoria para calcular con IA.
- `impact`, `confidence` y `ease` son enteros entre 1 y 10.
- `createdAt` y `updatedAt` deben guardarse como cadenas ISO.

## Cálculo ICE

- Centraliza el cálculo en `src/utils/ice.ts`.
- No dupliques la fórmula en componentes, contexto ni servicios.
- Valida rangos con una función común.

```typescript
export function calculateIceScore(
  impact: number,
  confidence: number,
  ease: number,
): number {
  return Number(((impact + confidence + ease) / 3).toFixed(1));
}

export function isValidScore(value: number): boolean {
  return Number.isInteger(value) && value >= 1 && value <= 10;
}
```

## Servicios

### `aiService.ts`

- Envía la descripción de la tarea a la API externa.
- Solicita y procesa una respuesta JSON.
- Valida que `impact`, `confidence` y `ease` sean enteros entre 1 y 10.
- Devuelve un objeto tipado.
- Lanza errores controlados si la respuesta no es válida o falla la petición.
- No importa React y no persiste datos.

```typescript
export interface AIEstimate {
  impact: number;
  confidence: number;
  ease: number;
  reasoning: string;
}
```

### `storageService.ts`

- Encapsula todo acceso a `localStorage`.
- Lee tareas desde almacenamiento.
- Guarda tareas.
- Se recupera de JSON inválido devolviendo una lista vacía.
- Usa una constante para la clave de almacenamiento.

```typescript
export const TASKS_STORAGE_KEY = "smart-task-manager.tasks";
```

## TypeScript

- No uses `any` salvo caso excepcional y documentado.
- Usa `unknown` para datos externos antes de validarlos.
- Tipar entradas y salidas de funciones exportadas.
- Usa `interface` para props.
- Usa union types para filtros, estados y acciones.
- Evita `enum` si un union type de strings es suficiente.
- Usa nombres de archivo en minúsculas para tipos y servicios: `task.ts`, `aiService.ts`.
- Usa PascalCase para componentes: `TaskTable.tsx`.

## Material UI

- Usa `ThemeProvider` y un `theme.ts` centralizado.
- Prioriza componentes MUI antes de crear controles desde cero.
- Usa `sx` para ajustes locales pequeños.
- Si un estilo se repite, extrae componente, constante o helper.
- Evita mezclar MUI con frameworks CSS externos.
- Usa `Container`, `Stack`, `Box` y `Paper` para layouts simples.
- Usa `Dialog` para crear o editar tareas.
- Usa `Snackbar` o `Alert` para feedback de errores.
- Mantén contraste suficiente y diseño responsive básico.

## Checklist de edición

- TypeScript compila.
- No hay `any` innecesario.
- No hay cálculo ICE duplicado.
- No hay `localStorage` en componentes.
- No hay llamadas API directas desde componentes.
- El reducer sigue siendo puro.
- Los componentes acceden al contexto mediante `useTasks()`.
- Los arrays se ordenan o filtran sin mutar el original.
- La UI usa Material UI de forma consistente.
