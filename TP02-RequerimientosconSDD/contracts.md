# Contratos de API Globales

## 1. Convenciones Generales

### 1.1. Base URL

Todos los endpoints de la API siguen el patrón:

```
{BASE_URL}/api/v1/{resource}
```

Donde `BASE_URL` es `http://localhost:3001` en desarrollo.

### 1.2. Content-Type

- Todas las requests con body deben enviar header: `Content-Type: application/json`
- Todas las responses del servidor son `application/json`
- Excepción: endpoints de descarga de certificados retornan `application/pdf`

### 1.3. Autenticación

Los endpoints protegidos requieren el header:

```
Authorization: Bearer <access_token>
```

---

## 2. Esquema de Respuestas

### 2.1. Respuesta Exitosa (recurso único)

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "Ejemplo",
    "createdAt": "2026-05-01T10:00:00.000Z",
    "updatedAt": "2026-05-01T10:00:00.000Z"
  }
}
```

### 2.2. Respuesta Exitosa (colección con paginación)

```json
{
  "success": true,
  "data": [
    { "id": 1, "name": "Evento 1" },
    { "id": 2, "name": "Evento 2" }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 45,
    "totalPages": 3
  }
}
```

### 2.3. Respuesta de Error de Validación (400)

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "La solicitud contiene campos inválidos",
    "details": [
      { "field": "email", "message": "Formato de email inválido" },
      { "field": "password", "message": "Mínimo 8 caracteres requerido" }
    ]
  }
}
```

### 2.4. Respuesta de Error de Negocio (409, 422)

```json
{
  "success": false,
  "error": {
    "code": "ENROLLMENT_CLOSED",
    "message": "El período de inscripción ha finalizado",
    "details": []
  }
}
```

### 2.5. Respuesta de Error No Autenticado (401)

```json
{
  "success": false,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Token de acceso inválido o expirado",
    "details": []
  }
}
```

### 2.6. Respuesta de Error de Permisos (403)

```json
{
  "success": false,
  "error": {
    "code": "FORBIDDEN",
    "message": "No tiene permisos para realizar esta acción",
    "details": []
  }
}
```

### 2.7. Respuesta de Recurso No Encontrado (404)

```json
{
  "success": false,
  "error": {
    "code": "NOT_FOUND",
    "message": "El recurso solicitado no existe",
    "details": []
  }
}
```

### 2.8. Respuesta de Error Interno (500)

```json
{
  "success": false,
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "Ha ocurrido un error interno del servidor",
    "details": []
  }
}
```

---

## 3. Códigos de Error de Negocio

Los siguientes códigos de error (`error.code`) pueden ser retornados por la API:

### Autenticación y Usuarios

| Código | HTTP | Descripción |
|---|---|---|
| `INVALID_CREDENTIALS` | 401 | Email o contraseña incorrectos |
| `EMAIL_ALREADY_EXISTS` | 409 | El email ya está registrado |
| `TOKEN_EXPIRED` | 401 | El JWT ha expirado |
| `INVALID_TOKEN` | 401 | El JWT es inválido |
| `REFRESH_TOKEN_INVALID` | 401 | El refresh token no es válido |
| `USER_NOT_FOUND` | 404 | El usuario no existe |

### Eventos

| Código | HTTP | Descripción |
|---|---|---|
| `EVENT_NOT_FOUND` | 404 | El evento no existe |
| `EVENT_NOT_APPROVED` | 403 | El evento no ha sido aprobado por un admin |
| `EVENT_ALREADY_APPROVED` | 409 | El evento ya está aprobado |
| `EVENT_DATE_IN_PAST` | 422 | La fecha del evento es anterior a la fecha actual |
| `EVENT_NOT_ORGANIZER` | 403 | El usuario no es el organizador del evento |

### Inscripciones

| Código | HTTP | Descripción |
|---|---|---|
| `ENROLLMENT_ALREADY_EXISTS` | 409 | El usuario ya está inscripto al evento |
| `ENROLLMENT_CLOSED` | 422 | El período de inscripción ha cerrado |
| `ENROLLMENT_NOT_FOUND` | 404 | La inscripción no existe |
| `CAPACITY_REACHED` | 409 | Se alcanzó el cupo máximo del evento |
| `ENROLLMENT_NOT_PENDING` | 422 | La inscripción no está en estado pendiente |
| `ENROLLMENT_NOT_APPROVED` | 422 | Se requiere aprobación del organizador |

### Acreditación y Certificados

| Código | HTTP | Descripción |
|---|---|---|
| `ATTENDANCE_ALREADY_RECORDED` | 409 | Ya se registró asistencia para este usuario |
| `CERTIFICATE_NOT_ELIGIBLE` | 422 | El usuario no cumple los requisitos para el certificado |
| `CERTIFICATE_NOT_FOUND` | 404 | El certificado no existe |

### Encuestas y Comentarios

| Código | HTTP | Descripción |
|---|---|---|
| `SURVEY_NOT_AVAILABLE` | 422 | La encuesta aún no está disponible |
| `SURVEY_ALREADY_RESPONDED` | 409 | El usuario ya respondió esta encuesta |
| `SURVEY_NOT_FOUND` | 404 | La encuesta no existe |
| `COMMENT_NOT_ALLOWED` | 422 | No se permiten comentarios en este evento |

### Informes

| Código | HTTP | Descripción |
|---|---|---|
| `REPORT_NOT_AUTHORIZED` | 403 | No tiene permisos para ver el informe |
| `EVENT_HAS_NO_SESSIONS` | 404 | El evento no tiene sesiones para generar agenda |

---

## 4. Autenticación JWT

### 4.1. Flow de Autenticación

