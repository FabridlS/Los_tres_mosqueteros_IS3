# Sistema de Gestión de Eventos Académicos

## 1. Descripción del Sistema

Sistema web para la organización, gestión y administración de eventos académicos (cursos, jornadas, congresos, charlas). Permite a los usuarios registrarse, gestionar eventos, inscribir participantes, generar certificados, y obtener informes post-evento.

### Módulos del Sistema

| Módulo | Descripción | Archivo de Spec |
|--------|-------------|-----------------|
| **Autenticación y Usuarios** | Registro, login, gestión de perfiles y roles | `specs/auth-users.md` |
| **Gestión de Eventos** | CRUD de eventos, configuración de cupos y fechas | `specs/events.md` |
| **Inscripciones** | Inscripción de participantes, control de cupo y fechas límite | `specs/registrations.md` |
| **Acreditación** | Registro y verificación de asistencia | `specs/accreditation.md` |
| **Encuestas y Comentarios** | Feedback y satisfacción post-evento | `specs/surveys.md` |
| **Certificados** | Generación de certificados (asistencia, aprobación, expositor) | `specs/certificates.md` |
| **Informes y Reportes** | Generación de informes y agendas del evento | `specs/reports.md` |

---

## 2. Stack Tecnológico

| Capa | Tecnología | Versión |
|------|------------|---------|
| **Frontend** | React + TypeScript | Latest |
| **UI Framework** | TailwindCSS + shadcn/ui | Latest |
| **Backend** | Node.js + Express / NestJS | LTS |
| **Base de datos** | PostgreSQL | 16+ |
| **ORM** | Prisma | Latest |
| **Autenticación** | JWT + bcrypt | Latest |
| **Generación PDF** | PDFKit / React-PDF | Latest |
| **Testing** | Jest + React Testing Library | Latest |
| **Documentación API** | Swagger / OpenAPI | 3.0 |

---

## 3. Estructura del Repositorio

```
├── backend/
│   ├── src/
│   │   ├── modules/
│   │   │   ├── auth/
│   │   │   ├── users/
│   │   │   ├── events/
│   │   │   ├── registrations/
│   │   │   ├── accreditation/
│   │   │   ├── surveys/
│   │   │   ├── certificates/
│   │   │   └── reports/
│   │   ├── common/
│   │   │   ├── filters/
│   │   │   ├── interceptors/
│   │   │   ├── decorators/
│   │   │   └── utils/
│   │   ├── shared/
│   │   │   ├── models/
│   │   │   └── types/
│   │   └── main.ts
│   ├── prisma/
│   │   └── schema.prisma
│   ├── test/
│   └── package.json
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── hooks/
│   │   ├── services/
│   │   ├── types/
│   │   └── App.tsx
│   └── package.json
├── specs/
│   ├── auth-users.md
│   ├── events.md
│   ├── registrations.md
│   ├── accreditation.md
│   ├── surveys.md
│   ├── certificates.md
│   └── reports.md
├── contracts.md
└── project.md
```

---

## 4. Convenciones de Código

### Backend
- **Naming**: camelCase para variables/funciones, PascalCase para clases, UPPER_SNAKE_CASE para constantes
- **Arquitectura**: Modular por dominio (auth, events, registrations...)
- **DTOs**: Validación con class-validator
- **Mensajes**: Inglés para código y comentarios
- **Git commits**: Conventional Commits (`feat:`, `fix:`, `refactor:`, `docs:`)

### Frontend
- **Naming**: PascalCase para componentes, camelCase para funciones/variables
- **Estructura**: Feature-based folder organization
- **Estado**: React Context / Zustand según complejidad
- **Estilos**: TailwindCSS utility-first
- **Accesibilidad**: WCAG 2.1 AA mínimo

### General
- ESLint + Prettier obligatorio
- Pull requests con descripción y contexto
- Código en español para documentación, inglés para código

---

## 5. Formato Estándar de Respuestas HTTP

### Success Response
```json
{
  "success": true,
  "data": { ... },
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 150
  }
}
```

