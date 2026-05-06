# Spec: Acreditación y Certificados

## 1. Objetivo y Contexto

Este módulo gestiona el proceso de acreditación (registro de asistencia) de participantes a eventos académicos y la generación de certificados en distintos formatos según el rol y la participación del usuario. La acreditación permite confirmar que un participante efectivamente asistió al evento, y los certificados acreditan formalmente dicha participación.

### Alcance

- Registro de asistencia (check-in) de participantes acreditados
- Verificación del estado de acreditación de un participante
- Estadísticas de acreditación por evento
- Generación de certificados por tipo (asistencia, aprobación, autor/expositor)
- Descarga de certificados en formato PDF
- Verificación pública de validez de certificados mediante código
- Listado de certificados emitidos por usuario

### Fuera de alcance

- Verificación de identidad con documento/DNI (no se requiere validación biométrica)
- Certificados con firma digital avanzada (no se implementa firma criptográfica)
- Notificaciones automáticas por email de certificado disponible (se puede agregar en futuro)

---

## 2. Historias de Usuario y Criterios de Aceptación

### HU-ACR-01: Registrar Asistencia (Check-in)

**Como** organizador o staff del evento, **quiero** registrar la asistencia de un participante **para** confirmar que asistió al evento.

**Criterios de aceptación:**

1. Solo el organizador del evento puede registrar asistencia.
2. El participante debe tener una inscripción con estado "aprobada" en el evento.
3. Si el participante ya tiene asistencia registrada, no se permite duplicar.
4. La fecha de acreditación se registra automáticamente (timestamp UTC).
5. Se puede registrar asistencia en cualquier momento después del inicio del evento.

### HU-ACR-02: Ver Estado de Acreditación

**Como** organizador, **quiero** verificar si un participante fue acreditado **para** controlar su asistencia.

**Criterios de aceptación:**

1. El organizador puede consultar el estado de acreditación de cualquier inscripto a su evento.
2. Se muestra: nombre del participante, email, estado (acreditado/no acreditado), fecha de acreditación.
3. Si el participante no tiene inscripción aprobada, se indica que no corresponde.

### HU-ACR-03: Ver Estadísticas de Acreditación

**Como** organizador, **quiero** ver estadísticas de asistencia de mi evento **para** conocer el nivel de participación.

**Criterios de aceptación:**

1. Las estadísticas incluyen: total inscriptos, total acreditados, porcentaje de asistencia, acreditados pendientes.
2. El porcentaje se calcula: (acreditados / inscriptos_aprobados) * 100.
3. Si no hay inscriptos aprobados, el porcentaje es 0.
4. Solo el organizador del evento puede ver estas estadísticas.

### HU-CERT-01: Generar Certificados Masivamente

**Como** organizador, **quiero** generar certificados para todos los participantes acreditados de mi evento **para** entregarles constancia formal.

**Criterios de aceptación:**

1. Solo el organizador del evento puede solicitar generación masiva de certificados.
2. El organizador puede elegir el tipo de certificado: asistencia, aprobación, autor, expositor.
3. Por defecto, los certificados se generan solo para participantes acreditados.
4. Si el tipo es "autor" o "expositor", solo se generan para participantes con ese rol en el evento.
5. La generación es asíncrona (se retorna un jobId para consultar estado).
6. Cada certificado tiene un código único de verificación.

### HU-CERT-02: Descargar Mi Certificado

**Como** participante, **quiero** descargar mis certificados emitidos **para** conservarlos como constancia.

**Criterios de aceptación:**

1. El participante puede ver todos sus certificados emitidos.
2. Cada certificado muestra: tipo, evento, fecha de emisión, código de verificación.
3. Se puede descargar el PDF del certificado.
4. El PDF incluye: nombre del participante, nombre del evento, fechas, tipo de certificado, código de verificación.
5. Si no se generaron certificados aún, se indica que no hay certificados disponibles.

### HU-CERT-03: Verificar Validez de Certificado

**Como** cualquier persona, **quiero** verificar que un certificado es válido **para** confirmar su autenticidad.

**Criterios de aceptación:**

1. La verificación es pública (no requiere autenticación).
2. Se ingresa el código de verificación del certificado.
3. Si el código es válido, se muestra: nombre del participante, nombre del evento, tipo de certificado, fecha de emisión.
4. Si el código no existe, se indica que el certificado no es válido.
5. No se muestran datos sensibles (email, etc.) en la verificación pública.

### HU-CERT-04: Ver Mis Certificados

**Como** participante, **quiero** ver un listado de todos mis certificados **para** gestionar mis constancias.

