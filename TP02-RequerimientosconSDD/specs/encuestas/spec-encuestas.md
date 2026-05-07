# Spec: Encuestas y Comentarios

## 1. Objetivo y Contexto

Este módulo gestiona el feedback post-evento de los participantes. Permite a los organizadores crear encuestas de satisfacción para evaluar sus eventos, y a los participantes responderlas. También permite dejar comentarios y valoraciones sobre la experiencia en el evento. Este módulo es clave para la mejora continua de los eventos académicos y proporciona datos valiosos para los informes.

### Alcance

- Creación y gestión de encuestas por parte del organizador
- Respuesta a encuestas por participantes aprobados
- Comentarios y valoraciones post-evento por participantes
- Listado de comentarios públicos por evento
- Resultados y estadísticas de encuestas para el organizador
- Activación de encuestas/comentarios solo después de que el evento finaliza

### Fuera de alcance

- Encuestas pre-evento
- Encuestas anónimas (siempre vinculadas al usuario)
- Moderación de comentarios por parte de admin
- Notificaciones push o email para encuestas

---

## 2. Historias de Usuario y Criterios de Aceptación

### HU-ENC-01: Crear Encuesta (Organizador)

**Como** organizador de un evento, **quiero** crear una encuesta de satisfacción **para** recibir feedback de los participantes.

**Criterios de aceptación:**

1. Solo el organizador del evento puede crear encuestas.
2. Una encuesta tiene: título, descripción, y una lista de preguntas.
3. Los tipos de pregunta soportados son: escala (1-5), opción múltiple (selección única), texto abierto (respuesta libre).
4. Para preguntas de tipo opción múltiple, se deben definir las opciones posibles.
5. Una encuesta se crea en estado "borrador" y el organizador la activa ("activa") cuando está lista.
6. Solo se permite una encuesta activa por evento.
7. La encuesta no se puede activar hasta que el evento haya finalizado.

### HU-ENC-02: Responder Encuesta (Participante)

**Como** participante aprobado de un evento, **quiero** responder la encuesta de satisfacción **para** dar mi opinión sobre el evento.

**Criterios de aceptación:**

1. Solo usuarios con inscripción "aprobada" en el evento pueden responder la encuesta.
2. La encuesta debe estar en estado "activa".
3. El evento debe haber finalizado (endDate <= fecha actual).
4. Un usuario solo puede responder la encuesta una vez por evento.
5. Todas las preguntas marcadas como obligatorias deben ser respondidas.
6. Las preguntas de tipo escala aceptan valores del 1 al 5.
7. Las preguntas de tipo opción múltiple deben seleccionar una de las opciones predefinidas.
8. Las preguntas de tipo texto abierto aceptan texto libre (máximo 1000 caracteres).
9. Al enviar la encuesta, se registra la fecha de respuesta.

### HU-ENC-03: Ver Resultados de Encuesta (Organizador)

**Como** organizador, **quiero** ver los resultados de las respuestas a mi encuesta **para** analizar el feedback.

**Criterios de aceptación:**

1. Solo el organizador puede ver los resultados.
2. Se muestra: total de respuestas, promedio de satisfacción por pregunta de escala, distribución de respuestas por pregunta.
3. Para preguntas de texto abierto, se muestran todas las respuestas individuales.
4. Para preguntas de opción múltiple, se muestra el conteo por opción.

### HU-ENC-04: Dejar Comentario en un Evento

**Como** participante aprobado de un evento, **quiero** dejar un comentario con una valoración **para** compartir mi experiencia.

**Criterios de aceptación:**

1. Solo usuarios con inscripción "aprobada" pueden comentar.
2. El evento debe haber finalizado.
3. Los comentarios deben estar habilitados para el evento (configuración del organizador).
4. Un comentario tiene: texto (obligatorio, max 1000 caracteres), rating opcional (1-5), y autor.
5. El comentario se publica inmediatamente y es visible públicamente.
6. Un usuario puede dejar solo un comentario por evento.

### HU-ENC-05: Ver Comentarios de un Evento

**Como** cualquier usuario, **quiero** ver los comentarios de un evento **para** conocer las experiencias de otros participantes.

