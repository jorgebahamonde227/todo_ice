# Ice Task Manager — Agent Instructions

SPA de gestión de tareas con modelo ICE (Impact × Confidence × Ease), construida con React 18 + TypeScript strict + Vite + Material UI v6.  
Sin backend, sin router, sin autenticación. Persistencia en `localStorage`.
Use always english

## Documentación de referencia
- Alcance funcional y criterios de completitud: [docs/MVP_SCOPE.md](../docs/MVP_SCOPE.md)
- Convenciones técnicas completas (stack, estructura, tipos, reglas): [docs/DEV_GUIDE.md](../docs/DEV_GUIDE.md)

## Comandos clave
```bash
npm run dev      # servidor de desarrollo
npm run build    # compila TypeScript + bundle Vite
npm run lint     # ESLint — debe pasar sin errores antes de cada commit
```
Requiere `VITE_AI_API_KEY` en `.env` (copiar de `.env.example`). Sin ella, "Sugerir ICE con IA" se deshabilita; el resto funciona con normalidad.

## Estructura de carpetas (`ice-task-manager/src/`)
```
components/   # Presentación pura — sin lógica de negocio, sin hooks de estado global
hooks/        # useTasks (fuente de verdad), useIceSuggestion
services/     # aiService.ts — llamadas a Google Gemini
types/        # task.ts — única definición de tipos del dominio
utils/        # ice.ts — funciones puras (calculateIceScore, validación de rangos)
```

## Reglas de arquitectura
- `useTasks` es la única fuente de verdad del array de tareas.
- `App.tsx` solo compone: obtiene estado de `useTasks` y pasa props; **sin lógica condicional de negocio**.
- Dependencia unidireccional: `components/` nunca importa desde `hooks/` o `services/`.
- No usar Context API, Redux, Zustand ni Jotai.
- No añadir librerías sin justificación explícita y comparación con alternativa nativa.

> Convenciones de código TypeScript, MUI, nombrado y checklist: [.github/instructions/react-typescript.instructions.md](./instructions/react-typescript.instructions.md)