**Criterios de aceptación:**

1. Se muestran todos los certificados emitidos al usuario.
2. Se puede filtrar por tipo de certificado.
3. Se puede filtrar por evento.
4. Cada certificado en la lista muestra: evento, tipo, fecha de emisión, botón de descarga.

---

## 3. Requisitos Funcionales y Reglas de Negocio

### RF-ACR-01: Registro de Asistencia

- **RF-ACR-01.1:** Solo usuarios con rol ORGANIZADOR del evento pueden registrar asistencia.
- **RF-ACR-01.2:** El participante debe tener inscripción con estado APROBADA en el evento.
- **RF-ACR-01.3:** No se permite registrar asistencia duplicada para el mismo participante en el mismo evento.
- **RF-ACR-01.4:** La fecha de acreditación se registra automáticamente en UTC.

### RF-ACR-02: Estadísticas de Acreditación

- **RF-ACR-02.1:** Total inscriptos = count de inscripciones con estado APROBADA.
- **RF-ACR-02.2:** Total acreditados = count de inscripciones con acreditado = true.
- **RF-ACR-02.3:** Porcentaje de asistencia = (acreditados / inscriptos_aprobados) * 100.
- **RF-ACR-02.4:** Si no hay inscriptos aprobados, porcentaje = 0.

### RF-CERT-01: Tipos de Certificado

- **RF-CERT-01.1:** ASISTENCIA: para todos los participantes acreditados.
- **RF-CERT-01.2:** APROBACION: para participantes que aprobaron el evento (requiere evaluación).
- **RF-CERT-01.3:** AUTOR: para participantes con rol de autor en el evento.
- **RF-CERT-01.4:** EXPOSITOR: para participantes con rol de disertante/expositor.

### RF-CERT-02: Elegibilidad para Certificado

- **RF-CERT-02.1:** Para ASISTENCIA: el participante debe estar acreditado.
- **RF-CERT-02.2:** Para APROBACION: el participante debe estar acreditado y tener evaluación aprobada.
- **RF-CERT-02.3:** Para AUTOR/EXPOSITOR: el participante debe tener ese rol asignado en el evento.
- **RF-CERT-02.4:** Si no cumple los requisitos, retornar error CERTIFICATE_NOT_ELIGIBLE (422).

### RF-CERT-03: Generación de Certificados

- **RF-CERT-03.1:** Los certificados se generan en formato PDF usando pdfkit.
- **RF-CERT-03.2:** Cada certificado tiene un código de verificación único (formato: CERT-YYYY-XXXXXX).
- **RF-CERT-03.3:** El código de verificación se genera con UUID truncado + año.
- **RF-CERT-03.4:** Los certificados se almacenan en el sistema de archivos o storage.
- **RF-CERT-03.5:** La generación masiva es asíncrona (job en background).

### RF-CERT-04: Contenido del PDF

- **RF-CERT-04.1:** El PDF incluye: logo del evento/institución (si existe), título del certificado, nombre completo del participante, nombre del evento, fechas del evento, tipo de certificado, código de verificación, fecha de emisión.
- **RF-CERT-04.2:** El diseño es formal y profesional.
- **RF-CERT-04.3:** El nombre del archivo: `certificado_{tipo}_{nombre_participante}.pdf`.

### RF-CERT-05: Verificación Pública

- **RF-CERT-05.1:** Endpoint público (sin autenticación) para verificar certificados.
- **RF-CERT-05.2:** Se busca por código de verificación.
- **RF-CERT-05.3:** Si existe, retorna datos básicos del certificado (sin información sensible).
- **RF-CERT-05.4:** Si no existe, retorna 404 con código CERTIFICATE_NOT_FOUND.

---

## 4. Restricciones Técnicas Específicas

- **RT-ACR-01:** El campo `acreditado` se almacena en el modelo Enrollment, no se crea modelo separado.
- **RT-ACR-02:** La fecha de acreditación se almacena como `accreditedAt` (DateTime, nullable).
- **RT-CERT-01:** Los certificados se generan con pdfkit (mencionado en project.md stack).
- **RT-CERT-02:** El código de verificación debe ser único a nivel global (índice unique en Prisma).
- **RT-CERT-03:** La generación masiva retorna 202 Accepted con jobId (procesamiento asíncrono).
- **RT-CERT-04:** Los PDFs se almacenan en `backend/uploads/certificates/` (configurable via env).

---

## 5. Modelo de Datos

### 5.1. Enum CertificateType

```prisma
enum CertificateType {
  ASISTENCIA
  APROBACION
  AUTOR
  EXPOSITOR
}
```

