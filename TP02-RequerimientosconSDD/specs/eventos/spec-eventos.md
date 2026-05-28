# Spec: Gestión de Eventos y Catálogo

## 1. Objetivo y Contexto

Este módulo gestiona el ciclo de vida completo de los eventos académicos en la plataforma. Permite a los organizadores crear, editar y administrar eventos (cursos, jornadas, congresos, charlas, etc.), mientras que el catálogo de eventos es accesible públicamente para cualquier usuario. Los eventos poseen tipos, fechas de realización, configuración de cupos y fechas límite de inscripción.

### Alcance

- Listado público de eventos con filtros (futuros/pasados, por tipo, búsqueda textual)
- Creación, edición y eliminación lógica de eventos (organizador)
- Aprobación de eventos por administrador
- Configuración de cupos mínimos y máximos
- Configuración de fechas límite de inscripción
- Detalle de evento con toda su información
- Gestión de sesiones/actividades dentro de un evento

### Fuera de alcance

- Aprobación de eventos por parte de usuarios no admin
- Gestión de pagos o procesamiento de transacciones
- Streaming o integración con plataformas de video

---

## 2. Historias de Usuario y Criterios de Aceptación

### HU-EVT-01: Ver Listado Público de Eventos

**Como** cualquier usuario (autenticado o no), **quiero** ver un listado de eventos académicos **para** explorar las opciones disponibles.

**Criterios de aceptación:**

1. El listado de eventos es accesible sin autenticación.
2. Se muestran solo los eventos aprobados y no cancelados.
3. El listado es paginado (20 por página por defecto, máximo 100).
4. Cada tarjeta de evento muestra: título, tipo, fecha de inicio, fecha de fin, modalidad.
5. Se puede ordenar por fecha de inicio, fecha de creación o título.

### HU-EVT-02: Filtrar Eventos por Fecha (Futuros/Pasados)

**Como** usuario, **quiero** filtrar los eventos para ver solo los futuros o los que ya pasaron **para** encontrar eventos relevantes.

**Criterios de aceptación:**

1. El filtro `future=true` muestra solo eventos cuya fecha de fin es posterior a la fecha actual.
2. El filtro `past=true` muestra solo eventos cuya fecha de fin es anterior o igual a la fecha actual.
3. Si no se aplica filtro, se muestran todos los eventos aprobados.
4. El filtro se combina con otros filtros (tipo, búsqueda textual).

### HU-EVT-03: Filtrar Eventos por Tipo

**Como** usuario, **quiero** filtrar eventos por tipo (curso, jornada, congreso, charla) **para** encontrar eventos de mi interés.

**Criterios de aceptación:**

1. Los tipos disponibles son: curso, jornada, congreso, charla, taller, seminario, otro.
2. Se puede filtrar por un solo tipo.
3. Si no se especifica tipo, se muestran todos los tipos.

### HU-EVT-04: Ver Detalle de un Evento

**Como** usuario, **quiero** ver toda la información de un evento **para** decidir si participar.

**Criterios de aceptación:**

1. El detalle es accesible públicamente (sin autenticación).
2. Se muestra: título, descripción, tipo, fechas de inicio y fin, modalidad, ubicación (si presencial), URL de streaming (si virtual), organizador, cupos (si aplica), fecha límite de inscripción, tipo de inscripción, costo (si aplica).
3. Si el evento no fue aprobado aún, el detalle retorna 404 para usuarios no admin.
4. Si el evento fue cancelado, se muestra un indicador de cancelación.

### HU-EVT-05: Crear un Evento (Organizador)

**Como** usuario registrado, **quiero** crear un evento académico **para** organizar una actividad.

**Criterios de aceptación:**

