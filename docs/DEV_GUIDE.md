# Guía de desarrollo — Ice Task Manager

> Documento de control técnico para el desarrollo del proyecto.  
> Para el alcance funcional consultar [MVP_SCOPE.md](./MVP_SCOPE.md).

---

## Stack tecnológico

| Capa          | Tecnología                         |
|---------------|------------------------------------|
| Framework     | React 18 + TypeScript (strict)     |
| Bundler       | Vite                               |
| UI Library    | Material UI v6 (`@mui/material`)   |
| Iconos        | `@mui/icons-material`              |
| Estado global | React hooks (`useState`, `useReducer`) |
| Persistencia  | `localStorage` via hook custom     |
| IA            | Google Gemini API (free tier)      |
| Linting       | ESLint + `eslint-plugin-react-hooks` |
| Formateo      | Prettier                           |

> **Regla de oro:** no añadir ni eliminar librerías sin justificación explícita. Cualquier nueva dependencia debe nombrarse, motivarse y compararse con una alternativa nativa antes de instalarse.

---

## Estructura de carpetas

```
ice-task-manager/
├── public/
├── src/
│   ├── components/             # Componentes UI (solo presentación)
│   │   ├── FilterBar/
│   │   │   └── FilterBar.tsx
│   │   ├── IceScoreBadge/
│   │   │   └── IceScoreBadge.tsx
│   │   ├── TaskCard/
│   │   │   └── TaskCard.tsx
│   │   ├── TaskForm/
│   │   │   └── TaskForm.tsx
│   │   └── TaskList/
│   │       └── TaskList.tsx
│   ├── hooks/                  # Lógica de negocio y estado
│   │   ├── useIceSuggestion.ts
│   │   └── useTasks.ts
│   ├── services/               # Acceso a APIs externas
│   │   └── aiService.ts
│   ├── types/                  # Tipos e interfaces compartidos
│   │   └── task.ts
│   ├── utils/                  # Funciones puras sin efectos
│   │   └── ice.ts
│   ├── App.tsx                 # Composición raíz; sin lógica de negocio
│   ├── main.tsx
│   └── theme.ts                # Configuración del tema MUI
├── .env.example
├── index.html
└── vite.config.ts
```

### Reglas de estructura

- Cada componente vive en su propia carpeta con el mismo nombre (`TaskCard/TaskCard.tsx`).  
- No crear sub-carpetas dentro de `hooks/`, `services/`, `utils/` salvo que superen los 5 archivos.  
- Nunca importar desde `components/` dentro de `hooks/` o `services/` (dependencia unidireccional).

---

## Tipos principales (`src/types/task.ts`)

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

> Extender desde aquí; no duplicar shapes en los componentes.

---

## División en componentes

| Componente       | Responsabilidad única                                              | Props clave                          |
|------------------|--------------------------------------------------------------------|--------------------------------------|
| `App`            | Componer layout + pasar estado desde `useTasks`                   | —                                    |
| `FilterBar`      | Renderizar controles de filtro y orden; emitir eventos al padre   | `filter`, `sort`, `onChange`         |
| `TaskList`       | Renderizar la lista mapeada de `TaskCard`                         | `tasks`, `onEdit`, `onDelete`        |
| `TaskCard`       | Mostrar datos de una tarea; cambiar estado inline                 | `task`, `onEdit`, `onDelete`, `onStatusChange` |
| `IceScoreBadge`  | Badge visual del score ICE (color semáforo por rango)             | `score`                              |
| `TaskForm`       | Formulario controlado crear/editar + botón sugerencia IA         | `task?`, `onSave`, `onCancel`        |

### Reglas de componentes

- **Un componente = una responsabilidad.** Si un componente hace más de una cosa, se divide.  
- **Componentes presentacionales puros:** no llaman hooks de estado global directamente; reciben datos y callbacks via props.  
- **`App.tsx` únicamente compone:** obtiene estado de `useTasks`, pasa props, no contiene lógica condicional de negocio.  
- Tamaño orientativo: < 80 líneas de JSX por componente. Si se supera, extraer sub-componente.

---

## Gestión del estado

```
useTasks (fuente de verdad)
  └── state: Task[]
  └── acciones: addTask · updateTask · deleteTask · setFilter · setSort
  └── efecto de sincronización → localStorage
  └── datos derivados: filteredTasks (useMemo)

useIceSuggestion
  └── state: loading · suggestion · error
  └── acción: fetchSuggestion(description)
  └── depende de: aiService.ts
```

### Reglas de estado

- **Una sola fuente de verdad:** `useTasks` es el único lugar donde vive el array de tareas.  
- **Datos derivados con `useMemo`**, nunca con `useEffect` + `setState` en cascada.  
- **Estado local para UI efímera** (modal abierto/cerrado, valor de input no guardado) — no sube a `useTasks`.  
- **`useReducer`** si `useTasks` supera 4 acciones independientes; `useState` es suficiente para casos más simples.  
- No usar Context API ni librerías de estado externas (Redux, Zustand, Jotai) en este MVP.