**Criterios de aceptación:**

1. Los comentarios son visibles públicamente (sin autenticación).
2. Se muestran ordenados por fecha de creación (más recientes primero).
3. Cada comentario muestra: nombre del autor, texto, rating (si tiene), fecha.
4. El listado es paginado.
5. Se puede filtrar por rating (mostrar solo comentarios con rating >= X).

### HU-ENC-06: Editar/Eliminar Mi Comentario

**Como** autor de un comentario, **quiero** editar o eliminar mi comentario **para** corregir o remover mi opinión.

**Criterios de aceptación:**

1. Solo el autor puede editar o eliminar su propio comentario.
2. Se puede editar el texto y el rating.
3. Al eliminar, el comentario se borra completamente (hard delete).
4. No se puede editar después de 30 días desde la publicación.

### HU-ENC-07: Configurar Comentarios del Evento

**Como** organizador, **quiero** habilitar o deshabilitar los comentarios en mi evento **para** controlar si los participantes pueden opinar públicamente.

**Criterios de aceptación:**

1. Por defecto, los comentarios están habilitados.
2. El organizador puede deshabilitar los comentarios en cualquier momento.
3. Al deshabilitar, no se pueden crear nuevos comentarios pero los existentes permanecen visibles.
4. El organizador puede volver a habilitar los comentarios.

### HU-ENC-08: Estadísticas de Satisfacción del Evento

**Como** organizador o administrador, **quiero** ver las estadísticas globales de satisfacción de mi evento **para** tener una métrica general de calidad.

**Criterios de aceptación:**

1. Se calcula el promedio general de satisfacción basado en: promedio de preguntas de escala de todas las respuestas de encuesta + promedio de ratings de todos los comentarios con rating.
2. Se muestra el total de respuestas de encuesta y el total de comentarios.
3. Se muestra la tasa de respuesta: (respuestas de encuesta / inscriptos aprobados) * 100.
4. Si no hay datos, se indica que aún no hay feedback disponible.

---

## 3. Requisitos Funcionales y Reglas de Negocio

### RF-ENC-01: Creación de Encuestas

- **RF-ENC-01.1:** Solo el organizador puede crear encuestas para su evento.
- **RF-ENC-01.2:** Cada encuesta tiene un título (obligatorio, max 200 chars) y descripción (opcional, max 500 chars).
- **RF-ENC-01.3:** Una encuesta debe tener al menos 1 pregunta.
- **RF-ENC-01.4:** Cada pregunta tiene: texto (obligatorio, max 300 chars), tipo (escala, opción múltiple, texto abierto), y flag de obligatoriedad.
- **RF-ENC-01.5:** Las preguntas de tipo `opcion_multiple` deben tener al menos 2 opciones.
- **RF-ENC-01.6:** Estado inicial de la encuesta: `BORRADOR`.
- **RF-ENC-01.7:** Solo se puede activar una encuesta si el evento finalizó.

### RF-ENC-02: Tipos de Pregunta

- **RF-ENC-02.1:** `ESCALA`: respuesta numérica del 1 al 5.
- **RF-ENC-02.2:** `OPCION_MULTIPLE`: selección de una opción predefinida.
- **RF-ENC-02.3:** `TEXTO_ABIERTO`: respuesta libre en texto (max 1000 chars).

### RF-ENC-03: Respuestas a Encuestas

- **RF-ENC-03.1:** Solo participantes con inscripción `APROBADA` pueden responder.
- **RF-ENC-03.2:** Un usuario responde una encuesta una sola vez por evento.
- **RF-ENC-03.3:** Las respuestas se guardan como un conjunto completo (no parcial).
- **RF-ENC-03.4:** Las preguntas obligatorias sin respuesta generan error de validación.
- **RF-ENC-03.5:** Se registra `respondedAt` con la fecha/hora de la respuesta.

### RF-ENC-04: Estados de Encuesta

- **RF-ENC-04.1:** Estados: `BORRADOR`, `ACTIVA`, `CERRADA`.
- **RF-ENC-04.2:** Transiciones:
  - `BORRADOR` → `ACTIVA` (organizador, solo si evento finalizó)
  - `ACTIVA` → `CERRADA` (organizador o automático tras 30 días)
  - `CERRADA` → (estado terminal)