### 5.2. Modelo Enrollment (campos adicionales)

El modelo Enrollment existente se extiende con campos de acreditación:

```prisma
model Enrollment {
  // ... campos existentes ...
  
  acreditado    Boolean    @default(false) @map("accredited")
  accreditedAt  DateTime?  @map("accredited_at")
  
  certificates  Certificate[]
}
```

### 5.3. Modelo Certificate

```prisma
model Certificate {
  id                Int             @id @default(autoincrement())
  enrollmentId      Int             @map("enrollment_id")
  type              CertificateType @map("type")
  verificationCode  String          @unique @map("verification_code")
  pdfPath           String          @map("pdf_path")
  issuedAt          DateTime        @default(now()) @map("issued_at")

  enrollment        Enrollment      @relation(fields: [enrollmentId], references: [id])

  @@index([enrollmentId])
  @@index([verificationCode])
  @@index([type])
  @@map("certificates")
}
```

### 5.4. Diagrama de Relaciones

```
Event 1 ──── N Enrollment
Enrollment 1 ──── N Certificate
User 1 ──── N Enrollment (como participante)
```

### 5.5. DTOs de Respuesta

#### AccreditationStatusDTO

```typescript
interface AccreditationStatusDTO {
  enrollmentId: number;
  acreditado: boolean;
  accreditedAt: string | null;
  evento: {
    id: number;
    titulo: string;
  };
  participante: {
    id: number;
    nombre: string;
    email: string;
  };
}
```

#### AccreditationStatsDTO

```typescript
interface AccreditationStatsDTO {
  totalInscriptos: number;
  totalAcreditados: number;
  porcentajeAsistencia: number;
  acreditadosPendientes: number;
}
```

#### CertificateDTO

```typescript
interface CertificateDTO {
  id: number;
  type: CertificateType;
  verificationCode: string;
  pdfPath: string;
  issuedAt: string;
  evento: {
    id: number;
    titulo: string;
  };
  participante: {
    id: number;
    nombre: string;
  };
}
```

#### CertificateVerificationDTO

```typescript
interface CertificateVerificationDTO {
  valido: boolean;
  certificado: {
    type: CertificateType;
    issuedAt: string;
    evento: {
      titulo: string;
      tipo: string;
      fechaInicio: string;
    };
    participante: {
      nombre: string;
    };
  } | null;
}
```

#### CertificateGenerationJobDTO

```typescript
interface CertificateGenerationJobDTO {
  jobId: string;
  message: string;
  estimado: string | null;
}
```

---

## 6. Plan de Tareas

### Tarea 1: Definición de modelos y migraciones
- Definir enum CertificateType en Prisma
- Agregar campos `acreditado` y `accreditedAt` al modelo Enrollment
- Definir modelo Certificate con relaciones
- Crear y ejecutar migración

### Tarea 2: Servicio de acreditación
- Implementar `AccreditationService` con método `checkIn(enrollmentId)`
- Implementar verificación de elegibilidad (inscripción aprobada)
- Implementar prevención de doble acreditación
- Implementar `getAccreditationStatus(enrollmentId)`
- Implementar `getAccreditationStats(eventId)`

### Tarea 3: Endpoints de acreditación
- Implementar `POST /accreditation/:enrollmentId/check-in` (registrar asistencia)
- Implementar `GET /accreditation/:enrollmentId` (ver estado)
- Implementar `GET /events/:eventId/accreditation/stats` (estadísticas)

### Tarea 4: Servicio de certificados
- Implementar `CertificateService` con método `generateForEvent(eventId, type, onlyAccredited)`
- Implementar verificación de elegibilidad por tipo de certificado
- Implementar generación de código de verificación único
- Implementar generación de PDF con pdfkit
- Implementar método `verifyCertificate(verificationCode)`

### Tarea 5: Generación de PDF
- Implementar `CertificatePdfGenerator` con pdfkit
- Diseñar layout formal del certificado
- Incluir: nombre, evento, fechas, tipo, código de verificación, fecha de emisión
- Implementar guardado de archivo en `uploads/certificates/`

### Tarea 6: Endpoints de certificados
- Implementar `POST /certificates/generate` (generación masiva)
- Implementar `GET /certificates/:id` (detalle de certificado)
- Implementar `GET /certificates/verify/:code` (verificación pública)
- Implementar `GET /users/me/certificates` (mis certificados)
- Implementar `GET /certificates/:id/download` (descarga PDF)

