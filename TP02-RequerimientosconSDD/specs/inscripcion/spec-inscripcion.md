# Spec: Inscripción de Participantes y Cupos

## 1. Objetivo y Contexto

Este módulo gestiona el proceso de inscripción de participantes a eventos académicos. Maneja el flujo completo desde la solicitud de inscripción hasta la confirmación, considerando cupos, fechas límite y el tipo de inscripción (automática o manual). También permite que el personal del evento inscriba participantes de forma asistida.

### Alcance

- Inscripción autónoma del participante a un evento
- Inscripción asistida por personal del evento (organizador/staff)
- Cancelación de inscripción por el participante
- Aprobación/rechazo de inscripción por organizador (para inscripción manual)
- Gestión de cupos (mínimo y máximo)
- Registro de inscripciones por evento
- Manejo de eventos de pago (registro, sin procesamiento de pago real)

### Fuera de alcance

- Procesamiento de pagos reales (solo registro del estado de pago)
- Lista de espera (no implementada en esta versión)
- Reembolsos

---

## 2. Historias de Usuario y Criterios de Aceptación

### HU-INS-01: Inscribirse a un Evento

**Como** participante, **quiero** inscribirme a un evento **para** poder asistir.

**Criterios de aceptación:**

1. El participante puede inscribirse a cualquier evento con estado de inscripción "abierta".
2. Al inscribirse, se verifica que no exceda el cupo máximo.
3. Si el evento tiene inscripción automática, la inscripción queda "aprobada" inmediatamente.
4. Si el evento tiene inscripción manual, la inscripción queda "pendiente" de aprobación.
5. Un usuario no puede inscribirse dos veces al mismo evento.
6. Si la fecha límite de inscripción ya pasó, no se puede inscribir.
7. Si el evento está cancelado o no aprobado, no se puede inscribir.

### HU-INS-02: Cancelar Inscripción

**Como** participante inscripto, **quiero** cancelar mi inscripción **para** liberar mi lugar.

**Criterios de aceptación:**

1. El participante puede cancelar su inscripción mientras no haya pasado la fecha límite de inscripción.
2. Si la inscripción era automática, el lugar queda disponible inmediatamente.
3. Si la inscripción era manual y estaba pendiente, se elimina la solicitud.
4. Si la inscripción estaba aprobada y el evento ya comenzó, no se puede cancelar.
5. Al cancelar, se libera el cupo ocupado.

### HU-INS-03: Aprobar/Rechazar Inscripción (Organizador)

**Como** organizador de un evento con inscripción manual, **quiero** aprobar o rechazar las solicitudes de inscripción **para** controlar quién participa.

**Criterios de aceptación:**

1. El organizador puede ver todas las inscripciones pendientes de su evento.
2. El organizador puede aprobar una inscripción, cambiando su estado a "aprobada".
3. El organizador puede rechazar una inscripción con un motivo opcional.
4. Al aprobar, se verifica que aún haya cupo disponible.
5. Si al aprobar no hay cupo, retornar error.
6. El participante debe poder ver el estado de su inscripción.

### HU-INS-04: Inscribir a Otro Participante (Staff)

**Como** organizador o staff del evento, **quiero** inscribir a otra persona **para** registrar inscripciones que se hicieron fuera de la plataforma.

**Criterios de aceptación:**

1. El organizador puede inscribir a un usuario registrado por su email.
2. La inscripción hecha por el staff se marca como "aprobada" automáticamente.
3. Se registra quién realizó la inscripción (el staff/organizador).
4. Se pueden inscribir múltiples personas a la vez (carga masiva por emails).
5. Si el email no corresponde a un usuario registrado, se crea la inscripción pero se marca como "pendiente de registro" (el usuario debe registrarse para confirmarla).
6. [OWASP] El endpoint de carga masiva debe implementar un límite estricto en la cantidad de registros por petición (Rate Limiting y Payload Limit) para prevenir ataques de denegación de servicio (DoS) por agotamiento de recursos en el servidor de base de datos.
7. [OWASP] Cada dirección de correo electrónico recibida en el lote debe ser validada individualmente mediante una expresión regular robusta y segura (evitando ReDoS) antes de interactuar con Prisma, bloqueando caracteres maliciosos de inyección SQL/NoSQL.
8. [OWASP] En las respuestas de error de la carga masiva, no se deben revelar detalles internos del esquema de la base de datos ni indicar de forma explícita qué cuentas existen o no en el sistema a menos que sea estrictamente necesario para el flujo del staff, previniendo la enumeración masiva de usuarios externos.

