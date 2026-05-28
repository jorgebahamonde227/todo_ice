---
applyTo: "src/**/*.{ts,tsx}"
description: "Use when creating or editing React components, hooks, services or utilities. Covers TypeScript strict rules, MUI conventions, component design, state management, naming and import order."
---

# React + TypeScript — Convenciones de código

## TypeScript

- Modo estricto. **Prohibido `any`**; usar `unknown` + type guard.
- Props con `type`; `interface` solo para herencia con `extends`.
- Tipos en `src/types/`; no duplicar shapes en los componentes.
- Prohibido `as unknown as X` salvo en `services/` con JSON externo (documentar con comentario).
- Retorno explícito cuando el tipo no es obvio por inferencia.

### Tipos del dominio (`src/types/task.ts`)

```typescript
type TaskStatus = 'pending' | 'in-progress' | 'completed';

interface IceScore {
  impact: number;      // 1–10
  confidence: number;  // 1–10
  ease: number;        // 1–10
}

interface Task {
  id: string;
  title: string;
  description: string;
  ice: IceScore;
  iceScore: number;    // impact × confidence × ease — calculado, nunca mutado a mano
  status: TaskStatus;
  createdAt: string;   // ISO 8601
}

interface IceSuggestion {
  impact: number;
  confidence: number;
  ease: number;
  reasoning: string;
}
```

Extender desde aquí; no crear shapes locales en los componentes.

## Componentes

- Un componente = una responsabilidad. Si hace más de una cosa, dividir.
- Presentacionales puros: reciben datos y callbacks vía props; no llaman hooks de estado global.
- Tamaño orientativo: < 80 líneas de JSX. Si se supera, extraer sub-componente.
- Cada componente en su propia carpeta: `TaskCard/TaskCard.tsx`.

| Componente | Responsabilidad | Props clave |
|---|---|---|
| `App` | Componer layout + pasar estado desde `useTasks` | — |
| `FilterBar` | Controles de filtro y orden; emitir eventos al padre | `filter`, `sort`, `onChange` |
| `TaskList` | Renderizar lista de `TaskCard` | `tasks`, `onEdit`, `onDelete` |
| `TaskCard` | Mostrar datos de una tarea; cambiar estado inline | `task`, `onEdit`, `onDelete`, `onStatusChange` |
| `IceScoreBadge` | Badge visual del score ICE (color semáforo) | `score` |
| `TaskForm` | Formulario controlado crear/editar + botón IA | `task?`, `onSave`, `onCancel` |

## Funciones

- Máximo **un nivel de abstracción** por función.
- Funciones en `utils/` deben ser **puras**: sin efectos secundarios, sin acceso a estado externo.
- Los comentarios explican el *porqué*, nunca el *qué*; el nombre debe ser suficiente.

## Estado

- Datos derivados con `useMemo`; nunca `useEffect` + `setState` en cascada.
- Estado local para UI efímera (modal abierto/cerrado, input no guardado); no sube a `useTasks`.
- `useReducer` si `useTasks` supera 4 acciones independientes; `useState` para casos simples.

## Material UI

- Usar siempre componentes MUI para controles (Button, TextField, Select, Chip, Dialog…). No HTML nativo equivalente.
- Estilar con prop `sx`; sin `style` inline ni CSS externo salvo excepción documentada.
- Layouts con `Box`, `Stack`, `Grid2`; no añadir librerías de layout adicionales.
- Colores de `IceScoreBadge` desde `theme.palette`, no strings hexadecimales literales.

| Rango ICE Score | Color de `theme.palette` |
|---|---|
| 1–200 | `error.main` |
| 201–500 | `warning.main` |
| 501–1000 | `success.main` |

## Nombrado

| Artefacto | Convención | Ejemplo |
|---|---|---|
| Componente / Tipo | `PascalCase` | `TaskCard` |
| Hook | `use` + PascalCase | `useTasks` |
| Función / variable | `camelCase` | `calculateIceScore` |
| Constante global | `SCREAMING_SNAKE_CASE` | `MAX_ICE_VALUE` |
| Archivo | igual que su export | `TaskCard.tsx` |

Textos de UI en español; claves de código en inglés.

## Orden de imports

Librerías externas → MUI → hooks propios → componentes propios → tipos → utils.

## Prohibiciones

- No `console.log` en código de producción.
- No mutar estado directamente; usar siempre inmutabilidad.
- No hardcodear API keys; usar `import.meta.env.VITE_AI_API_KEY`.
- No duplicar lógica entre componentes; extraer a `utils/` o `hooks/`.
- Sin comentarios que expliquen *qué* hace el código; el nombre debe ser suficiente.

## Checklist antes de marcar completado

**Componente nuevo:** una responsabilidad · props con `type` · sin lógica de negocio · usa MUI · `sx` para estilos · sin `console.log` · sin imports no usados.

**Hook nuevo:** nombre con `use` · lógica de negocio completa · derivados con `useMemo` · sin dependencias en componentes concretos.

**Antes de cada commit:** `npm run lint` sin errores · `npm run build` sin errores de TypeScript.
