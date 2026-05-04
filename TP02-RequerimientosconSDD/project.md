# Proyecto: Gestión de Eventos Académicos

## 1. Objetivo y Contexto

Desarrollar una aplicación web para la organización y gestión de eventos académicos (cursos, jornadas, congresos, charlas, etc.) que permita a grupos de personas organizar, administrar y dar seguimiento a eventos de tipo educativo.

La plataforma facilita la inscripción de participantes, la gestión de roles, la acreditación, la generación de certificados, la recolección de feedback post-evento y la generación de informes.

### Equipo de Desarrollo

- DA ROSA, Jesica M.
- de LOS SANTOS, Fabrizzio R.
- GRABOVIESKI, Matias A.

### Materia

Ingeniería del Software 3

---

## 2. Stack Tecnológico

### Frontend

| Tecnología | Versión | Propósito |
|---|---|---|
| React | 18+ | Biblioteca de UI |
| TypeScript | 5+ | Tipado estático |
| Vite | 5+ | Build tool y dev server |
| Bootstrap / React-Bootstrap | 5+ | Componentes UI y estilos |
| React Router | 6+ | Enrutamiento del lado cliente |
| Axios | 1+ | Cliente HTTP para API |
| React Hook Form + Zod | latest | Formularios y validación |
| Zustand o Context API | - | Gestión de estado global |

### Backend

| Tecnología | Versión | Propósito |
|---|---|---|
| Node.js | 20+ LTS | Runtime de JavaScript |
| Express | 4+ | Framework web |
| TypeScript | 5+ | Tipado estático |
| Prisma ORM | 5+ | ORM y gestión de base de datos |
| PostgreSQL | 15+ | Base de datos relacional |
| jsonwebtoken | 9+ | Generación y verificación de JWT |
| bcrypt | 5+ | Hash de contraseñas |
| pdfkit | 0.15+ | Generación de certificados PDF |
| multer | 1+ | Manejo de subida de archivos (si aplica) |
| cors | 2+ | Middleware CORS |
| express-rate-limit | 7+ | Rate limiting |
| swagger-jsdoc + swagger-ui-express | - | Documentación de API |
| jest + supertest | - | Testing unitario e integración |

---

## 3. Arquitectura del Proyecto

### 3.1. Estructura del Monorepo

```
Los_tres_mosqueteros_IS3/
├── TP02-RequerimientosconSDD/
│   ├── project.md              (este archivo)
│   ├── contracts.md            (contratos de API globales)
│   └── specs/
│       ├── auth/spec.md        (Autenticación y Gestión de Usuarios)
│       ├── eventos/spec.md     (Gestión de Eventos y Catálogo)
│       ├── inscripcion/spec.md (Inscripción de Participantes y Cupos)
│       ├── acreditacion/spec.md (Acreditación y Certificados)
│       ├── encuestas/spec.md   (Encuestas y Comentarios)
│       └── informes/spec.md    (Informes y Agenda)
│
├── backend/                    (API REST en Node.js + Express + TypeScript)
│   ├── src/
│   │   ├── index.ts            (entry point)
│   │   ├── app.ts              (configuración de Express)
│   │   ├── config/             (configuraciones: DB, JWT, etc.)
│   │   ├── controllers/        (controladores por dominio)
│   │   ├── routes/             (rutas por dominio)
│   │   ├── middlewares/        (auth, validación, manejo de errores)
│   │   ├── services/           (lógica de negocio)
│   │   ├── utils/              (funciones auxiliares)
│   │   ├── types/              (tipos TypeScript compartidos)
│   │   └── prisma/
│   │       └── schema.prisma   (esquema de Prisma ORM)
│   ├── package.json
│   ├── tsconfig.json
│   ├── .env.example
│   └── jest.config.ts
│
└── frontend/                   (SPA en React + TypeScript + Vite)
    ├── src/
    │   ├── main.tsx            (entry point)
    │   ├── App.tsx             (componente raíz con routing)
    │   ├── components/         (componentes reutilizables)
    │   ├── pages/              (páginas/vistas por dominio)
    │   ├── services/           (llamadas a la API)
    │   ├── hooks/              (custom hooks)
    │   ├── context/            (contextos de React)
    │   ├── types/              (tipos TypeScript)
    │   ├── utils/              (funciones auxiliares)
    │   └── assets/             (imágenes, estilos globales)
    ├── index.html
    ├── package.json
    ├── tsconfig.json
    ├── vite.config.ts
    └── .env.example
```

### 3.2. Separación de responsabilidades

- **Frontend:** Solo presentación, validación de formularios, estado de UI, comunicación con API.
- **Backend:** Validación de negocio, autenticación/autorización, acceso a datos, generación de PDFs.
- **Base de datos:** Persistencia relacional con PostgreSQL, esquema definido por Prisma.

---