---

## Material UI — convenciones de uso

### Tema

Definir el tema centralizado en `src/theme.ts` y envolverlo en `main.tsx`:

```tsx
// theme.ts
import { createTheme } from '@mui/material/styles';

export const theme = createTheme({
  palette: {
    primary:   { main: '#1976d2' },
    secondary: { main: '#9c27b0' },
  },
  typography: {
    fontFamily: '"Roboto", "Helvetica", "Arial", sans-serif',
  },
});
```

### Reglas MUI

- Usar **siempre componentes MUI** para controles (Button, TextField, Select, Chip, Dialog…). No crear HTML nativo equivalente.  
- Estilar con la prop **`sx`**; no usar `style` inline ni CSS externo salvo casos excepcionales documentados.  
- Para layouts usar **`Box`**, **`Stack`** y **`Grid2`** de MUI; no añadir librerías de layout adicionales.  
- Los colores de `IceScoreBadge` deben venir de `theme.palette`, no de strings hexadecimales literales.  
- Los textos de la UI en español; las claves de código en inglés.

### Escala de color para ICE Score

| Rango de score | Color sugerido   |
|----------------|-----------------|
| 1 – 200        | `error.main`    |
| 201 – 500      | `warning.main`  |
| 501 – 1000     | `success.main`  |

---

## TypeScript — reglas de tipado

- **Modo estricto** activado en `tsconfig.json` (`"strict": true`).  
- Props tipadas con `type`; usar `interface` solo cuando se necesite herencia con `extends`.  
- **Prohibido `any`**; usar `unknown` + type guard si el tipo es indeterminado.  
- Prohibido `as unknown as X` salvo en la capa de `services/` con JSON externo, documentado con comentario.  
- Los tipos viven en `src/types/`; no duplicar shapes localmente en los componentes.  
- Funciones con retorno explícito cuando el tipo no es obvio por inferencia.

---

## Clean Code — reglas generales

### Nombres

| Artefacto         | Convención             | Ejemplo                  |
|-------------------|------------------------|--------------------------|
| Componente / Tipo | `PascalCase`           | `TaskCard`, `IceScore`   |
| Hook              | `use` + `PascalCase`   | `useTasks`               |
| Función / variable| `camelCase`            | `calculateIceScore`      |
| Constante global  | `SCREAMING_SNAKE_CASE` | `MAX_ICE_VALUE`          |
| Archivo           | igual que su export    | `TaskCard.tsx`           |

### Funciones

- Máximo **un nivel de abstracción** por función.  
- Sin efectos secundarios en funciones de `utils/`; deben ser funciones puras.  
- Sin comentarios que expliquen *qué* hace el código; el nombre debe ser suficiente. Los comentarios explican el *porqué*.

### Imports

- Orden: librerías externas → MUI → hooks propios → componentes propios → tipos → utils.  
- Sin imports no usados (ESLint lo aplica automáticamente).

### Prohibiciones

- `console.log` en código de producción.  
- Secretos o API keys hardcodeados; usar `.env` + `VITE_` prefix.  
- Lógica duplicada entre componentes; extraer a `utils/` o `hooks/`.  
- Mutación directa de estado (`task.title = '...'`); usar siempre inmutabilidad.

---

## Configuración ESLint + Prettier (mínima)

```jsonc
// .eslintrc (fragmento relevante)
{
  "extends": [
    "eslint:recommended",
    "plugin:react/recommended",
    "plugin:react-hooks/recommended",
    "plugin:@typescript-eslint/recommended"
  ],
  "rules": {
    "no-console": "warn",
    "@typescript-eslint/no-explicit-any": "error",
    "react/react-in-jsx-scope": "off"   // Vite no requiere import React
  }
}
```

```jsonc
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "es5",
  "printWidth": 100
}
```

---

## Variables de entorno

```bash
# .env.example
VITE_AI_API_KEY=   # Google Gemini API key (free tier)
```

- Nunca commitear `.env` con valores reales; commitear `.env.example` con claves vacías.  
- Acceder siempre via `import.meta.env.VITE_AI_API_KEY`.

---

## Lista de control de desarrollo

Usar esta checklist antes de marcar una tarea como completada:

### Por cada componente nuevo
- [ ] Tiene una sola responsabilidad.
- [ ] Props tipadas explícitamente con `type`.
- [ ] No contiene lógica de negocio; solo render.
- [ ] Usa componentes MUI; no HTML nativo para controles.
- [ ] Estilado con `sx`; sin `style` inline.
- [ ] Sin `console.log`.
- [ ] Sin imports no usados.

### Por cada hook nuevo
- [ ] Nombre empieza con `use`.
- [ ] Contiene toda la lógica de negocio relacionada.
- [ ] Datos derivados calculados con `useMemo`.
- [ ] Sin dependencias en componentes concretos.

### Antes de cada commit
- [ ] `npm run lint` pasa sin errores.
- [ ] `npm run build` compila sin errores de TypeScript.
- [ ] Los criterios de completitud del MVP afectados siguen siendo válidos.