### HU-INS-05: Ver Mis Inscripciones

**Como** participante, **quiero** ver todos los eventos a los que estoy inscripto **para** gestionar mi participación.

**Criterios de aceptación:**

1. Se muestran todas las inscripciones del usuario con estado (pendiente, aprobada, cancelada, rechazada).
2. Se muestra información básica del evento (título, fecha, estado).
3. Se puede filtrar por estado de la inscripción.
4. Desde la lista se puede acceder al detalle del evento o cancelar la inscripción.

### HU-INS-06: Ver Inscriptos del Evento (Organizador)

**Como** organizador, **quiero** ver la lista de inscriptos a mi evento **para** gestionar la participación.

**Criterios de aceptación:**

1. El organizador ve todas las inscripciones de su evento con datos del participante.
2. Se puede filtrar por estado de inscripción (pendiente, aprobada, cancelada, rechazada).
3. Se muestra el nombre, email y estado de cada inscripto.
4. La lista es paginada.
5. Se muestra el total de inscriptos aprobados vs cupo máximo.

### HU-INS-07: Cupo Mínimo y Máximo

**Como** organizador, **quiero** que se respeten los cupos definidos para mi evento **para** controlar la asistencia.

**Criterios de aceptación:**

1. No se permiten nuevas inscripciones aprobadas cuando se alcanza el cupo máximo.
2. El cupo mínimo no bloquea inscripciones (es informativo).
3. Si al cerrar la inscripción no se alcanzó el cupo mínimo, se notifica al organizador (se registra en log).

---

## 3. Requisitos Funcionales y Reglas de Negocio

### RF-INS-01: Inscripción Autónoma

- **RF-INS-01.1:** Solo usuarios autenticados pueden inscribirse a un evento.
- **RF-INS-01.2:** Un usuario solo puede tener una inscripción activa por evento.
- **RF-INS-01.3:** La inscripción verifica: evento aprobado, inscripción abierta, cupo disponible.

### RF-INS-02: Tipo de Inscripción

- **RF-INS-02.1:** Inscripción AUTOMATICA: se aprueba inmediatamente al registrarse.
- **RF-INS-02.2:** Inscripción MANUAL: queda pendiente hasta que el organizador la apruebe.
- **RF-INS-02.3:** El tipo de inscripción se define al crear el evento.

### RF-INS-03: Cupos

- **RF-INS-03.1:** El cupo máximo limita la cantidad de inscripciones con estado "aprobada".
- **RF-INS-03.2:** Las inscripciones "pendiente" no consumen cupo (se verifica cupo al momento de aprobar).
- **RF-INS-03.3:** Las inscripciones "cancelada" liberan el cupo.
- **RF-INS-03.4:** Si no hay cupo máximo definido, no hay límite de inscripciones.

### RF-INS-04: Fechas

- **RF-INS-04.1:** No se puede inscribir después de la fecha límite de inscripción.
- **RF-INS-04.2:** No se puede cancelar después de la fecha de inicio del evento.
- **RF-INS-04.3:** La fecha límite de inscripción debe ser anterior o igual a la fecha de inicio del evento.

### RF-INS-05: Inscripción por Staff

- **RF-INS-05.1:** Solo el organizador del evento puede inscribir a otros usuarios.
- **RF-INS-05.2:** Las inscripciones por staff se crean con estado "aprobada" automáticamente.
- **RF-INS-05.3:** Se registra el `enrolledById` (quién realizó la inscripción).
- **RF-INS-05.4:** Si el usuario no está registrado, se permite la inscripción con email (estado "pendiente de registro").

### RF-INS-06: Eventos de Pago

- **RF-INS-06.1:** Si el evento tiene costo mayor a 0, la inscripción se registra con estado de pago "pendiente".
- **RF-INS-06.2:** El organizador puede marcar el pago como "confirmado" manualmente.
- **RF-INS-06.3:** Un evento de pago requiere pago confirmado para que la inscripción esté "aprobada".
- **RF-INS-06.4:** No se procesan pagos reales en esta versión (solo registro del estado).

### RF-INS-07: Estados de Inscripción

- **RF-INS-07.1:** Estados posibles: `pendiente`, `aprobada`, `rechazada`, `cancelada`, `pendiente_registro`.
- **RF-INS-07.2:** Flujo de estados:
  - Automática: `aprobada` directamente.
  - Manual: `pendiente` → `aprobada` o `rechazada`.
  - Staff: `aprobada` directamente.
  - Usuario no registrado: `pendiente_registro` → `aprobada` (cuando se registra).

---

