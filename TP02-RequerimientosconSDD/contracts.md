# Contratos de API - Sistema de Gestión de Eventos Académicos

Base URL: `/api/v1`

Autenticación: Bearer Token JWT en header `Authorization`

---

## 1. Autenticación y Usuarios

### POST `/auth/register`
Registro de nuevo usuario.

**Request Body:**
```json
{
  "nombre": "string",
  "email": "string",
  "password": "string",
  "rol": "PARTICIPANTE"
}
```

**Response 201:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "nombre": "string",
      "email": "string",
      "rol": "PARTICIPANTE",
      "createdAt": "2026-05-01T10:00:00Z"
    },
    "accessToken": "string",
    "expiresIn": 3600
  }
}
```

---

### POST `/auth/login`
Inicio de sesión.

**Request Body:**
```json
{
  "email": "string",
  "password": "string"
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "accessToken": "string",
    "expiresIn": 3600,
    "user": {
      "id": "uuid",
      "nombre": "string",
      "email": "string",
      "rol": "string"
    }
  }
}
```

---

### POST `/auth/refresh`
Renovar token de acceso.

**Request Body:**
```json
{
  "refreshToken": "string"
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "accessToken": "string",
    "expiresIn": 3600
  }
}
```

---

### GET `/users/me`
Obtener perfil del usuario autenticado.

**Headers:** `Authorization: Bearer <token>`

**Response 200:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "nombre": "string",
    "email": "string",
    "rol": "string",
    "createdAt": "2026-05-01T10:00:00Z"
  }
}
```

---

### PUT `/users/me`
Actualizar perfil del usuario autenticado.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "nombre": "string",
  "email": "string"
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "nombre": "string",
    "email": "string",
    "rol": "string"
  }
}
```

---

### GET `/users` (Admin/Organizador)
Listar usuarios del sistema.

**Headers:** `Authorization: Bearer <token>`

**Query Params:**
- `page` (int, default: 1)
- `limit` (int, default: 20)
- `rol` (string, optional)

**Response 200:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "nombre": "string",
      "email": "string",
      "rol": "string",
      "createdAt": "2026-05-01T10:00:00Z"
    }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 50
  }
}
```

---

## 2. Eventos

### GET `/events`
Listar eventos públicos.

**Query Params:**
- `page` (int, default: 1)
- `limit` (int, default: 20)
- `tipo` (string, optional: CURSO, JORNADA, CONGRESO, CHARLA)
- `estado` (string, optional: PUBLICADO, EN_CURSO, FINALIZADO)
- `futuros` (boolean, default: false) - solo eventos futuros
- `pasados` (boolean, default: false) - solo eventos pasados
- `search` (string, optional) - búsqueda por título/descripción

**Response 200:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "titulo": "string",
      "descripcion": "string",
      "tipo": "CONGRESO",
      "fechaInicio": "2026-06-15T09:00:00Z",
      "fechaFin": "2026-06-17T18:00:00Z",
      "cupoMinimo": 20,
      "cupoMaximo": 200,
      "fechaLimiteInscripcion": "2026-06-01T23:59:59Z",
      "estado": "PUBLICADO",
      "inscripcionesActuales": 45,
      "organizador": {
        "id": "uuid",
        "nombre": "string"
      }
    }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 15
  }
}
```

---

### GET `/events/:id`
Obtener detalle de un evento.

**Response 200:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "titulo": "string",
    "descripcion": "string",
    "tipo": "CONGRESO",
    "fechaInicio": "2026-06-15T09:00:00Z",
    "fechaFin": "2026-06-17T18:00:00Z",
    "cupoMinimo": 20,
    "cupoMaximo": 200,
    "fechaLimiteInscripcion": "2026-06-01T23:59:59Z",
    "estado": "PUBLICADO",
    "inscripcionesActuales": 45,
    "organizador": {
      "id": "uuid",
      "nombre": "string",
      "email": "string"
    },
    "agenda": [
      {
        "horaInicio": "09:00",
        "horaFin": "10:30",
        "titulo": "string",
        "disertante": "string",
        "descripcion": "string"
      }
    ]
  }
}
```

---