1. Solo usuarios autenticados pueden crear eventos.
2. El evento se crea con estado "pendiente" de aprobación por admin.
3. Todos los campos obligatorios deben estar completos: título, descripción, tipo, fecha de inicio, fecha de fin, modalidad.
4. Los campos opcionales incluyen: ubicación, URL de streaming, cupo mínimo, cupo máximo, fecha límite de inscripción, tipo de inscripción, costo.
5. La fecha de fin debe ser posterior o igual a la fecha de inicio.
6. La fecha límite de inscripción debe ser anterior o igual a la fecha de inicio.
7. El cupo máximo, si se define, debe ser mayor que el cupo mínimo.
8. Al crear el evento, el usuario creador se asigna como organizador automáticamente.

### HU-EVT-06: Editar un Evento (Organizador)

**Como** organizador de un evento, **quiero** editar la información de mi evento **para** actualizar los detalles.

**Criterios de aceptación:**

1. Solo el organizador del evento puede editarlo.
2. Se pueden editar todos los campos excepto el tipo de evento una vez aprobado (para mantener coherencia).
3. Si el evento ya fue aprobado, cualquier edición vuelve a ponerlo en estado "pendiente" de re-aprobación.
4. Si el evento ya finalizó, no se puede editar.
5. Las mismas validaciones de creación aplican a la edición.
6. **[OWASP]** Al intentar actualizar el evento, el backend debe validar estrictamente el control de acceso a nivel de objeto (IDOR/BOLA), comprobando que el identificador del evento modificado pertenezca legítimamente al usuario autenticado. En caso de inconsistencia, denegar el acceso devolviendo un error HTTP 403 (Forbidden).
7. **[OWASP]** Sanitizar y validar las entradas de la actualización, comprobando específicamente que el campo de enlace de transmisión (streamUrl) cumpla con un formato estricto de URL segura (lista blanca de esquemas como `https://`) para evitar ataques de tipo Server-Side Request Forgery (SSRF) o inyecciones mediante esquemas maliciosos como `javascript:`.

### HU-EVT-07: Cancelar un Evento (Organizador)

**Como** organizador, **quiero** cancelar mi evento **para** notificar a los participantes que no se realizará.

**Criterios de aceptación:**

1. Solo el organizador puede cancelar su evento.
2. Al cancelar, el estado cambia a "cancelado" (soft delete con `deletedAt`).
3. Si el evento ya comenzó, no se puede cancelar (solo un admin puede).
4. El evento cancelado no aparece en el listado público.

### HU-EVT-08: Aprobar/Rechazar Evento (Admin)

**Como** administrador, **quiero** aprobar o rechazar los eventos creados **para** mantener la calidad de la plataforma.

**Criterios de aceptación:**

1. Solo usuarios con rol `admin` pueden aprobar o rechazar eventos.
2. Al aprobar, el estado cambia a "aprobado" y el evento aparece en el listado público.
3. Al rechazar, el estado cambia a "rechazado" y se puede incluir un motivo.
4. No se puede aprobar un evento que ya fue aprobado (409 `EVENT_ALREADY_APPROVED`).
5. No se puede aprobar un evento cuya fecha de inicio ya pasó (422 `EVENT_DATE_IN_PAST`).

### HU-EVT-09: Gestionar Sesiones de un Evento

**Como** organizador, **quiero** agregar sesiones/actividades a mi evento **para** definir la agenda.

**Criterios de aceptación:**

1. Solo el organizador puede crear, editar y eliminar sesiones de su evento.
2. Cada sesión tiene: título, descripción, disertante, fecha/hora de inicio, fecha/hora de fin, ubicación/sala.
3. La sesión debe estar dentro del rango de fechas del evento.
4. La fecha de fin de la sesión debe ser posterior a la de inicio.
5. Se pueden crear múltiples sesiones por evento.

---

## 3. Requisitos Funcionales y Reglas de Negocio

### RF-EVT-01: Catálogo Público

- **RF-EVT-01.1:** El listado de eventos es accesible sin autenticación.
- **RF-EVT-01.2:** Solo se muestran eventos con estado "aprobado" en el catálogo público.
- **RF-EVT-01.3:** Los eventos cancelados no aparecen en el catálogo público.
- **RF-EVT-01.4:** El catálogo soporta paginación, filtrado y ordenamiento.

### RF-EVT-02: Tipos de Evento