## 4. Restricciones Técnicas Específicas

- **RT-INS-01:** La verificación de cupo debe ser atómica para evitar race conditions (usar transacción de Prisma).
- **RT-INS-02:** El conteo de inscripciones aprobadas se calcula con Prisma `count` en cada operación.
- **RT-INS-03:** Los estados de inscripción se implementan como Prisma enum.
- **RT-INS-04:** El campo `enrolledById` es nullable (null cuando la inscripción es autónoma).

---

## 5. Modelo de Datos

### 5.1. Enum EnrollmentStatus

```prisma
enum EnrollmentStatus {
  PENDIENTE
  APROBADA
  RECHAZADA
  CANCELADA
  PENDIENTE_REGISTRO
}
```

### 5.2. Enum PaymentStatus

```prisma
enum PaymentStatus {
  NO_APLICA
  PENDIENTE
  CONFIRMADO
}
```

### 5.3. Modelo Enrollment

```prisma
model Enrollment {
  id              Int               @id @default(autoincrement())
  eventId         Int               @map("event_id")
  userId          Int?              @map("user_id") // null si el usuario no está registrado
  enrolledById    Int?              @map("enrolled_by_id") // quien inscribió (staff), null si autónoma
  status          EnrollmentStatus  @default(PENDIENTE)
  paymentStatus   PaymentStatus     @default(NO_APLICA) @map("payment_status")
  rejectionReason String?           @map("rejection_reason")
  guestEmail      String?           @map("guest_email") // email si el usuario no está registrado
  guestName       String?           @map("guest_name") // nombre si el usuario no está registrado
  enrolledAt      DateTime          @default(now()) @map("enrolled_at")
  updatedAt       DateTime          @updatedAt @map("updated_at")

  event           Event             @relation(fields: [eventId], references: [id])
  user            User?             @relation(fields: [userId], references: [id], onDelete: Cascade)
  enrolledBy      User?             @relation("EnrollmentEnroller", fields: [enrolledById], references: [id])

  @@unique([eventId, userId])
  @@unique([eventId, guestEmail])
  @@index([eventId])
  @@index([userId])
  @@index([status])
  @@map("enrollments")
}
```

### 5.4. Diagrama de Relaciones

```
Event 1 ──── N Enrollment
User 1 ──── N Enrollment (como participante)
User 1 ──── N Enrollment (como enroller, via enrolledBy)
```

### 5.5. Notas de Modelado

- `userId` es nullable para permitir inscripciones por staff de personas no registradas.
- Cuando `userId` es null, se usan `guestEmail` y `guestName`.
- La unicidad `[eventId, userId]` evita doble inscripción del mismo usuario.
- La unicidad `[eventId, guestEmail]` evita doble inscripción del mismo email invitado.

---

## 6. Plan de Tareas

### Tarea 1: Definición de modelos y migraciones
- Definir enums EnrollmentStatus y PaymentStatus en Prisma
- Definir modelo Enrollment con relaciones
- Crear y ejecutar migración

### Tarea 2: Endpoints de inscripción (participante)
- Implementar `POST /events/:id/enroll` (inscribirse a evento)
- Implementar `DELETE /enrollments/:id` (cancelar inscripción)
- Implementar `GET /enrollments/me` (ver mis inscripciones)
- Implementar `GET /enrollments/me/:id` (ver detalle de mi inscripción)

### Tarea 3: Endpoints de gestión de inscripciones (organizador)
- Implementar `GET /events/:id/enrollments` (ver inscriptos del evento)
- Implementar `PUT /enrollments/:id/status` (aprobar/rechazar inscripción)
- Implementar `POST /events/:id/enroll` (inscribir a otro participante - staff)
- Implementar `POST /events/:id/enroll/bulk` (inscripción masiva por emails)

### Tarea 4: Lógica de negocio de inscripciones
- Implementar verificación de cupo con transacción atómica
- Implementar verificación de fechas (deadline, inicio del evento)
- Implementar lógica de inscripción automática vs manual
- Implementar lógica de inscripción por staff
- Implementar cálculo de cupo disponible

### Tarea 5: Lógica de pagos
- Implementar actualización de estado de pago (solo organizador)
- Implementar validación: evento de pago requiere pago confirmado para aprobar
- Implementar endpoint `PUT /enrollments/:id/payment` (marcar pago como confirmado)

### Tarea 6: Frontend - Inscripción a evento
- Crear botón/tarjeta de inscripción en página de detalle de evento
- Mostrar estado de inscripción (no inscripto, pendiente, aprobado)
- Mostrar mensaje de error si no hay cupo o inscripción cerrada
- Confirmación visual de inscripción exitosa

