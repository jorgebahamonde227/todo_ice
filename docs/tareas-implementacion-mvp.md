# Tareas de implementación del MVP

Documento de trabajo con una secuencia recomendada de implementación para el proyecto **Ice Task Manager**, alineada con [MVP_SCOPE.md](../MVP_SCOPE.md) y [DEV_GUIDE.md](../DEV_GUIDE.md).

## 1. Inicializar la base del proyecto y la estructura técnica

**Objetivo:** dejar operativa la aplicación React + TypeScript + Vite con la estructura de carpetas, tema MUI y configuración mínima de desarrollo.

**Incluye:**
- Crear el proyecto `ice-task-manager/` con Vite + React + TypeScript.
- Instalar y configurar Material UI e iconos.
- Crear la estructura `components/`, `hooks/`, `services/`, `types/`, `utils/`.
- Configurar `theme.ts`, `main.tsx`, `.env.example` y base de ESLint/Prettier.

**Resultado esperado:**
- `npm run dev` arranca sin errores.
- La app muestra un layout base con tema MUI aplicado.

## 2. Definir tipos de dominio y utilidades ICE

**Objetivo:** establecer el modelo de datos y la lógica pura que soportará el resto de la app.

**Incluye:**
- Crear `src/types/task.ts` con `TaskStatus`, `IceScore`, `Task` e `IceSuggestion`.
- Crear `src/utils/ice.ts` con funciones puras para:
  - calcular el `iceScore`
  - validar rangos de `impact`, `confidence` y `ease`
- Preparar constantes reutilizables como mínimos y máximos del modelo ICE.

**Resultado esperado:**
- Existe una única definición tipada del dominio.
- El cálculo del score ICE queda centralizado y reutilizable.

## 3. Implementar el hook `useTasks` con CRUD y persistencia local

**Objetivo:** construir la fuente de verdad de tareas y sincronizarla con `localStorage`.

**Incluye:**
- Crear `useTasks.ts` para gestionar el array de tareas.
- Implementar acciones para crear, editar, eliminar y cambiar estado.
- Calcular `iceScore` automáticamente al alta y edición.
- Cargar tareas iniciales desde `localStorage` y persistir cambios.
- Exponer datos derivados necesarios para filtro y ordenación.

**Resultado esperado:**
- La lógica de negocio principal vive fuera de `App.tsx`.
- Las tareas sobreviven al recargar la página.

## 4. Construir la UI base de listado, filtros y ordenación

**Objetivo:** mostrar tareas y permitir navegación funcional dentro de la única vista del MVP.

**Incluye:**
- Crear `FilterBar`, `TaskList`, `TaskCard` e `IceScoreBadge`.
- Implementar filtrado por estado: todas, pendientes, en progreso y completadas.
- Implementar ordenación por score ICE y fecha de creación.
- Permitir cambio de estado directamente desde cada tarjeta.

**Resultado esperado:**
- La lista refleja correctamente filtros y orden activo.
- El score ICE se visualiza de forma clara y consistente con el tema.

## 5. Implementar el formulario de creación y edición de tareas

**Objetivo:** permitir crear y editar tareas desde una interfaz controlada y consistente.

**Incluye:**
- Crear `TaskForm` dentro de un modal o `Dialog` de MUI.
- Añadir campos de título, descripción, `impact`, `confidence` y `ease`.
- Mostrar el `iceScore` en tiempo real mientras el usuario edita.
- Integrar validaciones básicas de campos requeridos y rangos válidos.
- Conectar `onSave` y `onCancel` con el estado gestionado por `useTasks`.

**Resultado esperado:**
- Se pueden crear y editar tareas sin lógica duplicada.
- El score cambia en tiempo real según los valores ICE.

## 6. Integrar la sugerencia ICE con IA

**Objetivo:** añadir la capacidad de obtener una propuesta automática de valores ICE a partir de la descripción.

**Incluye:**
- Crear `services/aiService.ts` para llamar al proveedor elegido.
- Crear `hooks/useIceSuggestion.ts` para encapsular `loading`, `error` y respuesta.
- Añadir el botón **Sugerir ICE con IA** en `TaskForm`.
- Parsear y validar la respuesta con `impact`, `confidence`, `ease` y `reasoning`.
- Deshabilitar la acción cuando falte `VITE_AI_API_KEY` o la descripción no sea suficiente.

**Resultado esperado:**
- La app puede recuperar sugerencias válidas desde la API.
- Si la IA falla, el usuario sigue pudiendo completar el formulario manualmente.

## 7. Integrar la composición final en `App.tsx` y cerrar calidad

**Objetivo:** unir todas las piezas, verificar el flujo end-to-end y dejar el MVP listo para entrega.

**Incluye:**
- Conectar `App.tsx` con `useTasks` y los componentes presentacionales.
- Revisar estados vacíos, mensajes de error y comportamiento del modal.
- Validar los criterios del MVP: CRUD, persistencia, filtros, ordenación y cálculo ICE.
- Ejecutar `npm run lint` y `npm run build`.
- Ajustar detalles de UX mínimos para evitar fricción en la demo final.

**Resultado esperado:**
- El flujo completo funciona de inicio a fin.
- El proyecto queda estable, compilable y listo para demostración.

## Orden recomendado de ejecución

1. Base del proyecto.
2. Tipos y utilidades.
3. Hook de tareas con persistencia.
4. Listado, filtros y ordenación.
5. Formulario crear/editar.
6. Integración de IA.
7. Ensamblado final y validación.