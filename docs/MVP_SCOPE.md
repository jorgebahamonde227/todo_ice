# MVP — Gestor de Tareas Inteligente con Modelo ICE

> Alcance funcional del producto. Para estructura de carpetas, tipos, convenciones de código y Material UI consultar [DEV_GUIDE.md](./DEV_GUIDE.md).

## Descripción general

Aplicación web de página única (SPA) construida con **React + TypeScript + Vite** que permite crear, gestionar y priorizar tareas usando el modelo **ICE** (Impact, Confidence, Ease). El cálculo del score ICE se sugiere automáticamente mediante una **API de IA gratuita** a partir de la descripción de la tarea. Diseñada como proyecto de curso: sin backend, sin autenticación, sin persistencia real.

---

## Modelo ICE

| Dimensión      | Descripción                                              | Rango |
|----------------|----------------------------------------------------------|-------|
| **Impact**     | Impacto esperado si la tarea se completa                 | 1–10  |
| **Confidence** | Confianza en que producirá ese impacto                   | 1–10  |
| **Ease**       | Facilidad de implementación                              | 1–10  |

**ICE Score** = `Impact × Confidence × Ease`

---

## Funcionalidades incluidas en el MVP

### 1. Gestión de tareas (CRUD básico)
- Crear una tarea con: **título**, **descripción** y valores ICE (editables manualmente).
- Editar cualquier campo de una tarea existente.
- Eliminar una tarea.
- El ICE Score se calcula automáticamente al cambiar cualquier dimensión.

### 2. Sugerencia de ICE mediante IA
- Botón **"Sugerir ICE con IA"** disponible al crear o editar una tarea.
- Envía la descripción de la tarea a una **API de IA gratuita** (p. ej. Google Gemini API free tier u OpenRouter con modelo gratuito).
- La IA devuelve valores sugeridos para Impact, Confidence y Ease con una breve justificación.
- El usuario puede aceptar la sugerencia o ajustarla manualmente antes de guardar.
- La API key se configura mediante variable de entorno (`VITE_AI_API_KEY`).

### 3. Listado y ordenación
- Listado de todas las tareas visibles en pantalla.
- Ordenación por **ICE Score** (mayor a menor) con un solo clic.
- Ordenación por **fecha de creación** (más reciente primero).

### 4. Filtrado de estado
- Filtrar tareas por estado: **Todas · Pendientes · En progreso · Completadas**.
- Cambiar el estado de una tarea directamente desde la tarjeta.

### 5. Persistencia en localStorage
- Las tareas se guardan en `localStorage` al crearse, editarse o eliminarse.
- Se recuperan automáticamente al recargar la página.
- *(Sustituye la persistencia real; suficiente para el contexto del curso.)*

---

## Funcionalidades excluidas del MVP

| Característica          | Motivo de exclusión                        |
|-------------------------|--------------------------------------------|
| Backend / API propia    | Fuera del alcance del curso de React       |
| Autenticación           | Añade complejidad no pedagógica            |
| Base de datos real      | Sustituida por localStorage                |
| Paginación              | Innecesaria para el volumen de un curso    |
| Multiusuario            | Requiere backend                           |
| Tags / etiquetas        | Aumenta la complejidad de UI/estado        |

---

## Pantallas / vistas

| Vista                | Descripción                                                           |
|----------------------|-----------------------------------------------------------------------|
| **Lista de tareas**  | Pantalla principal con todas las tareas, filtros y botón de ordenar   |
| **Modal crear/editar** | Formulario con campos ICE, botón de sugerencia IA y preview del score |

> La aplicación es de **una sola ruta**; no requiere React Router.

---

## API de IA — integración mínima

- **Proveedor sugerido:** Google Gemini (`gemini-2.0-flash`) — tier gratuito.
- **Prompt:** Se envía la descripción de la tarea y se solicita un JSON con `impact`, `confidence`, `ease` y `reasoning`.
- **Manejo de errores:** Si la API falla o no hay key configurada, el botón se deshabilita y muestra un mensaje; el usuario puede ingresar los valores manualmente.

---

## Criterios de completitud del MVP

- [ ] CRUD completo de tareas funciona sin errores.
- [ ] ICE Score se calcula y muestra en tiempo real.
- [ ] La sugerencia IA devuelve valores válidos al menos con una API key real.
- [ ] Las tareas persisten al recargar la página.
- [ ] Filtrado por estado funciona correctamente.
- [ ] Ordenación por ICE Score funciona correctamente.
- [ ] El proyecto arranca con `npm run dev` sin configuración adicional (excepto la API key).