## 4. Convenciones de Código

### 4.1. Naming Conventions

| Elemento | Convención | Ejemplo |
|---|---|---|
| Archivos TS/TSX | camelCase | `authController.ts`, `UserCard.tsx` |
| Componentes React | PascalCase | `EventForm`, `SurveyResults` |
| Variables y funciones | camelCase | `handleLogin`, `userData` |
| Constantes | UPPER_SNAKE_CASE | `MAX_EVENT_CAPACITY`, `JWT_EXPIRY` |
| Interfaces y tipos | PascalCase | `IUser`, `EventDTO`, `EnrollmentStatus` |
| Enumeraciones | PascalCase | `EventStatus`, `UserRole` |
| Carpetas | kebab-case o minúsculas | `auth`, `event-sessions`, `user-profile` |

### 4.2. Estructura de archivos por dominio (Backend)

Cada dominio debe seguir esta estructura dentro de `backend/src/`:

```
controllers/
  └── authController.ts
routes/
  └── authRoutes.ts
services/
  └── authService.ts
types/
  └── auth.types.ts
```

### 4.3. Estructura de archivos por dominio (Frontend)

Cada dominio debe seguir esta estructura dentro de `frontend/src/`:

```
pages/
  └── Login.tsx
components/
  └── LoginForm.tsx
services/
  └── authService.ts
```

### 4.4. TypeScript

- Usar `strict: true` en `tsconfig.json`.
- Preferir interfaces para tipos de datos y tipos para uniones/intersecciones.
- No usar `any`. Usar `unknown` cuando sea necesario y hacer type narrowing.
- Usar tipos genéricos en servicios y hooks reutilizables.

### 4.5. Linting y Formato

- ESLint configurado con `@typescript-eslint`.
- Prettier como formatter.
- Husky + lint-staged para pre-commit hooks.
- Reglas obligatorias:
  - `no-unused-vars`: error
  - `no-console`: warn (permitir en desarrollo)
  - `@typescript-eslint/explicit-function-return-type`: off (opcional)

---

## 5. Convenciones de Git

### 5.1. Branch Naming

```
<tipo>/<descripcion-corta>
```

| Tipo | Uso |
|---|---|
| `feature/` | Nueva funcionalidad |
| `fix/` | Corrección de bug |
| `docs/` | Cambios en documentación |
| `refactor/` | Refactorización sin cambio de comportamiento |
| `chore/` | Tareas de mantenimiento |

Ejemplos: `feature/auth-module`, `fix/event-validation`, `docs/add-specs`

### 5.2. Commit Messages

Seguir Conventional Commits:

```
<tipo>(<scope>): <descripción>

[cuerpo opcional]
```

| Tipo | Descripción |
|---|---|
| `feat` | Nueva funcionalidad |
| `fix` | Corrección de bug |
| `docs` | Documentación |
| `style` | Formato, sin cambio de lógica |
| `refactor` | Refactorización |
| `test` | Tests |
| `chore` | Tareas de mantenimiento |

Ejemplos:
- `feat(auth): add login and register endpoints`
- `fix(events): validate max capacity before enrollment`
- `docs(specs): add accreditation module spec`

### 5.3. Pull Requests

- Cada PR debe tener un título descriptivo siguiendo Conventional Commits.
- Descripción debe incluir: qué se hizo, por qué, y cómo probarlo.
- Requiere al menos 1 revisión de un compañero antes de mergear.
- No mergear directamente a `main` desde rama de trabajo (requiere PR).

---

## 6. Estándares de API REST

### 6.1. Base URL

```
/api/v1/
```

Todos los endpoints deben estar versionados bajo `/api/v1/`.

### 6.2. Métodos HTTP

| Método | Uso |
|---|---|
| GET | Leer recursos (sin efectos secundarios) |
| POST | Crear recursos o ejecutar acciones |
| PUT | Actualizar recurso completo |
| PATCH | Actualización parcial |
| DELETE | Eliminar recurso |

### 6.3. Convenciones de Rutas

- Usar sustantivos plurales para recursos: `/events`, `/users`, `/enrollments`
- Las acciones específicas usan POST con ruta descriptiva: `/auth/login`, `/auth/register`
- Sub-recursos anidados: `/events/:id/sessions`, `/events/:id/enrollments`
- Filtrado y paginación por query params: `?status=active&page=1&limit=20`

### 6.4. Status Codes

| Código | Uso |
|---|---|
| 200 | OK (GET, PUT, PATCH exitosos) |
| 201 | Created (POST exitoso que crea recurso) |
| 204 | No Content (DELETE exitoso) |
| 400 | Bad Request (validación fallida) |
| 401 | Unauthorized (no autenticado) |
| 403 | Forbidden (sin permisos) |
| 404 | Not Found |
| 409 | Conflict (ej: cupo lleno, email duplicado) |
| 422 | Unprocessable Entity (datos inválidos semánticamente) |
| 500 | Internal Server Error |

