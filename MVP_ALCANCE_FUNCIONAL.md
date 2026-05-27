# Alcance funcional del MVP: Gestor de Tareas Inteligente con modelo ICE

## 1. Objetivo del MVP

Construir una aplicacion React sencilla para gestionar tareas y priorizarlas mediante el modelo ICE:

- **Impacto**: valor o beneficio esperado de completar la tarea.
- **Confianza**: seguridad sobre la estimacion y el resultado esperado.
- **Facilidad**: facilidad relativa para completar la tarea.

El MVP permitira crear tareas manualmente y, a partir de una descripcion en lenguaje natural, solicitar a una API de IA gratuita una estimacion inicial de Impacto, Confianza y Facilidad. La aplicacion calculara el puntaje ICE y ordenara las tareas por prioridad.

El proyecto esta pensado para un curso de React, por lo que debe priorizar claridad, componentes simples, estado local y una experiencia facil de entender.

## 2. Stack propuesto

- **Frontend**: React.
- **Estilos**: CSS simple, CSS Modules o framework ligero ya usado en el curso.
- **Estado**: `useState` y, si se quiere ordenar/filtrar con claridad, `useMemo`.
- **IA**: llamada HTTP desde el frontend a una API con capa gratuita.
- **Backend**: no incluido.
- **Persistencia**: no incluida. Los datos se pierden al recargar la pagina.

### API de IA recomendada para el curso

Para una demo simple, se puede usar **Gemini API de Google AI Studio**, ya que su documentacion oficial mantiene modelos con capa gratuita. Recomendacion practica:

- Modelo sugerido: un modelo rapido y economico/free tier, por ejemplo `gemini-2.5-flash` o el modelo gratuito disponible en Google AI Studio en el momento de la clase.
- La clave se configuraria en una variable de entorno de Vite, por ejemplo `VITE_GEMINI_API_KEY`.
- Como no hay backend, esta clave quedara expuesta en el navegador. Esto debe explicarse como una limitacion pedagogica del MVP, no como una practica segura para produccion.

## 3. Funcionalidades incluidas

### 3.1 Crear tarea

El usuario podra crear una tarea con:

- Titulo.
- Descripcion.
- Valores ICE manuales:
  - Impacto: 1 a 10.
  - Confianza: 1 a 10.
  - Facilidad: 1 a 10.

Al guardar, la aplicacion calculara automaticamente el puntaje ICE.

### 3.2 Calcular ICE con IA desde una descripcion

El usuario podra escribir una descripcion de la tarea y pulsar un boton, por ejemplo **"Calcular ICE con IA"**.

La aplicacion enviara la descripcion a la API de IA y solicitara una respuesta estructurada con:

- `impact`: numero de 1 a 10.
- `confidence`: numero de 1 a 10.
- `ease`: numero de 1 a 10.
- `reasoning`: explicacion breve para mostrar al usuario.

La respuesta de IA rellenara automaticamente los campos ICE del formulario. El usuario podra modificar los valores antes de guardar.

### 3.3 Calculo del puntaje ICE

Para mantener el MVP didactico, el puntaje se calculara como promedio:

```text
ICE = (Impacto + Confianza + Facilidad) / 3
```

El resultado se mostrara con un decimal.

Ejemplo:

```text
Impacto = 8
Confianza = 7
Facilidad = 6
ICE = 7.0
```

> Nota: en contextos de producto tambien se usa la formula `Impacto * Confianza * Facilidad`. Para el curso se recomienda el promedio porque es mas facil de explicar, visualizar y validar.

### 3.4 Listar tareas

La aplicacion mostrara una lista de tareas con:

- Titulo.
- Descripcion corta.
- Impacto.
- Confianza.
- Facilidad.
- Puntaje ICE.
- Estado: pendiente o completada.

### 3.5 Ordenar tareas por prioridad

La lista se ordenara por defecto de mayor a menor puntaje ICE.

Esto permite que las tareas mas prometedoras aparezcan arriba.

### 3.6 Marcar tarea como completada

El usuario podra marcar una tarea como completada o volverla a dejar como pendiente.

### 3.7 Eliminar tarea

El usuario podra eliminar una tarea de la lista.

### 3.8 Editar tarea

El usuario podra editar:

- Titulo.
- Descripcion.
- Impacto.
- Confianza.
- Facilidad.
- Estado.

Al modificar cualquier valor ICE, el puntaje se recalculara.

### 3.9 Estados de interfaz

La aplicacion debera contemplar estados basicos:

- Lista vacia.
- Cargando calculo de IA.
- Error al llamar a la API de IA.
- Respuesta invalida de la IA.

## 4. Funcionalidades excluidas

El MVP no incluye:

- Backend.
- Autenticacion.
- Persistencia real.
- Paginacion.
- Multiusuario.
- Tags o etiquetas.
- Roles o permisos.
- Subtareas.
- Fechas limite.
- Notificaciones.
- Adjuntos.
- Busqueda avanzada.
- Analitica o reportes.

## 5. Pantallas o vistas del MVP

### 5.1 Vista principal

Una unica pantalla con:

- Encabezado simple con nombre de la aplicacion.
- Formulario de creacion/edicion.
- Boton para calcular ICE con IA.
- Lista de tareas ordenada por ICE.

No se requieren rutas ni navegacion compleja.

### 5.2 Formulario de tarea

Campos:

- Titulo.
- Descripcion.
- Impacto.
- Confianza.
- Facilidad.
- Boton "Calcular ICE con IA".
- Boton "Guardar tarea".
- Boton "Cancelar edicion", solo si se esta editando.

### 5.3 Lista de tareas

Cada tarea puede representarse como una tarjeta o fila con:

- Titulo.
- Descripcion.
- Puntaje ICE destacado.
- Valores I, C y E.
- Boton completar/pendiente.
- Boton editar.
- Boton eliminar.

## 6. Modelo de datos propuesto

```js
{
  id: "uuid-o-timestamp",
  title: "Preparar presentacion del curso",
  description: "Crear slides y demo para explicar useState y componentes",
  impact: 8,
  confidence: 7,
  ease: 6,
  iceScore: 7.0,
  completed: false,
  createdAt: "2026-05-27T10:00:00.000Z"
}
```

## 7. Prompt sugerido para la IA

```text
Analiza la siguiente tarea y estima sus valores ICE.

Tarea:
"{{description}}"

Devuelve solo JSON valido con esta estructura:
{
  "impact": number,
  "confidence": number,
  "ease": number,
  "reasoning": "explicacion breve"
}

Reglas:
- impact, confidence y ease deben ser numeros enteros entre 1 y 10.
- No incluyas markdown.
- No incluyas texto fuera del JSON.
- Usa criterios sencillos para un gestor de tareas personal.
```

## 8. Reglas de validacion

- El titulo es obligatorio.
- La descripcion es obligatoria para usar el calculo con IA.
- Impacto, Confianza y Facilidad deben estar entre 1 y 10.
- Si la IA devuelve valores fuera de rango, se deben ajustar o rechazar con un mensaje de error.
- Si falla la API, el usuario debe poder introducir los valores manualmente.

## 9. Criterios de aceptacion

El MVP se considera completo cuando:

- El usuario puede crear una tarea.
- El usuario puede calcular valores ICE desde una descripcion usando IA.
- El usuario puede ajustar manualmente los valores ICE.
- La aplicacion calcula y muestra el puntaje ICE.
- Las tareas se muestran ordenadas por mayor puntaje ICE.
- El usuario puede editar, completar y eliminar tareas.
- La aplicacion muestra estados de carga y error para la IA.
- No existe backend, autenticacion, persistencia real, paginacion, multiusuario ni tags.

## 10. Componentes React sugeridos

Para mantener el desarrollo simple:

- `App`: estado principal y composicion de la pantalla.
- `TaskForm`: formulario de creacion y edicion.
- `TaskList`: renderizado de la lista ordenada.
- `TaskItem`: una tarea individual.
- `IceScore`: presentacion visual del puntaje ICE.
- `aiService`: funcion aislada para llamar a la API de IA.

## 11. Orden de desarrollo recomendado para el curso

1. Crear layout base de la aplicacion.
2. Crear formulario controlado con `useState`.
3. Agregar tareas a un array en estado local.
4. Calcular ICE manualmente.
5. Mostrar lista ordenada por ICE.
6. Implementar completar, editar y eliminar.
7. Crear funcion `calculateIceWithAI`.
8. Conectar boton "Calcular ICE con IA".
9. Manejar loading, error y respuesta invalida.
10. Pulir estilos y mensajes de interfaz.

## 12. Riesgos y limitaciones

- Al no haber backend, la clave de API queda visible en el cliente.
- La capa gratuita de una API de IA puede cambiar sus limites, modelos disponibles o condiciones de uso.
- La IA puede devolver valores inconsistentes, por lo que la aplicacion debe validar la respuesta.
- Sin persistencia, las tareas se pierden al recargar la pagina.
- El calculo ICE es una ayuda de priorizacion, no una decision automatica definitiva.

## 13. Resultado esperado

Una aplicacion React de una sola pantalla, facil de explicar en clase, que combine conceptos basicos de React con una integracion sencilla de IA:

- Componentes.
- Props.
- Estado local.
- Formularios controlados.
- Renderizado condicional.
- Listas.
- Eventos.
- Llamadas HTTP.
- Manejo de errores.
- Calculo derivado de datos.