### RF-ENC-05: Comentarios

- **RF-ENC-05.1:** Solo participantes con inscripción `APROBADA` pueden comentar.
- **RF-ENC-05.2:** El evento debe haber finalizado.
- **RF-ENC-05.3:** Los comentarios deben estar habilitados para el evento (`commentsEnabled: true`).
- **RF-ENC-05.4:** Un usuario solo puede tener un comentario por evento.
- **RF-ENC-05.5:** El texto es obligatorio (max 1000 chars).
- **RF-ENC-05.6:** El rating es opcional (1-5).
- **RF-ENC-05.7:** Los comentarios son públicos (visibles sin autenticación).

### RF-ENC-06: Edición de Comentarios

- **RF-ENC-06.1:** Solo el autor puede editar su comentario.
- **RF-ENC-06.2:** Se puede editar dentro de los 30 días posteriores a la publicación.
- **RF-ENC-06.3:** Se puede eliminar en cualquier momento (solo el autor).
- **RF-ENC-06.4:** La eliminación es hard delete (no soft delete).

### RF-ENC-07: Configuración del Evento

- **RF-ENC-07.1:** El campo `commentsEnabled` en el modelo Event controla si se permiten comentarios.
- **RF-ENC-07.2:** Valor por defecto: `true`.
- **RF-ENC-07.3:** El organizador puede cambiar este valor en cualquier momento.

### RF-ENC-08: Acceso y Permisos

- **RF-ENC-08.1:** Las encuestas y sus resultados solo son accesibles por el organizador y admins.
- **RF-ENC-08.2:** Los comentarios son públicos (cualquier usuario puede verlos).
- **RF-ENC-08.3:** Responder encuestas y dejar comentarios requiere autenticación e inscripción aprobada.

---

## 4. Restricciones Técnicas Específicas

- **RT-ENC-01:** La verificación de inscripción aprobada se hace con una consulta Prisma al modelo `Enrollment`.
- **RT-ENC-02:** Las respuestas de encuesta se guardan en transacción para asegurar atomicidad (todas las preguntas o ninguna).
- **RT-ENC-03:** El cálculo de estadísticas usa consultas agregadas de Prisma (`average`, `count`, `groupBy`).
- **RT-ENC-04:** Los comentarios usan hard delete (no tienen `deletedAt`).
- **RT-ENC-05:** El cierre automático de encuestas tras 30 días se implementa como verificación on-demand (no cron job).
- **RT-ENC-06:** Se usa `@@unique([eventId])` en Survey para garantizar una sola encuesta por evento.
- **RT-ENC-07:** Se usa `@@unique([eventId, userId])` en Comment para garantizar un comentario por usuario por evento.

---

## 5. Modelo de Datos

### 5.1. Modelos

Este módulo introduce los siguientes modelos nuevos:

- `Survey` - Encuesta asociada a un evento
- `SurveyQuestion` - Preguntas dentro de una encuesta
- `SurveyQuestionOption` - Opciones para preguntas de tipo opción múltiple
- `SurveyResponse` - Respuesta completa de un usuario a una encuesta
- `SurveyAnswer` - Respuesta individual a cada pregunta
- `Comment` - Comentarios públicos sobre un evento

Además, se agrega el campo `commentsEnabled` al modelo `Event` existente.

Enum `SurveyStatus`: `BORRADOR`, `ACTIVA`, `CERRADA`

Enum `QuestionType`: `ESCALA`, `OPCION_MULTIPLE`, `TEXTO_ABIERTO`

### 5.2. DTOs de Respuesta

#### SurveyQuestionDTO

```typescript
interface SurveyQuestionDTO {
  id: number;
  text: string;
  type: string;
  isRequired: boolean;
  orderIndex: number;
  options?: {
    id: number;
    text: string;
    orderIndex: number;
  }[];
}
```

#### SurveyDTO

```typescript
interface SurveyDTO {
  id: number;
  title: string;
  description: string | null;
  status: string;
  questions: SurveyQuestionDTO[];
  createdAt: string;
  updatedAt: string;
  activatedAt: string | null;
}
```