- **RF-EVT-02.1:** Los tipos válidos son: `CURSO`, `JORNADA`, `CONGRESO`, `CHARLA`, `TALLER`, `SEMINARIO`, `OTRO`.
- **RF-EVT-02.2:** El tipo se define al crear el evento y es obligatorio.
- **RF-EVT-02.3:** El tipo no se puede modificar una vez que el evento está aprobado.

### RF-EVT-03: Estados del Evento

- **RF-EVT-03.1:** Estados posibles: `pendiente`, `aprobado`, `rechazado`, `cancelado`, `finalizado`.
- **RF-EVT-03.2:** Estado inicial al crear: `pendiente`.
- **RF-EVT-03.3:** Transiciones válidas:
  - `pendiente` → `aprobado` (admin), `rechazado` (admin), `cancelado` (organizador)
  - `aprobado` → `cancelado` (organizador o admin), `finalizado` (sistema al pasar fecha fin)
  - `rechazado` → `pendiente` (organizador re-envía para revisión)
  - `finalizado` → (estado terminal, no hay transiciones)
- **RF-EVT-03.4:** Un evento pasa a `finalizado` automáticamente cuando la fecha actual es posterior a la fecha de fin (verificación on-demand).

### RF-EVT-04: Modalidad

- **RF-EVT-04.1:** Modalidades válidas: `PRESENCIAL`, `VIRTUAL`, `HIBRIDO`.
- **RF-EVT-04.2:** Si la modalidad es `PRESENCIAL` o `HIBRIDO`, la ubicación es obligatoria.
- **RF-EVT-04.3:** Si la modalidad es `VIRTUAL` o `HIBRIDO`, la URL de streaming es obligatoria.

### RF-EVT-05: Cupos y Fechas Límite

- **RF-EVT-05.1:** `minCapacity` es informativo y debe ser >= 0.
- **RF-EVT-05.2:** `maxCapacity` limita la cantidad de inscripciones aprobadas y debe ser > 0 si se define.
- **RF-EVT-05.3:** Si se define `maxCapacity`, debe ser >= `minCapacity`.
- **RF-EVT-05.4:** `maxCapacity` puede ser null (sin límite).
- **RF-EVT-05.5:** `enrollmentDeadline` debe ser <= `startDate`.
- **RF-EVT-05.6:** Si `enrollmentDeadline` es null, se toma `startDate` como límite implícito.

### RF-EVT-06: Fechas

- **RF-EVT-06.1:** `startDate` debe ser posterior a la fecha actual al momento de crear (para eventos nuevos).
- **RF-EVT-06.2:** `endDate` debe ser >= `startDate`.
- **RF-EVT-06.3:** Todas las fechas se almacenan en UTC.

### RF-EVT-07: Tipo de Inscripción y Costo

- **RF-EVT-07.1:** Tipos de inscripción: `AUTOMATICA`, `MANUAL`.
- **RF-EVT-07.2:** El costo es un valor decimal >= 0. Si es 0, el evento es gratuito.
- **RF-EVT-07.3:** No se procesan pagos reales en esta versión.

### RF-EVT-08: Ownership y Permisos

- **RF-EVT-08.1:** Solo el organizador (creador) puede editar o cancelar su evento.
- **RF-EVT-08.2:** Solo los admins pueden aprobar o rechazar eventos.
- **RF-EVT-08.3:** Los admins pueden ver y editar cualquier evento.
- **RF-EVT-08.4:** El detalle de evento aprobado es público.

### RF-EVT-09: Sesiones

- **RF-EVT-09.1:** Las sesiones pertenecen a un único evento.
- **RF-EVT-09.2:** La fecha/hora de inicio y fin de una sesión deben estar dentro del rango del evento.
- **RF-EVT-09.3:** Un evento puede tener 0 o más sesiones.
- **RF-EVT-09.4:** Las sesiones no requieren aprobación de admin.

---

## 4. Restricciones Técnicas Específicas