### Error Response
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Descripción legible del error",
    "details": [
      { "field": "email", "message": "Formato inválido" }
    ]
  }
}
```

### Códigos HTTP
| Código | Uso |
|--------|-----|
| `200` | GET exitoso, PUT exitoso |
| `201` | Recurso creado |
| `204` | DELETE exitoso |
| `400` | Datos inválidos |
| `401` | No autenticado |
| `403` | Sin permisos |
| `404` | Recurso no encontrado |
| `409` | Conflicto (ej. cupo lleno) |
| `422` | Regla de negocio violada |
| `500` | Error interno |

---

## 6. Modelos de Datos Compartidos

### Usuario
| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | UUID | Identificador único |
| `nombre` | String | Nombre completo |
| `email` | String | Email único |
| `password` | String | Hash de contraseña |
| `rol` | Enum | `ADMIN`, `ORGANIZADOR`, `DISERTANTE`, `PARTICIPANTE` |
| `createdAt` | DateTime | Fecha de registro |
| `updatedAt` | DateTime | Última actualización |

### Evento
| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | UUID | Identificador único |
| `titulo` | String | Nombre del evento |
| `descripcion` | Text | Descripción detallada |
| `tipo` | Enum | `CURSO`, `JORNADA`, `CONGRESO`, `CHARLA` |
| `fechaInicio` | DateTime | Fecha y hora de inicio |
| `fechaFin` | DateTime | Fecha y hora de fin |
| `cupoMinimo` | Int | Cupo mínimo requerido |
| `cupoMaximo` | Int | Cupo máximo permitido |
| `fechaLimiteInscripcion` | DateTime | Fecha límite para inscribirse |
| `estado` | Enum | `BORRADOR`, `PUBLICADO`, `EN_CURSO`, `FINALIZADO`, `CANCELADO` |
| `organizadorId` | UUID | FK al usuario organizador |
| `createdAt` | DateTime | Fecha de creación |
| `updatedAt` | DateTime | Última actualización |

### Inscripción
| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | UUID | Identificador único |
| `usuarioId` | UUID | FK al usuario inscripto |
| `eventoId` | UUID | FK al evento |
| `estado` | Enum | `PENDIENTE`, `CONFIRMADA`, `CANCELADA` |
| `fechaInscripcion` | DateTime | Fecha de inscripción |
| `acreditado` | Boolean | Indicador de acreditación |
| `createdAt` | DateTime | Fecha de registro |

### Certificado
| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | UUID | Identificador único |
| `inscripcionId` | UUID | FK a la inscripción |
| `tipo` | Enum | `ASISTENCIA`, `APROBACION`, `AUTOR`, `EXPOSITOR` |
| `urlPdf` | String | Ruta/URL del PDF generado |
| `codigoVerificacion` | String | Código único de verificación |
| `emitidoAt` | DateTime | Fecha de emisión |

### Encuesta
| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | UUID | Identificador único |
| `eventoId` | UUID | FK al evento |
| `usuarioId` | UUID | FK al usuario que responde |
| `calificacion` | Int | Puntuación (1-5) |
| `comentario` | Text | Comentario opcional |
| `respondidoAt` | DateTime | Fecha de respuesta |

---

## 7. Roles del Sistema

| Rol | Permisos |
|-----|----------|
| **Participante** | Registro, ver eventos públicos, inscribirse a eventos, descargar certificados propios, completar encuestas |
| **Disertante** | Todo lo de Participante + ver agenda de eventos donde expone, subir material de presentación |
| **Organizador** | CRUD de eventos propios, gestionar inscripciones, acreditar participantes, ver informes, generar certificados, gestionar disertantes |
| **Admin** | Acceso total al sistema, gestión de usuarios, moderación, estadísticas globales |

---

## 8. Dependencias Aprobadas

### Backend
| Paquete | Uso |
|---------|-----|
| `express` / `@nestjs/core` | Framework backend |
| `@prisma/client` | ORM y acceso a datos |
| `prisma` | CLI de migraciones |
| `jsonwebtoken` | Autenticación JWT |
| `bcrypt` | Hash de contraseñas |
| `class-validator` | Validación de DTOs |
| `@pdf-lib` | Generación de PDFs |
| `jest` | Testing |
| `swagger-jsdoc` | Documentación API |

### Frontend
| Paquete | Uso |
|---------|-----|
| `react` | Librería UI |
| `react-router-dom` | Routing |
| `tailwindcss` | Framework de estilos |
| `zustand` | Gestión de estado global |
| `axios` | Cliente HTTP |
| `react-hook-form` | Manejo de formularios |
| `@testing-library/react` | Testing de componentes |