### POST `/events` (Organizador/Admin)
Crear un nuevo evento.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "titulo": "string",
  "descripcion": "string",
  "tipo": "CURSO",
  "fechaInicio": "2026-06-15T09:00:00Z",
  "fechaFin": "2026-06-17T18:00:00Z",
  "cupoMinimo": 20,
  "cupoMaximo": 200,
  "fechaLimiteInscripcion": "2026-06-01T23:59:59Z",
  "estado": "BORRADOR"
}
```

**Response 201:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "titulo": "string",
    "descripcion": "string",
    "tipo": "CURSO",
    "fechaInicio": "2026-06-15T09:00:00Z",
    "fechaFin": "2026-06-17T18:00:00Z",
    "cupoMinimo": 20,
    "cupoMaximo": 200,
    "fechaLimiteInscripcion": "2026-06-01T23:59:59Z",
    "estado": "BORRADOR",
    "organizadorId": "uuid",
    "createdAt": "2026-05-01T10:00:00Z"
  }
}
```

---

### PUT `/events/:id` (Organizador/Admin)
Actualizar un evento existente.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "titulo": "string",
  "descripcion": "string",
  "tipo": "JORNADA",
  "fechaInicio": "2026-06-15T09:00:00Z",
  "fechaFin": "2026-06-17T18:00:00Z",
  "cupoMinimo": 20,
  "cupoMaximo": 200,
  "fechaLimiteInscripcion": "2026-06-01T23:59:59Z",
  "estado": "PUBLICADO"
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "titulo": "string",
    "estado": "PUBLICADO"
  }
}
```

---

### DELETE `/events/:id` (Organizador/Admin)
Eliminar un evento.

**Headers:** `Authorization: Bearer <token>`

**Response 204:** Sin contenido

---

## 3. Inscripciones

### POST `/events/:eventId/registrations`
Inscribirse a un evento.

**Headers:** `Authorization: Bearer <token>`

**Response 201:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "eventoId": "uuid",
    "usuarioId": "uuid",
    "estado": "CONFIRMADA",
    "fechaInscripcion": "2026-05-01T10:00:00Z"
  }
}
```

**Error 409 - Cupo lleno:**
```json
{
  "success": false,
  "error": {
    "code": "EVENT_CAPACITY_FULL",
    "message": "El evento ha alcanzado su cupo máximo"
  }
}
```

**Error 422 - Fecha límite pasada:**
```json
{
  "success": false,
  "error": {
    "code": "REGISTRATION_DEADLINE_PASSED",
    "message": "La fecha límite de inscripción ha pasado"
  }
}
```

---

### GET `/events/:eventId/registrations` (Organizador)
Listar inscripciones de un evento.

**Headers:** `Authorization: Bearer <token>`

**Query Params:**
- `page` (int, default: 1)
- `limit` (int, default: 20)
- `estado` (string, optional: PENDIENTE, CONFIRMADA, CANCELADA)

**Response 200:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "usuario": {
        "id": "uuid",
        "nombre": "string",
        "email": "string"
      },
      "estado": "CONFIRMADA",
      "fechaInscripcion": "2026-05-01T10:00:00Z",
      "acreditado": false
    }
  ],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 45
  }
}
```

---

### GET `/users/me/registrations`
Listar mis inscripciones.

**Headers:** `Authorization: Bearer <token>`

**Response 200:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "evento": {
        "id": "uuid",
        "titulo": "string",
        "tipo": "CONGRESO",
        "fechaInicio": "2026-06-15T09:00:00Z"
      },
      "estado": "CONFIRMADA",
      "fechaInscripcion": "2026-05-01T10:00:00Z",
      "acreditado": true
    }
  ]
}
```

---

### PUT `/events/:eventId/registrations/:registrationId` (Organizador)
Actualizar estado de una inscripción.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "estado": "CONFIRMADA"
}
```

**Response 200:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "estado": "CONFIRMADA"
  }
}
```

---

### DELETE `/events/:eventId/registrations/:registrationId`
Cancelar una inscripción.

**Headers:** `Authorization: Bearer <token>`

**Response 204:** Sin contenido

---

### POST `/events/:eventId/registrations/manual` (Organizador)
Inscribir manualmente a un participante.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "email": "string",
  "nombre": "string"
}
```

**Response 201:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "usuario": {
      "id": "uuid",
      "nombre": "string",
      "email": "string"
    },
    "estado": "CONFIRMADA",
    "fechaInscripcion": "2026-05-01T10:00:00Z"
  }
}
```