#### SurveyResponseDTO

```typescript
interface SurveyResponseDTO {
  id: number;
  respondedAt: string;
  answers: {
    questionId: number;
    questionText: string;
    questionType: string;
    scaleValue: number | null;
    textValue: string | null;
    optionId: number | null;
    optionText: string | null;
  }[];
}
```

#### SurveyResultsDTO

```typescript
interface SurveyResultsDTO {
  survey: {
    id: number;
    title: string;
    status: string;
  };
  totalResponses: number;
  responseRate: number; // porcentaje sobre inscriptos aprobados
  questionResults: {
    questionId: number;
    text: string;
    type: string;
    averageScale?: number | null; // promedio 1-5 para preguntas de escala
    distribution?: { // para preguntas de opción múltiple
      optionId: number;
      optionText: string;
      count: number;
      percentage: number;
    }[];
    textResponses?: string[]; // respuestas de texto abierto
  }[];
}
```

#### CommentDTO

```typescript
interface CommentDTO {
  id: number;
  text: string;
  rating: number | null;
  author: {
    name: string;
  };
  createdAt: string;
  updatedAt: string;
}
```

#### CommentListResponseDTO

```typescript
interface CommentListResponseDTO {
  comments: CommentDTO[];
  meta: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}
```

#### SatisfactionStatsDTO

```typescript
interface SatisfactionStatsDTO {
  averageSatisfaction: number | null; // promedio combinado (encuestas + comentarios)
  surveyAverage: number | null; // promedio solo de encuestas
  commentAverage: number | null; // promedio solo de comentarios
  totalSurveyResponses: number;
  totalComments: number;
  responseRate: number; // porcentaje sobre inscriptos aprobados
}
```

---

## 6. Plan de Tareas

### Tarea 1: Actualizar modelo Event y definir nuevos modelos
- Agregar campo `commentsEnabled` al modelo `Event`
- Definir enums `SurveyStatus` y `QuestionType` en Prisma
- Definir modelos `Survey`, `SurveyQuestion`, `SurveyQuestionOption`, `SurveyResponse`, `SurveyAnswer`, `Comment`
- Crear y ejecutar migración

### Tarea 2: Servicio de encuestas (organizador)
- Implementar `SurveyService` con métodos para crear, editar, activar y cerrar encuestas
- Implementar validación de tipos de pregunta y opciones
- Implementar verificación de evento finalizado para activación
- Implementar cierre automático on-demand (verificar 30 días)
- Implementar ensamblado del `SurveyDTO`

### Tarea 3: Servicio de respuestas de encuesta
- Implementar `SurveyResponseService` para procesar respuestas
- Implementar validación de inscripción aprobada, encuesta activa, evento finalizado
- Implementar guardado atómico en transacción (todas las preguntas)
- Implementar verificación de preguntas obligatorias

### Tarea 4: Servicio de resultados
- Implementar `SurveyResultsService` para calcular estadísticas
- Implementar consultas agregadas: promedio por pregunta de escala, conteo por opción
- Implementar cálculo de tasa de respuesta
- Implementar ensamblado del `SurveyResultsDTO`

### Tarea 5: Servicio de comentarios
- Implementar `CommentService` para CRUD de comentarios
- Implementar validación de texto (max 1000 chars) y rating (1-5)
- Implementar verificación de `commentsEnabled` en el evento
- Implementar verificación de plazo de edición (30 días)
- Implementar ensamblado del `CommentDTO` y `CommentListResponseDTO`

### Tarea 6: Servicio de estadísticas
- Implementar `SatisfactionStatsService` para calcular métricas combinadas
- Implementar promedio combinado de encuestas + comentarios
- Implementar ensamblado del `SatisfactionStatsDTO`

### Tarea 7: Endpoints de gestión de encuestas (organizador)
- Implementar `POST /events/:id/survey` (crear encuesta con preguntas)
- Implementar `PUT /surveys/:id` (editar encuesta en borrador)
- Implementar `PUT /surveys/:id/activate` (activar encuesta)
- Implementar `PUT /surveys/:id/close` (cerrar encuesta)
- Implementar `GET /events/:id/survey` (ver encuesta del evento)