### 6.5. Formato de Respuesta Exitosa

```json
{
  "success": true,
  "data": { ... }
}
```

Para listados con paginación:

```json
{
  "success": true,
  "data": [...],
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

### 6.6. Formato de Respuesta de Error

```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "El campo email es obligatorio",
    "details": [
      { "field": "email", "message": "Formato de email inválido" }
    ]
  }
}
```

### 6.7. Paginación

- Parámetros: `page` (default: 1), `limit` (default: 20, max: 100)
- Incluir metadata en `meta` del response
- Cursor-based pagination no es requerido para este proyecto

### 6.8. Filtrado y Ordenamiento

- Filtrado por query params: `?status=approved&type=congress`
- Ordenamiento: `?sort=createdAt&order=desc`
- Campos de filtrado deben estar documentados por endpoint

---

## 7. Seguridad

### 7.1. Autenticación

- JWT con access token (15 min) y refresh token (7 días)
- Access token enviado en header `Authorization: Bearer <token>`
- Refresh token almacenado en httpOnly cookie
- Contraseñas hasheadas con bcrypt (salt rounds: 12)

### 7.2. Autorización

- Middleware de autenticación para rutas protegidas
- Middleware de autorización basado en roles
- Validación de ownership en recursos (ej: solo el organizador puede editar su evento)

### 7.3. Validación de Entrada

- Validar todos los inputs del lado del servidor con Zod o Joi
- Sanitizar strings para prevenir XSS
- Usar prepared statements (Prisma lo hace automáticamente) para prevenir SQL injection

### 7.4. CORS

- Configurar CORS para permitir solo el dominio del frontend en producción
- En desarrollo: `http://localhost:5173` (Vite default)

### 7.5. Rate Limiting

- API rate limiting: 100 requests por 15 minutos por IP
- Auth endpoints: 10 requests por 15 minutos por IP (login, register)

---

## 8. Base de Datos

### 8.1. Configuración

- Motor: PostgreSQL 15+
- ORM: Prisma 5+
- Migrations: gestionadas con `prisma migrate`
- Seed: script de seed para datos iniciales (templates de encuestas, admin por defecto)

### 8.2. Convenciones de Modelado

- IDs como `Int` con `@id @default(autoincrement())` o `String` con `@id @default(uuid())`
- Timestamps: `createdAt` y `updatedAt` en todos los modelos principales
- Soft delete preferido sobre hard delete para entidades de negocio (campo `deletedAt`)
- Relaciones definidas explícitamente con `@relation`
- Índices en campos de búsqueda frecuente

### 8.3. Variables de Entorno

Backend `.env`:

```
DATABASE_URL="postgresql://user:password@localhost:5432/eventos_db?schema=public"
JWT_ACCESS_SECRET="your-super-secret-access-key"
JWT_REFRESH_SECRET="your-super-secret-refresh-key"
JWT_ACCESS_EXPIRY="15m"
JWT_REFRESH_EXPIRY="7d"
PORT=3001
CORS_ORIGIN="http://localhost:5173"
NODE_ENV="development"
```

Frontend `.env`:

```
VITE_API_URL="http://localhost:3001/api/v1"
```

---

## 9. Despliegue Local

### 9.1. Requisitos

- Node.js 20+ LTS
- PostgreSQL 15+
- npm o pnpm

### 9.2. Pasos de instalación

```bash
# 1. Clonar repositorio
git clone <repo-url>
cd Los_tres_mosqueteros_IS3

# 2. Configurar backend
cd backend
cp .env.example .env
# Editar .env con credenciales de PostgreSQL
npm install
npx prisma migrate dev
npx prisma db seed
npm run dev

# 3. Configurar frontend (en otra terminal)
cd frontend
cp .env.example .env
npm install
npm run dev
```

### 9.3. URLs de desarrollo

- Frontend: `http://localhost:5173`
- Backend API: `http://localhost:3001/api/v1`
- Swagger Docs: `http://localhost:3001/api-docs`

---

## 10. Testing

### 10.1. Herramientas

- Backend: Jest + Supertest
- Frontend: Vitest + React Testing Library

### 10.2. Cobertura mínima

- Servicios y controladores del backend: 70%+ de cobertura
- Componentes críticos del frontend: 60%+ de cobertura

### 10.3. Tipos de tests

- Unitarios: funciones puras, servicios
- Integración: endpoints de API, interacciones con DB
- E2E: no requerido para este proyecto

---

## 11. Documentación

- Swagger/OpenAPI para documentación de la API (autogenerado desde comentarios JSDoc)
- Cada spec en `specs/` documenta un módulo completo
- `contracts.md` define los contratos globales de la API
- README en cada carpeta principal si es necesario