---

## 4. Acreditación

### POST `/accreditation/:registrationId/check-in` (Organizador)
Registrar acreditación de un participante.

**Headers:** `Authorization: Bearer <token>`

**Response 200:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "acreditado": true,
    "fechaAcreditacion": "2026-06-15T09:00:00Z"
  }
}
```

---

### GET `/accreditation/:registrationId` (Organizador)
Ver estado de acreditación.

**Headers:** `Authorization: Bearer <token>`

**Response 200:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "acreditado": true,
    "fechaAcreditacion": "2026-06-15T09:00:00Z",
    "evento": {
      "id": "uuid",
      "titulo": "string"
    },
    "participante": {
      "id": "uuid",
      "nombre": "string",
      "email": "string"
    }
  }
}
```

---

### GET `/events/:eventId/accreditation/stats` (Organizador)
Estadísticas de acreditación de un evento.

**Headers:** `Authorization: Bearer <token>`

**Response 200:**
```json
{
  "success": true,
  "data": {
    "totalInscriptos": 150,
    "totalAcreditados": 120,
    "porcentajeAsistencia": 80.0,
    "acreditadosPendientes": 30
  }
}
```

---

## 5. Encuestas y Comentarios

### GET `/events/:eventId/survey`
Obtener encuesta de un evento.

**Headers:** `Authorization: Bearer <token>`

**Response 200:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "eventoId": "uuid",
    "preguntas": [
      {
        "id": "uuid",
        "texto": "string",
        "tipo": "CALIFICACION"
      }
    ]
  }
}
```

---

### POST `/events/:eventId/survey/responses`
Enviar respuesta de encuesta.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "calificacion": 4,
  "comentario": "Muy buen evento, organizado y contenido relevante"
}
```

**Response 201:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "calificacion": 4,
    "comentario": "string",
    "respondidoAt": "2026-06-18T10:00:00Z"
  }
}
```

**Error 409 - Ya respondida:**
```json
{
  "success": false,
  "error": {
    "code": "SURVEY_ALREADY_RESPONDED",
    "message": "Ya has respondido la encuesta de este evento"
  }
}
```

---

### GET `/events/:eventId/survey/results` (Organizador)
Obtener resultados de encuesta.

**Headers:** `Authorization: Bearer <token>`

**Response 200:**
```json
{
  "success": true,
  "data": {
    "totalRespuestas": 120,
    "calificacionPromedio": 4.2,
    "distribucion": {
      "1": 5,
      "2": 10,
      "3": 15,
      "4": 40,
      "5": 50
    },
    "comentarios": [
      {
        "id": "uuid",
        "comentario": "string",
        "respondidoAt": "2026-06-18T10:00:00Z"
      }
    ]
  }
}
```

---

## 6. Certificados

### POST `/certificates/generate` (Organizador)
Generar certificados para un evento.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "eventoId": "uuid",
  "tipo": "ASISTENCIA",
  "soloAcreditados": true
}
```

**Response 202:**
```json
{
  "success": true,
  "data": {
    "jobId": "uuid",
    "message": "Generación de certificados en proceso",
    "estimado": "2026-06-20T12:00:00Z"
  }
}
```

---

### GET `/certificates/:id`
Obtener detalle de un certificado.

**Headers:** `Authorization: Bearer <token>`