### Tarea 8: Endpoints de respuesta a encuestas (participante)
- Implementar `POST /surveys/:id/respond` (responder encuesta)
- Implementar `GET /surveys/:id/my-response` (ver mi respuesta)

### Tarea 9: Endpoints de resultados (organizador)
- Implementar `GET /surveys/:id/results` (ver resultados estadísticos)

### Tarea 10: Endpoints de comentarios
- Implementar `POST /events/:id/comments` (crear comentario)
- Implementar `GET /events/:id/comments` (listar comentarios públicos con paginación)
- Implementar `PUT /comments/:id` (editar mi comentario)
- Implementar `DELETE /comments/:id` (eliminar mi comentario)
- Implementar `GET /events/:id/comments/stats` (estadísticas de satisfacción)
- Soportar query param `minRating` para filtrar comentarios

### Tarea 11: Endpoint de configuración de comentarios (organizador)
- Implementar `PUT /events/:id/comments-config` (habilitar/deshabilitar comentarios)

### Tarea 12: Frontend - Crear encuesta (organizador)
- Formulario de creación de encuesta con preguntas dinámicas
- Selector de tipo de pregunta para cada pregunta
- Agregar opciones para preguntas de opción múltiple
- Toggle de obligatoriedad por pregunta
- Botón de activar encuesta (solo si evento finalizó)

### Tarea 13: Frontend - Responder encuesta (participante)
- Página de encuesta con renderizado dinámico según tipo de pregunta
- Inputs de escala (1-5 estrellas o radio buttons)
- Radio buttons para opción múltiple
- Textarea para texto abierto
- Validación de preguntas obligatorias

### Tarea 14: Frontend - Resultados de encuesta (organizador)
- Página de resultados con métricas
- Gráficos de barras para preguntas de escala (promedio)
- Gráficos de torta para preguntas de opción múltiple
- Listado de respuestas de texto abierto

### Tarea 15: Frontend - Comentarios
- Sección de comentarios en la página de detalle del evento
- Formulario para dejar comentario (texto + rating opcional)
- Listado de comentarios con paginación
- Indicador visual de rating (estrellas)
- Filtro por rating mínimo
- Botones de editar/eliminar para el autor
- Toggle para habilitar/deshabilitar comentarios (organizador)

### Tarea 16: Frontend - Estadísticas de satisfacción
- Mostrar promedio general de satisfacción en el detalle del evento
- Mostrar total de respuestas y comentarios
- Mostrar tasa de respuesta

### Tarea 17: Tests
- Tests unitarios de validación de respuestas de encuesta
- Tests unitarios de cálculos estadísticos
- Tests de integración de endpoints de encuestas
- Tests de integración de endpoints de comentarios
- Tests de permisos (solo organizador, solo participante aprobado, público)
- Tests de verificación de evento finalizado

### Tarea 18: Documentación Swagger
- Agregar JSDoc swagger a todos los endpoints de encuestas y comentarios

---

## 7. Estrategia de Verificación

### 7.1. Tests Unitarios

| Componente | Qué verificar |
|---|---|
| `validateSurveyAnswers` | Todas las preguntas obligatorias respondidas, tipos correctos |
| `validateScaleAnswer` | Valor entre 1 y 5 |
| `validateMultipleChoiceAnswer` | Opción existe en la pregunta |
| `validateTextAnswer` | No vacío, max 1000 chars |
| `canActivateSurvey` | Evento finalizó, encuesta en borrador, tiene al menos 1 pregunta |
| `canRespondToSurvey` | Inscripción aprobada, encuesta activa, evento finalizado, no respondió antes |
| `canEditComment` | Es autor, dentro de 30 días |
| `canComment` | Inscripción aprobada, evento finalizado, commentsEnabled = true, no comentó antes |
| `calculateSurveyResults` | Promedios correctos, conteos correctos, tasa de respuesta |
| `calculateSatisfactionStats` | Promedio combinado de encuestas + comentarios |

### 7.2. Tests de Integración