### Tarea 7: Frontend - Mis inscripciones
- Crear página "Mis Inscripciones" en perfil del usuario
- Mostrar lista de inscripciones con estado y datos del evento
- Permitir cancelar inscripción (si es posible)
- Filtrar por estado

### Tarea 8: Frontend - Gestión de inscripciones (organizador)
- Crear página "Inscriptos" en panel del organizador
- Tabla de inscriptos con nombre, email, estado, fecha de inscripción
- Acciones: aprobar, rechazar (con motivo), ver detalle
- Mostrar contador de cupo usado/disponible
- Formulario para inscribir a otro participante (por email)

### Tarea 9: Frontend - Gestión de pagos
- Indicar en la tabla de inscriptos si el pago está pendiente
- Botón para marcar pago como confirmado (solo organizador)

### Tarea 10: Tests
- Tests unitarios de lógica de cupo y fechas
- Tests de integración de endpoints de inscripción
- Tests de concurrencia (race condition de cupo)
- Tests de inscripción por staff

### Tarea 11: Documentación Swagger
- Agregar JSDoc swagger a todos los endpoints de inscripción

---

## 7. Estrategia de Verificación

### 7.1. Tests Unitarios

| Componente | Qué verificar |
|---|---|
| `checkEnrollmentEligibility` | Evento aprobado, inscripción abierta, usuario no duplicado |
| `checkCapacity` | Cupo disponible >= 1 para inscripciones automáticas o aprobaciones |
| `calculateAvailableCapacity` | maxCapacity - count(aprobadas) |
| `canCancelEnrollment` | Fecha actual <= fecha inicio del evento |

### 7.2. Tests de Integración

| Endpoint | Escenario | Resultado esperado |
|---|---|---|
| `POST /events/:id/enroll` | Inscripción automática válida | 201, enrollment con status APROBADA |
| `POST /events/:id/enroll` | Inscripción manual válida | 201, enrollment con status PENDIENTE |
| `POST /events/:id/enroll` | Duplicado | 409, ENROLLMENT_ALREADY_EXISTS |
| `POST /events/:id/enroll` | Cupo lleno | 409, CAPACITY_REACHED |
| `POST /events/:id/enroll` | Inscripción cerrada | 422, ENROLLMENT_CLOSED |
| `DELETE /enrollments/:id` | Cancelar antes del evento | 204, status CANCELADA |
| `DELETE /enrollments/:id` | Cancelar después del inicio | 422, no se puede cancelar |
| `GET /enrollments/me` | Usuario con inscripciones | 200, lista de inscripciones |
| `GET /events/:id/enrollments` | Organizador de evento | 200, lista de inscriptos |
| `PUT /enrollments/:id/status` | Aprobar inscripción pendiente | 200, status APROBADA |
| `PUT /enrollments/:id/status` | Aprobar sin cupo | 409, CAPACITY_REACHED |
| `PUT /enrollments/:id/status` | Rechazar con motivo | 200, status RECHAZADA con reason |
| `POST /events/:id/enroll` (staff) | Inscribir usuario existente | 201, enrollment APROBADA |
| `POST /events/:id/enroll` (staff) | Inscribir email no registrado | 201, enrollment PENDIENTE_REGISTRO |

### 7.3. Tests Manuales (QA)

1. Inscribirse a evento automático → verificar que queda aprobada inmediatamente.
2. Inscribirse a evento manual → verificar que queda pendiente → como organizador aprobar → verificar que queda aprobada.
3. Llenar cupo máximo → intentar inscribirse → debe fallar con CAPACITY_REACHED.
4. Cancelar inscripción → verificar que se libera cupo → otro usuario puede inscribirse.
5. Como staff: inscribir a persona con email no registrado → verificar estado pendiente_registro.
6. Evento de pago: inscribirse → verificar pago pendiente → organizador marca confirmado → verificar inscripción aprobada.

### 7.4. Criterios de Aceptación del Módulo

- [ ] Todos los endpoints documentados en Swagger
- [ ] Cobertura de tests unitarios >= 70%
- [ ] Inscripción automática y manual funcionando correctamente
- [ ] Cupo máximo se respeta correctamente (incluyendo concurrencia)
- [ ] Inscripción por staff de usuarios registrados y no registrados
- [ ] Cancelación de inscripción con validación de fechas
- [ ] Gestión de pagos (estado) funcional
- [ ] Panel de organizador muestra inscriptos con filtros
- [ ] Frontend responsivo con Bootstrap