- **RT-EVT-01:** El listado público usa consultas con `where: { status: 'APROBADO', deletedAt: null }` en Prisma.
- **RT-EVT-02:** Los filtros de futuros/pasados se implementan comparando `endDate` con `new Date()` en UTC.
- **RT-EVT-03:** El soft delete se implementa con el campo `deletedAt` en el modelo Event.
- **RT-EVT-04:** La aprobación de eventos usa transacciones de Prisma para actualizar estado y registrar fecha de aprobación.
- **RT-EVT-05:** El ordenamiento por defecto es por `startDate` ascending para eventos futuros y `startDate` descending para eventos pasados.
- **RT-EVT-06:** La búsqueda textual (`search`) usa operador `contains` de Prisma sobre `title` y `description`.
- **RT-EVT-07:** Las sesiones se eliminan en cascada cuando se elimina el evento (`onDelete: Cascade`).

---

## 5. Modelo de Datos

### 5.1. Modelos

Este módulo introduce los siguientes modelos nuevos:

- `Event` - Entidad principal del evento con todos sus atributos
- `EventSession` - Sesiones/actividades dentro de un evento

Enum `EventType`: `CURSO`, `JORNADA`, `CONGRESO`, `CHARLA`, `TALLER`, `SEMINARIO`, `OTRO`

Enum `EventStatus`: `PENDIENTE`, `APROBADO`, `RECHAZADO`, `CANCELADO`, `FINALIZADO`

Enum `EventModality`: `PRESENCIAL`, `VIRTUAL`, `HIBRIDO`

Enum `EnrollmentType`: `AUTOMATICA`, `MANUAL`

### 5.2. DTOs de Respuesta

#### EventDTO

```typescript
interface EventDTO {
  id: number;
  title: string;
  description: string;
  type: string;
  status: string;
  modality: string;
  location: string | null;
  streamUrl: string | null;
  minCapacity: number | null;
  maxCapacity: number | null;
  enrollmentDeadline: string | null;
  enrollmentType: string;
  cost: number;
  startDate: string;
  endDate: string;
  organizer: {
    id: number;
    name: string;
    email: string;
  };
  createdAt: string;
  updatedAt: string;
}
```

#### EventListItemDTO

```typescript
interface EventListItemDTO {
  id: number;
  title: string;
  type: string;
  modality: string;
  startDate: string;
  endDate: string;
  location: string | null;
}
```

#### EventListMetaDTO

```typescript
interface EventListMetaDTO {
  page: number;
  limit: number;
  total: number;
  totalPages: number;
}
```

#### EventListResponseDTO

```typescript
interface EventListResponseDTO {
  events: EventListItemDTO[];
  meta: EventListMetaDTO;
}
```

#### EventSessionDTO

```typescript
interface EventSessionDTO {
  id: number;
  title: string;
  description: string | null;
  speaker: string;
  startTime: string;
  endTime: string;
  location: string | null;
}
```

#### EventDetailDTO

```typescript
interface EventDetailDTO {
  event: EventDTO;
  sessions: EventSessionDTO[];
  availableCapacity: number | null; // null si no hay maxCapacity
  isEnrolled: boolean; // true si el usuario autenticado está inscripto
}
```

#### CreateEventDTO

```typescript
interface CreateEventDTO {
  id: number;
  title: string;
  type: string;
  status: string; // siempre "PENDIENTE" al crear
  startDate: string;
  endDate: string;
  createdAt: string;
}
```

#### PendingEventDTO

```typescript
interface PendingEventDTO {
  id: number;
  title: string;
  type: string;
  organizer: {
    id: number;
    name: string;
    email: string;
  };
  startDate: string;
  endDate: string;
  createdAt: string;
}
```

---

## 6. Plan de Tareas

### Tarea 1: Definición de modelos y migraciones
- Definir enums `EventType`, `EventStatus`, `EventModality`, `EnrollmentType` en Prisma
- Definir modelo `Event` con todos los campos y relaciones
- Definir modelo `EventSession` con relación a `Event`
- Crear y ejecutar migración