**Response 200:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "tipo": "ASISTENCIA",
    "urlPdf": "/downloads/certificates/uuid.pdf",
    "codigoVerificacion": "CERT-2026-ABC123",
    "emitidoAt": "2026-06-20T12:00:00Z",
    "evento": {
      "id": "uuid",
      "titulo": "string"
    },
    "participante": {
      "id": "uuid",
      "nombre": "string"
    }
  }
}
```

---

### GET `/certificates/verify/:codigo`
Verificar validez de un certificado (público).

**Response 200:**
```json
{
  "success": true,
  "data": {
    "valido": true,
    "certificado": {
      "id": "uuid",
      "tipo": "ASISTENCIA",
      "emitidoAt": "2026-06-20T12:00:00Z",
      "evento": {
        "titulo": "string",
        "tipo": "CONGRESO",
        "fechaInicio": "2026-06-15T09:00:00Z"
      },
      "participante": {
        "nombre": "string"
      }
    }
  }
}
```

**Error 404 - Certificado no encontrado:**
```json
{
  "success": false,
  "error": {
    "code": "CERTIFICATE_NOT_FOUND",
    "message": "El código de verificación no corresponde a ningún certificado"
  }
}
```

---

### GET `/users/me/certificates`
Listar mis certificados.

**Headers:** `Authorization: Bearer <token>`

**Response 200:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "tipo": "ASISTENCIA",
      "urlPdf": "/downloads/certificates/uuid.pdf",
      "codigoVerificacion": "CERT-2026-ABC123",
      "emitidoAt": "2026-06-20T12:00:00Z",
      "evento": {
        "id": "uuid",
        "titulo": "string"
      }
    }
  ]
}
```

---

## 7. Informes y Reportes

### GET `/events/:eventId/reports/summary` (Organizador)
Obtener resumen de un evento.

**Headers:** `Authorization: Bearer <token>`

**Response 200:**
```json
{
  "success": true,
  "data": {
    "evento": {
      "id": "uuid",
      "titulo": "string",
      "tipo": "CONGRESO",
      "fechaInicio": "2026-06-15T09:00:00Z",
      "fechaFin": "2026-06-17T18:00:00Z"
    },
    "estadisticas": {
      "totalInscriptos": 150,
      "totalAcreditados": 120,
      "porcentajeAsistencia": 80.0,
      "calificacionPromedio": 4.2,
      "certificadosGenerados": 120
    },
    "inscripcionesPorEstado": {
      "CONFIRMADA": 130,
      "CANCELADA": 15,
      "PENDIENTE": 5
    }
  }
}
```

---

### GET `/events/:eventId/agenda`
Obtener agenda del evento.

**Response 200:**
```json
{
  "success": true,
  "data": {
    "evento": {
      "id": "uuid",
      "titulo": "string"
    },
    "agenda": [
      {
        "fecha": "2026-06-15",
        "sesiones": [
          {
            "horaInicio": "09:00",
            "horaFin": "10:30",
            "titulo": "string",
            "disertante": "string",
            "descripcion": "string",
            "lugar": "string"
          }
        ]
      }
    ]
  }
}
```

---

### POST `/events/:eventId/reports/export` (Organizador)
Exportar informe en PDF/Excel.

**Headers:** `Authorization: Bearer <token>`

**Request Body:**
```json
{
  "formato": "PDF",
  "incluir": ["inscripciones", "acreditaciones", "encuestas"]
}
```

**Response 200:** (PDF como archivo binario)
```
Content-Type: application/pdf
Content-Disposition: attachment; filename="informe-evento.pdf"
```

---

## Errores Estándar

### 400 - Bad Request
```json
{
  "success": false,
  "error": {
    "code": "INVALID_REQUEST",
    "message": "Datos de entrada inválidos",
    "details": [
      { "field": "email", "message": "Formato de email inválido" },
      { "field": "password", "message": "Mínimo 8 caracteres" }
    ]
  }
}
```

### 401 - Unauthorized
```json
{
  "success": false,
  "error": {
    "code": "UNAUTHORIZED",
    "message": "Token de acceso inválido o expirado"
  }
}
```

### 403 - Forbidden
```json
{
  "success": false,
  "error": {
    "code": "FORBIDDEN",
    "message": "No tienes permisos para acceder a este recurso"
  }
}
```

### 404 - Not Found
```json
{
  "success": false,
  "error": {
    "code": "NOT_FOUND",
    "message": "El recurso solicitado no existe"
  }
}
```

### 409 - Conflict
```json
{
  "success": false,
  "error": {
    "code": "CONFLICT",
    "message": "Conflicto con el estado actual del recurso"
  }
}
```

### 422 - Unprocessable Entity
```json
{
  "success": false,
  "error": {
    "code": "BUSINESS_RULE_VIOLATION",
    "message": "Se violó una regla de negocio"
  }
}
```

### 500 - Internal Server Error
```json
{
  "success": false,
  "error": {
    "code": "INTERNAL_ERROR",
    "message": "Error interno del servidor"
  }
}
```