### Tarea 7: Frontend - Registro de asistencia
- Crear botón/marco de check-in en panel del organizador
- Mostrar lista de inscriptos con indicador de acreditado/no acreditado
- Permitir check-in individual con confirmación visual
- Mostrar estadísticas de acreditación en tarjetas

### Tarea 8: Frontend - Gestión de certificados (organizador)
- Crear sección "Certificados" en panel del organizador
- Formulario para solicitar generación masiva (seleccionar tipo)
- Mostrar progreso de generación (si es asíncrono)
- Mostrar lista de certificados emitidos

### Tarea 9: Frontend - Mis certificados (participante)
- Crear página "Mis Certificados" en perfil del usuario
- Mostrar lista de certificados con tipo, evento, fecha
- Botón de descarga para cada certificado
- Filtros por tipo y evento

### Tarea 10: Frontend - Verificación pública
- Crear página pública "/verificar-certificado"
- Campo para ingresar código de verificación
- Mostrar resultado: válido/no válido con datos del certificado
- Diseño simple y accesible

### Tarea 11: Tests
- Tests unitarios de verificación de elegibilidad por tipo
- Tests unitarios de cálculo de estadísticas de acreditación
- Tests unitarios de generación de código de verificación (unicidad)
- Tests de integración de endpoints de acreditación
- Tests de integración de endpoints de certificados
- Tests de generación de PDF (mock de pdfkit)

### Tarea 12: Documentación Swagger
- Agregar JSDoc swagger a todos los endpoints de acreditación
- Agregar JSDoc swagger a todos los endpoints de certificados

---

## 7. Estrategia de Verificación

### 7.1. Tests Unitarios

| Componente | Qué verificar |
|---|---|
| `checkIn` | Inscripción aprobada, no duplicado, fecha acreditación |
| `getAccreditationStats` | Cálculo correcto de totales y porcentaje |
| `checkCertificateEligibility` | Elegibilidad por tipo (asistencia, aprobación, autor, expositor) |
| `generateVerificationCode` | Formato CERT-YYYY-XXXXXX, unicidad |
| `calculateAttendancePercentage` | (acreditados / aprobados) * 100, maneja división por cero |

### 7.2. Tests de Integración

| Endpoint | Escenario | Resultado esperado |
|---|---|---|
| `POST /accreditation/:id/check-in` | Inscripción aprobada | 200, acreditado = true |
| `POST /accreditation/:id/check-in` | Ya acreditado | 409, ATTENDANCE_ALREADY_RECORDED |
| `POST /accreditation/:id/check-in` | Inscripción no aprobada | 422, no elegible |
| `POST /accreditation/:id/check-in` | No organizador | 403, FORBIDDEN |
| `GET /accreditation/:id` | Organizador del evento | 200, AccreditationStatusDTO |
| `GET /events/:id/accreditation/stats` | Organizador con datos | 200, stats correctos |
| `POST /certificates/generate` | Evento con acreditados | 202, jobId retornado |
| `POST /certificates/generate` | Sin acreditados | 422, CERTIFICATE_NOT_ELIGIBLE |
| `GET /certificates/verify/:code` | Código válido | 200, valido = true con datos |
| `GET /certificates/verify/:code` | Código inválido | 404, CERTIFICATE_NOT_FOUND |
| `GET /users/me/certificates` | Usuario con certificados | 200, lista de certificados |
| `GET /certificates/:id/download` | Certificado existente | 200, application/pdf |

### 7.3. Tests Manuales (QA)

1. Como organizador: registrar asistencia de participante → verificar que aparece como acreditado.
2. Intentar registrar asistencia duplicada → debe fallar con error.
3. Ver estadísticas de acreditación → verificar números correctos.
4. Generar certificados de asistencia masivamente → verificar que se crean PDFs.
5. Descargar certificado como participante → verificar contenido y formato del PDF.
6. Verificar certificado con código válido → debe mostrar datos correctos.
7. Verificar con código inválido → debe indicar que no es válido.
8. Intentar generar certificados de tipo EXPOSITOR para evento sin disertantes → debe fallar.

### 7.4. Criterios de Aceptación del Módulo

- [ ] Todos los endpoints documentados en Swagger
- [ ] Cobertura de tests unitarios >= 70%
- [ ] Registro de asistencia con validación de elegibilidad
- [ ] Prevención de doble acreditación
- [ ] Estadísticas de acreditación calculadas correctamente
- [ ] Generación masiva de certificados funcional
- [ ] PDFs generados con formato profesional y contenido completo
- [ ] Verificación pública de certificados funcionando
- [ ] Listado de certificados del usuario con filtros
- [ ] Frontend responsivo con Bootstrap