### Tarea 2: Servicio de catálogo público
- Implementar `EventCatalogService` con métodos para listado público y detalle
- Implementar filtros por fecha (futuros/pasados), tipo, búsqueda textual
- Implementar paginación y ordenamiento
- Implementar ensamblado del `EventListResponseDTO` y `EventDetailDTO`

### Tarea 3: Servicio de gestión de eventos
- Implementar `EventManagementService` para crear, editar y cancelar eventos
- Implementar validaciones de fechas, cupos y modalidad
- Implementar lógica de transición de estados
- Implementar verificación de ownership (solo organizador)

### Tarea 4: Servicio de aprobación (admin)
- Implementar `EventApprovalService` para aprobar y rechazar eventos
- Implementar listado de eventos pendientes
- Implementar validación de fecha no pasada para aprobación

### Tarea 5: Servicio de sesiones
- Implementar `SessionService` para CRUD de sesiones
- Implementar validación de fechas dentro del rango del evento
- Implementar ensamblado del `EventSessionDTO`

### Tarea 6: Endpoints de catálogo público
- Implementar `GET /events` (listado público con filtros y paginación)
- Implementar `GET /events/:id` (detalle de evento)

### Tarea 7: Endpoints de gestión (organizador)
- Implementar `POST /events` (crear evento)
- Implementar `PUT /events/:id` (editar evento)
- Implementar `DELETE /events/:id` (cancelar evento - soft delete)
- Implementar `GET /events/organizer` (listar mis eventos)

### Tarea 8: Endpoints de aprobación (admin)
- Implementar `GET /events/pending` (listar eventos pendientes)
- Implementar `PUT /events/:id/approve` (aprobar evento)
- Implementar `PUT /events/:id/reject` (rechazar evento con motivo)

### Tarea 9: Endpoints de sesiones
- Implementar `POST /events/:id/sessions` (crear sesión)
- Implementar `PUT /sessions/:id` (editar sesión)
- Implementar `DELETE /sessions/:id` (eliminar sesión)
- Implementar `GET /events/:id/sessions` (listar sesiones de un evento)

### Tarea 10: Frontend - Catálogo público
- Crear página principal con listado de eventos
- Implementar paginación con controles de página
- Implementar filtros: futuros/pasados, por tipo, búsqueda textual
- Tarjetas de evento con título, tipo, fecha, modalidad
- Diseño responsivo con Bootstrap

### Tarea 11: Frontend - Detalle de evento
- Crear página de detalle de evento
- Mostrar toda la información del evento
- Mostrar sesiones en formato cronológico
- Indicador visual de evento cancelado

### Tarea 12: Frontend - Panel del organizador
- Crear página "Mis Eventos" para el organizador
- Formulario de creación de evento con validación
- Formulario de edición de evento
- Botones de cancelar evento

### Tarea 13: Frontend - Panel de administración
- Crear página de eventos pendientes de aprobación (solo admin)
- Acciones de aprobar/rechazar con campo de motivo

### Tarea 14: Frontend - Gestión de sesiones
- Formulario para agregar sesiones a un evento
- Tabla/listado de sesiones con edición
- Validación de fechas de sesión dentro del rango del evento

### Tarea 15: Tests
- Tests unitarios de validaciones de fechas y cupos
- Tests unitarios de transiciones de estado
- Tests de integración de endpoints de catálogo
- Tests de integración de endpoints de creación/edición/aprobación
- Tests de autorización (organizador, admin, público)

### Tarea 16: Documentación Swagger
- Agregar JSDoc swagger a todos los endpoints de eventos y sesiones

---

## 7. Estrategia de Verificación

### 7.1. Tests Unitarios