```
1. POST /auth/login → retorna { accessToken, refreshToken }
2. accessToken se usa en header Authorization para requests protegidas
3. Cuando accessToken expira (15 min), se llama a POST /auth/refresh
4. POST /auth/refresh → retorna nuevo { accessToken, refreshToken }
5. POST /auth/logout → invalida refresh token
```

### 4.2. Estructura del Access Token (payload)

```json
{
  "sub": 1,
  "email": "usuario@email.com",
  "roles": ["participant"],
  "iat": 1714579200,
  "exp": 1714580100
}
```

### 4.3. Estructura del Refresh Token (payload)

```json
{
  "sub": 1,
  "jti": "unique-token-id",
  "iat": 1714579200,
  "exp": 1715184000
}
```

### 4.4. Headers requeridos por tipo de endpoint

| Tipo de endpoint | Headers requeridos |
|---|---|
| Públicos (login, register, listado eventos) | Ninguno especial |
| Protegidos (perfil, inscripciones, etc.) | `Authorization: Bearer <token>` |
| Admin (gestión de usuarios, aprobación eventos) | `Authorization: Bearer <token>` + rol `admin` |
| Descarga de certificados | `Authorization: Bearer <token>` (retorna PDF) |

---

## 5. Paginación

### 5.1. Parámetros de Query

| Parámetro | Tipo | Default | Max | Descripción |
|---|---|---|---|---|
| `page` | number | 1 | - | Número de página |
| `limit` | number | 20 | 100 | Cantidad de resultados por página |

### 5.2. Ejemplo de Request

```
GET /api/v1/events?page=2&limit=10
```

### 5.3. Validación

- Si `page < 1`, retornar 400
- Si `limit < 1` o `limit > 100`, retornar 400
- Si los parámetros no son numéricos, retornar 400

---

## 6. Filtrado y Ordenamiento

### 6.1. Parámetros Estándar

| Parámetro | Tipo | Descripción |
|---|---|---|
| `sort` | string | Campo por el cual ordenar (default: `createdAt`) |
| `order` | string | Dirección del orden: `asc` o `desc` (default: `desc`) |
| `search` | string | Búsqueda textual (aplica según el recurso) |

### 6.2. Filtros específicos por recurso

Cada recurso define sus propios filtros en su spec. Ejemplo para eventos:

```
GET /api/v1/events?status=approved&type=congress&future=true
```

---

## 7. Validación de Request Body

### 7.1. Reglas Generales

- Todos los campos marcados como requeridos deben estar presentes
- Los strings no pueden estar vacíos (solo whitespace)
- Los emails deben tener formato válido
- Las fechas deben estar en formato ISO 8601 (`YYYY-MM-DDTHH:mm:ss.sssZ`)
- Los números deben estar dentro del rango especificado

### 7.2. Campos de Fecha

- Todas las fechas se almacenan en UTC
- Las fechas se envían y reciben en formato ISO 8601
- Ejemplo: `"2026-05-06T23:00:00.000Z"`

### 7.3. Validación en el Backend

- Se usa Zod para validación de schemas
- Los errores de validación retornan 400 con `details` descriptivos
- La validación ocurre antes de cualquier lógica de negocio

---

## 8. Upload de Archivos

### 8.1. Endpoints que aceptan archivos

- No se requiere upload de archivos en la versión inicial del proyecto
- Si se implementa en el futuro, usar `multipart/form-data`
- Límite máximo de archivo: 5MB
- Tipos permitidos: `.pdf`, `.jpg`, `.png`

---

## 9. CORS

### 9.1. Configuración de Desarrollo

```
Access-Control-Allow-Origin: http://localhost:5173
Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE, OPTIONS
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Allow-Credentials: true
```

### 9.2. Configuración de Producción

- El origen debe configurarse via variable de entorno `CORS_ORIGIN`
- No usar wildcard `*` en producción

---

## 10. Rate Limiting

| Endpoint | Límite | Ventana |
|---|---|---|
| `/auth/login` | 10 requests | 15 minutos |
| `/auth/register` | 5 requests | 15 minutos |
| `/auth/refresh` | 10 requests | 15 minutos |
| Resto de endpoints | 100 requests | 15 minutos |

### Respuesta de Rate Limit Excedido (429)

```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Demasiadas solicitudes. Intente nuevamente más tarde",
    "details": []
  }
}
```

Headers incluidos:
```
Retry-After: 900
X-RateLimit-Limit: 10
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1714579200
```

---

## 11. Swagger / OpenAPI

### 11.1. Configuración

- La documentación se genera automáticamente con `swagger-jsdoc`
- Se sirve con `swagger-ui-express` en `/api-docs`
- El archivo de configuración de Swagger se ubica en `backend/src/config/swagger.ts`

### 11.2. Formato de JSDoc para Endpoints

```typescript
/**
 * @swagger
 * /api/v1/events:
 *   get:
 *     summary: Listar todos los eventos públicos
 *     tags: [Events]
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *           default: 1
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           default: 20
 *     responses:
 *       200:
 *         description: Lista de eventos
 *       401:
 *         description: No autenticado
 */
```

### 11.3. Tags de Swagger

Los tags corresponden a los módulos del proyecto:

- `Auth` - Autenticación y gestión de usuarios
- `Events` - Gestión de eventos y catálogo
- `Enrollments` - Inscripciones
- `Attendance` - Acreditación
- `Certificates` - Certificados
- `Surveys` - Encuestas
- `Comments` - Comentarios
- `Reports` - Informes y agenda

---

## 12. Versionado de API

- La API está versionada en la URL: `/api/v1/`
- Cambios breaking requieren nueva versión: `/api/v2/`
- Cambios no-breaking (nuevos campos opcionales) no requieren nueva versión
- Las versiones deprecated se mantienen por al menos 1 versión