| Endpoint | Escenario | Resultado esperado |
|---|---|---|
| `POST /events/:id/survey` | Organizador crea encuesta válida | 201, encuesta en BORRADOR |
| `POST /events/:id/survey` | Sin preguntas | 400, VALIDATION_ERROR |
| `PUT /surveys/:id/activate` | Evento no finalizó | 422, SURVEY_NOT_AVAILABLE |
| `PUT /surveys/:id/activate` | Evento finalizó, encuesta válida | 200, encuesta ACTIVA |
| `POST /surveys/:id/respond` | Participante aprobado, encuesta activa | 201, respuesta creada |
| `POST /surveys/:id/respond` | Usuario ya respondió | 409, SURVEY_ALREADY_RESPONDED |
| `POST /surveys/:id/respond` | Inscripción no aprobada | 403, FORBIDDEN |
| `POST /surveys/:id/respond` | Pregunta obligatoria sin respuesta | 400, VALIDATION_ERROR |
| `GET /surveys/:id/results` | Organizador | 200, SurveyResultsDTO completo |
| `GET /surveys/:id/results` | No organizador | 403, FORBIDDEN |
| `POST /events/:id/comments` | Participante aprobado, evento finalizado | 201, comentario creado |
| `POST /events/:id/comments` | commentsEnabled = false | 422, COMMENT_NOT_ALLOWED |
| `POST /events/:id/comments` | Usuario ya comentó | 409, error de unicidad |
| `POST /events/:id/comments` | Texto > 1000 chars | 400, VALIDATION_ERROR |
| `GET /events/:id/comments` | Sin autenticar | 200, CommentListResponseDTO paginada |
| `GET /events/:id/comments?minRating=4` | Filtro por rating | 200, solo comentarios con rating >= 4 |
| `PUT /comments/:id` | Autor edita dentro de 30 días | 200, comentario actualizado |
| `PUT /comments/:id` | Autor edita después de 30 días | 422, no se puede editar |
| `DELETE /comments/:id` | Autor elimina | 204, comentario eliminado |
| `DELETE /comments/:id` | No autor intenta eliminar | 403, FORBIDDEN |
| `GET /events/:id/comments/stats` | Evento con feedback | 200, SatisfactionStatsDTO con datos |
| `GET /events/:id/comments/stats` | Evento sin feedback | 200, stats con valores null/0 |
| `PUT /events/:id/comments-config` | Organizador deshabilita | 200, commentsEnabled = false |

### 7.3. Tests Manuales (QA)

1. Como organizador: crear encuesta con 3 preguntas (escala, opción múltiple, texto) → verificar que queda en borrador.
2. Como organizador: intentar activar encuesta antes de que el evento finalice → debe fallar.
3. Avanzar fecha del evento (o usar evento pasado): activar encuesta → verificar que queda activa.
4. Como participante aprobado: responder encuesta → verificar que se guardan todas las respuestas.
5. Intentar responder encuesta por segunda vez → debe fallar con SURVEY_ALREADY_RESPONDED.
6. Como organizador: ver resultados → verificar promedios y conteos correctos.
7. Como participante aprobado: dejar comentario con rating → verificar que aparece públicamente.
8. Como usuario no autenticado: ver comentarios → verificar que son visibles.
9. Como autor: editar comentario dentro de 30 días → verificar que se actualiza.
10. Como organizador: deshabilitar comentarios → verificar que no se pueden crear nuevos.
11. Ver estadísticas de satisfacción → verificar promedio combinado correcto.

### 7.4. Criterios de Aceptación del Módulo

- [ ] Todos los endpoints documentados en Swagger
- [ ] Cobertura de tests unitarios >= 70%
- [ ] Creación de encuestas con múltiples tipos de pregunta
- [ ] Activación de encuesta solo después de finalizado el evento
- [ ] Respuesta a encuesta con validación de inscripción aprobada
- [ ] Un usuario solo responde una encuesta por evento
- [ ] Resultados estadísticos con promedios y conteos correctos
- [ ] Comentarios públicos visibles sin autenticación
- [ ] Un usuario solo comenta una vez por evento
- [ ] Edición de comentarios limitada a 30 días
- [ ] Configuración de comentarios habilitados/deshabilitados
- [ ] Estadísticas de satisfacción combinadas (encuestas + comentarios)
- [ ] Frontend responsivo con Bootstrap