| Componente | Qué verificar |
|---|---|
| `validateEventDates` | startDate < endDate, enrollmentDeadline <= startDate |
| `validateEventCapacity` | minCapacity >= 0, maxCapacity > 0, maxCapacity >= minCapacity |
| `validateEventModality` | ubicación requerida para PRESENCIAL/HIBRIDO, streamUrl requerida para VIRTUAL/HIBRIDO |
| `canEditEvent` | No editar si evento finalizado, re-aprobación si ya aprobado |
| `canCancelEvent` | No cancelar si evento ya comenzó (excepto admin) |
| `validateSessionDates` | startTime y endTime dentro del rango del evento, startTime < endTime |
| `getEffectiveEnrollmentDeadline` | Retorna enrollmentDeadline o startDate si es null |

### 7.2. Tests de Integración

| Endpoint | Escenario | Resultado esperado |
|---|---|---|
| `GET /events` | Sin filtros | 200, EventListResponseDTO con eventos aprobados |
| `GET /events?future=true` | Solo futuros | 200, eventos con endDate > hoy |
| `GET /events?past=true` | Solo pasados | 200, eventos con endDate <= hoy |
| `GET /events?type=CONGRESO` | Filtrado por tipo | 200, solo congresos |
| `GET /events?search=taller` | Búsqueda textual | 200, eventos con "taller" en título o descripción |
| `GET /events/:id` | Evento aprobado existente | 200, EventDetailDTO completo |
| `GET /events/:id` | Evento pendiente (no admin) | 404, EVENT_NOT_FOUND |
| `POST /events` | Datos válidos | 201, evento creado con estado PENDIENTE |
| `POST /events` | Fecha inicio en pasado | 422, EVENT_DATE_IN_PAST |
| `POST /events` | maxCapacity < minCapacity | 400, VALIDATION_ERROR |
| `PUT /events/:id` | Organizador edita su evento aprobado | 200, estado vuelve a PENDIENTE |
| `PUT /events/:id` | No organizador intenta editar | 403, EVENT_NOT_ORGANIZER |
| `DELETE /events/:id` | Organizador cancela | 200, estado CANCELADO |
| `GET /events/pending` | Admin | 200, lista de PendingEventDTO |
| `GET /events/pending` | No admin | 403, FORBIDDEN |
| `PUT /events/:id/approve` | Admin aprueba | 200, estado APROBADO |
| `PUT /events/:id/approve` | Ya aprobado | 409, EVENT_ALREADY_APPROVED |
| `PUT /events/:id/reject` | Admin rechaza con motivo | 200, estado RECHAZADO con reason |
| `POST /events/:id/sessions` | Sesión válida | 201, sesión creada |
| `POST /events/:id/sessions` | Sesión fuera del rango del evento | 422, VALIDATION_ERROR |

### 7.3. Tests Manuales (QA)

1. Acceder al catálogo sin estar logueado → verificar que se muestran eventos aprobados.
2. Aplicar filtro de eventos futuros → verificar que solo se muestran eventos con fecha futura.
3. Aplicar filtro de eventos pasados → verificar que solo se muestran eventos finalizados.
4. Crear evento como organizador → verificar que queda en estado pendiente.
5. Como admin: aprobar evento → verificar que aparece en catálogo público.
6. Como admin: rechazar evento con motivo → verificar que el estado cambia y se guarda el motivo.
7. Editar evento aprobado → verificar que vuelve a estado pendiente.
8. Cancelar evento como organizador → verificar que desaparece del catálogo público.
9. Agregar sesiones a un evento → verificar que se muestran en el detalle.
10. Intentar crear sesión fuera del rango de fechas → debe fallar con error de validación.

### 7.4. Criterios de Aceptación del Módulo

- [ ] Todos los endpoints documentados en Swagger
- [ ] Cobertura de tests unitarios >= 70%
- [ ] Catálogo público accesible sin autenticación
- [ ] Filtros de futuros/pasados funcionan correctamente
- [ ] Filtros por tipo y búsqueda textual funcionan
- [ ] Creación de evento con todas las validaciones
- [ ] Aprobación/rechazo por admin funcional
- [ ] Edición de evento aprobado requiere re-aprobación
- [ ] Soft delete implementado correctamente
- [ ] Gestión de sesiones con validación de fechas
- [ ] Frontend responsivo con Bootstrap
- [ ] Paginación estándar conforme a contracts.md
