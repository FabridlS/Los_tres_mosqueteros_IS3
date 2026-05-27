# Spec: Autenticación y Gestión de Usuarios

## 1. Objetivo y Contexto

Este módulo gestiona el registro, autenticación y administración de usuarios del sistema. Proporciona los mecanismos para que los usuarios creen cuentas, inicien sesión, mantengan su sesión activa mediante tokens JWT, y gestionen su perfil. También incluye la gestión de roles y permisos para el control de acceso a las distintas funcionalidades del sistema.

### Alcance

- Registro de nuevos usuarios con validación
- Inicio de sesión con email y contraseña
- Generación y renovación de tokens JWT (access + refresh)
- Cierre de sesión (invalidación de refresh token)
- Obtención y actualización del perfil de usuario
- Listado de usuarios (solo admin/organizador)
- Gestión de roles del sistema

### Fuera de alcance

- Registro con redes sociales (OAuth)
- Autenticación de dos factores (2FA)
- Recuperación de contraseña por email (se puede agregar en futuro)
- Gestión de permisos granulares (solo roles predefinidos)

---

## 2. Historias de Usuario y Criterios de Aceptación

### HU-AUTH-01: Registrarse en la Plataforma

**Como** nuevo usuario, **quiero** crear una cuenta en la plataforma **para** poder inscribirme a eventos y gestionar mi participación.

**Criterios de aceptación:**

1. El usuario puede registrarse con nombre, email y contraseña.
2. El email debe ser único en el sistema (no se permiten duplicados).
3. La contraseña debe tener al menos 8 caracteres.
4. Por defecto, el nuevo usuario recibe el rol PARTICIPANTE.
5. El registro retorna un access token y refresh token automáticamente.
6. Si el email ya existe, retornar error EMAIL_ALREADY_EXISTS (409).

### HU-AUTH-02: Iniciar Sesión (Reforzada)

**Como** usuario registrado, **quiero** iniciar sesión con mis credenciales **para** acceder a la plataforma.

**Criterios de aceptación:**

1. El usuario ingresa email y contraseña.
2. Si las credenciales son correctas, se retorna access token y refresh token.
3. [OWASP] Si las credenciales son incorrectas, el sistema debe retornar un mensaje de error genérico y uniforme (ej. "Credenciales inválidas"), sin revelar si el error fue en el email o en la contraseña para evitar la enumeración de usuarios. Retornar error INVALID_CREDENTIALS (401).
4. [OWASP] El sistema debe bloquear temporalmente la cuenta (Account Lockout) por 15 minutos tras 5 intentos de inicio de sesión fallidos consecutivos.
5. [OWASP] Durante el periodo de bloqueo, cualquier intento de inicio de sesión (incluso con la contraseña correcta) debe ser rechazado con el mismo mensaje genérico para no dar pistas al atacante.
6. El access token expira en 15 minutos.
7. El refresh token expira en 7 días y se almacena en una cookie httpOnly y Secure.

### HU-AUTH-03: Renovar Token de Acceso

**Como** usuario con sesión activa, **quiero** renovar mi access token cuando expira **para** continuar usando la plataforma sin volver a loguearme.

**Criterios de aceptación:**

1. Se envía el refresh token válido al endpoint de refresh.
2. Si el refresh token es válido, se retorna nuevo access token y nuevo refresh token.
3. Si el refresh token expiró, retornar error TOKEN_EXPIRED (401).
4. Si el refresh token es inválido, retornar error INVALID_TOKEN (401).
5. El refresh token anterior se invalida al generar uno nuevo (rotación).

### HU-AUTH-04: Cerrar Sesión

**Como** usuario, **quiero** cerrar mi sesión **para** salir de la plataforma de forma segura.

**Criterios de aceptación:**

1. Al cerrar sesión, el refresh token se invalida en el servidor.
2. El access token se descarta del lado del cliente.
3. Después de logout, el refresh token no puede usarse para renovar sesión.
4. Si el usuario no está autenticado, retornar 204 sin error.

### HU-AUTH-05: Ver Mi Perfil

**Como** usuario autenticado, **quiero** ver mi perfil **para** consultar mis datos.

**Criterios de aceptación:**

1. El usuario autenticado puede acceder a su perfil con `GET /users/me`.
2. Se muestra: id, nombre, email, rol, fecha de registro.
3. No se muestra la contraseña ni datos sensibles.

### HU-AUTH-06: Actualizar Mi Perfil

**Como** usuario autenticado, **quiero** actualizar mis datos personales **para** mantener mi información actualizada.

**Criterios de aceptación:**

1. El usuario puede actualizar su nombre y email.
2. No puede cambiar su rol directamente (solo admin puede).
3. Si cambia el email, debe ser único en el sistema.
4. Se retorna el perfil actualizado.

### HU-AUTH-07: Cambiar Contraseña

**Como** usuario autenticado, **quiero** cambiar mi contraseña **para** mantener la seguridad de mi cuenta.

**Criterios de aceptación:**

1. El usuario ingresa contraseña actual y nueva contraseña.
2. Si la contraseña actual es incorrecta, retornar error INVALID_CREDENTIALS (401).
3. La nueva contraseña debe tener al menos 8 caracteres.
4. La nueva contraseña no puede ser igual a la actual.
5. Se retorna éxito sin exponer datos sensibles.

### HU-AUTH-08: Listar Usuarios (Admin)

**Como** administrador, **quiero** listar todos los usuarios del sistema **para** gestionar la plataforma.

**Criterios de aceptación:**

1. Solo usuarios con rol ADMIN pueden acceder al listado completo.
2. La lista es paginada (page, limit).
3. Se puede filtrar por rol.
4. Se puede buscar por nombre o email.
5. Cada usuario en la lista muestra: id, nombre, email, rol, fecha de registro.

---

## 3. Requisitos Funcionales y Reglas de Negocio

### RF-AUTH-01: Registro

- **RF-AUTH-01.1:** El email es obligatorio y debe tener formato válido.
- **RF-AUTH-01.2:** La contraseña es obligatoria, mínimo 8 caracteres.
- **RF-AUTH-01.3:** El nombre es obligatorio, mínimo 2 caracteres, máximo 100.
- **RF-AUTH-01.4:** El rol por defecto es PARTICIPANTE.
- **RF-AUTH-01.5:** El email debe ser único en todo el sistema.
- **RF-AUTH-01.6:** La contraseña se hashea con bcrypt (salt rounds: 12).

### RF-AUTH-02: Autenticación

- **RF-AUTH-02.1:** Las credenciales se validan comparando email y contraseña hasheada.
- **RF-AUTH-02.2:** El access token contiene: sub (userId), email, roles, iat, exp.
- **RF-AUTH-02.3:** El refresh token contiene: sub (userId), jti (unique id), iat, exp.
- **RF-AUTH-02.4:** Los tokens se generan con la secret key configurada en el entorno.
- **RF-AUTH-02.5:** El access token se envía en header `Authorization: Bearer <token>`.
- **RF-AUTH-02.6:** El refresh token se almacena en cookie httpOnly con secure flag en producción.

### RF-AUTH-03: Token Refresh

- **RF-AUTH-03.1:** El refresh token debe existir y no estar revocado.
- **RF-AUTH-03.2:** Al renovar, se genera un nuevo refresh token y se revoca el anterior.
- **RF-AUTH-03.3:** El jti del refresh token se usa para control de revocación.

### RF-AUTH-04: Autorización por Roles

- **RF-AUTH-04.1:** Los roles disponibles: ADMIN, ORGANIZADOR, DISERTANTE, PARTICIPANTE.
- **RF-AUTH-04.2:** El middleware de autorización verifica el rol requerido por endpoint.
- **RF-AUTH-04.3:** Un usuario puede tener un solo rol activo a la vez.
- **RF-AUTH-04.4:** Solo ADMIN puede cambiar el rol de un usuario.

### RF-AUTH-05: Gestión de Perfil

- **RF-AUTH-05.1:** El usuario puede actualizar nombre y email.
- **RF-AUTH-05.2:** El email nuevo debe ser único.
- **RF-AUTH-05.3:** El usuario no puede modificar su propio rol.
- **RF-AUTH-05.4:** La contraseña se actualiza por separado (cambio de contraseña).

### RF-AUTH-06: Seguridad

- **RF-AUTH-06.1:** Rate limiting en endpoints de auth (login: 10/15min, register: 5/15min).
- **RF-AUTH-06.2:** Las contraseñas nunca se retornan en respuestas.
- **RF-AUTH-06.3:** Los errores de autenticación no revelan si el email existe o no (para login).
- **RF-AUTH-06.4:** CORS configurado para permitir credenciales (cookies).

---

## 4. Restricciones Técnicas Específicas

- **RT-AUTH-01:** bcrypt con salt rounds de 12 (definido en project.md).
- **RT-AUTH-02:** Access token expira en 15 minutos, refresh token en 7 días.
- **RT-AUTH-03:** Refresh token almacenado en httpOnly cookie (no accesible por JS).
- **RT-AUTH-04:** Las variables de entorno JWT_ACCESS_SECRET y JWT_REFRESH_SECRET deben estar configuradas.
- **RT-AUTH-05:** Se usa jsonwebtoken para generación y verificación de tokens.
- **RT-AUTH-06:** El campo `password` del modelo User nunca se incluye en respuestas serializadas.

---

## 5. Modelo de Datos

### 5.1. Enum UserRole

```prisma
enum UserRole {
  ADMIN
  ORGANIZADOR
  DISERTANTE
  PARTICIPANTE
}
```

### 5.2. Modelo User

```prisma
model User {
  id            Int        @id @default(autoincrement())
  nombre        String     @map("nombre")
  email         String     @unique @map("email")
  password      String     @map("password")
  rol           UserRole   @default(PARTICIPANTE) @map("rol")
  createdAt     DateTime   @default(now()) @map("created_at")
  updatedAt     DateTime   @updatedAt @map("updated_at")

  // Relaciones
  eventos       Event[]    @relation("EventOrganizer")       // Eventos que organiza
  inscripciones Enrollment[] @relation("EnrollmentUser")     // Inscripciones como participante
  inscripcionesPorStaff   Enrollment[] @relation("EnrollmentEnroller") // Inscripciones hechas por este usuario
  respuestas    SurveyResponse[]
  comentarios   Comment[]

  @@index([email])
  @@index([rol])
  @@map("users")
}
```

### 5.3. Modelo RefreshToken (para revocación)

```prisma
model RefreshToken {
  id        Int      @id @default(autoincrement())
  jti       String   @unique @map("jti")
  userId    Int      @map("user_id")
  expiresAt DateTime @map("expires_at")
  revoked   Boolean  @default(false) @map("revoked")
  createdAt DateTime @default(now()) @map("created_at")

  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@index([jti])
  @@index([revoked])
  @@map("refresh_tokens")
}
```

### 5.4. DTOs de Respuesta

#### UserDTO

```typescript
interface UserDTO {
  id: number;
  nombre: string;
  email: string;
  rol: UserRole;
  createdAt: string;
}
```

#### AuthResponseDTO

```typescript
interface AuthResponseDTO {
  user: UserDTO;
  accessToken: string;
  expiresIn: number; // segundos
}
```

#### LoginDTO

```typescript
interface LoginDTO {
  email: string;
  password: string;
}
```

#### RegisterDTO

```typescript
interface RegisterDTO {
  nombre: string;
  email: string;
  password: string;
}
```

#### ChangePasswordDTO

```typescript
interface ChangePasswordDTO {
  currentPassword: string;
  newPassword: string;
}
```

#### UpdateProfileDTO

```typescript
interface UpdateProfileDTO {
  nombre?: string;
  email?: string;
}
```

#### UserListDTO (paginado)

```typescript
interface UserListDTO {
  users: UserDTO[];
  meta: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}
```

---

## 6. Plan de Tareas

### Tarea 1: Definición de modelos y migraciones
- Definir enum UserRole en Prisma
- Definir modelo User con campos y relaciones
- Definir modelo RefreshToken para revocación
- Crear y ejecutar migración
- Crear seed script con usuario admin por defecto

### Tarea 2: Servicio de autenticación
- Implementar `AuthService` con métodos:
  - `register(nombre, email, password)`: crear usuario, hashear contraseña
  - `login(email, password)`: validar credenciales, generar tokens
  - `refreshToken(jti)`: verificar, revocar anterior, generar nuevos
  - `logout(jti)`: revocar refresh token
- Implementar generación de access y refresh tokens con jsonwebtoken
- Implementar validación de tokens (verify)

### Tarea 3: Middleware de autenticación
- Implementar `authMiddleware` para verificar access token
- Implementar extracción del token desde header `Authorization: Bearer`
- Implementar `attachUser` al request object
- Implementar manejo de errores: token inválido, expirado, ausente

### Tarea 4: Middleware de autorización
- Implementar `roleMiddleware(...roles)` para verificar rol del usuario
- Implementar verificación contra el campo `rol` del usuario
- Retornar 403 si el rol no está permitido

### Tarea 5: Endpoints de autenticación
- Implementar `POST /auth/register` (registro)
- Implementar `POST /auth/login` (inicio de sesión)
- Implementar `POST /auth/refresh` (renovar token)
- Implementar `POST /auth/logout` (cerrar sesión)
- Implementar `GET /auth/me` (perfil actual)
- Implementar `PUT /auth/me` (actualizar perfil)
- Implementar `PUT /auth/change-password` (cambiar contraseña)

### Tarea 6: Endpoint de listado de usuarios (Admin)
- Implementar `GET /users` (listado con paginación y filtros)
- Implementar filtrado por rol
- Implementar búsqueda por nombre/email
- Proteger con roleMiddleware(ADMIN)

### Tarea 7: Frontend - Registro
- Crear página de registro con formulario (React Hook Form + validación)
- Validación: email válido, contraseña mínimo 8 caracteres, nombre requerido
- Mostrar errores de validación inline
- Redirigir al login tras registro exitoso (o loguear automáticamente)

### Tarea 8: Frontend - Login
- Crear página de login con formulario
- Enviar credenciales al backend
- Almacenar access token en memoria/estado
- El refresh token se maneja via cookie httpOnly (automático)
- Redirigir al home tras login exitoso
- Mostrar errores de credenciales

### Tarea 9: Frontend - Gestión de sesión
- Implementar interceptor de Axios para incluir access token en requests
- Implementar refresh token automático cuando el access token expira (401)
- Implementar logout: llamar al endpoint y limpiar estado
- Implementar protección de rutas privadas (Redirect si no autenticado)

### Tarea 10: Frontend - Perfil de usuario
- Crear página de perfil con datos del usuario
- Formulario para editar nombre y email
- Formulario para cambiar contraseña
- Mostrar rol del usuario (solo lectura)

### Tarea 11: Frontend - Panel de administración (usuarios)
- Crear página de gestión de usuarios (solo ADMIN)
- Tabla de usuarios con paginación
- Filtros por rol
- Búsqueda por nombre/email

### Tarea 12: Tests
- Tests unitarios de registro (validación, hash de contraseña)
- Tests unitarios de login (credenciales correctas/incorrectas)
- Tests unitarios de token refresh y revocación
- Tests unitarios de middlewares de auth y roles
- Tests de integración de endpoints de autenticación
- Tests de rate limiting en endpoints de auth

### Tarea 13: Documentación Swagger
- Agregar JSDoc swagger a todos los endpoints de autenticación
- Configurar esquema de seguridad Bearer en Swagger

---

## 7. Estrategia de Verificación

### 7.1. Tests Unitarios

| Componente | Qué verificar |
|---|---|
| `register` | Email único, contraseña hasheada, rol por defecto PARTICIPANTE |
| `login` | Credenciales correctas/incorrectas, tokens generados correctamente |
| `generateAccessToken` | Payload correcto (sub, email, roles, exp en 15min) |
| `generateRefreshToken` | Payload correcto (sub, jti, exp en 7 días) |
| `verifyToken` | Token válido retorna payload, token expirado lanza error |
| `changePassword` | Password actual correcta, nueva password válida, hash correcto |

### 7.2. Tests de Integración

| Endpoint | Escenario | Resultado esperado |
|---|---|---|
| `POST /auth/register` | Datos válidos | 201, user + accessToken |
| `POST /auth/register` | Email duplicado | 409, EMAIL_ALREADY_EXISTS |
| `POST /auth/register` | Password < 8 caracteres | 400, VALIDATION_ERROR |
| `POST /auth/login` | Credenciales correctas | 200, accessToken + refreshToken |
| `POST /auth/login` | Credenciales incorrectas | 401, INVALID_CREDENTIALS |
| `POST /auth/refresh` | Refresh token válido | 200, nuevo accessToken |
| `POST /auth/refresh` | Refresh token expirado | 401, TOKEN_EXPIRED |
| `POST /auth/logout` | Refresh token válido | 204, token revocado |
| `POST /auth/logout` | Sin autenticación | 204, sin error |
| `GET /auth/me` | Autenticado | 200, UserDTO |
| `GET /auth/me` | No autenticado | 401, UNAUTHORIZED |
| `PUT /auth/me` | Datos válidos | 200, perfil actualizado |
| `PUT /auth/me` | Email duplicado | 409, EMAIL_ALREADY_EXISTS |
| `PUT /auth/change-password` | Correcto | 200, contraseña actualizada |
| `PUT /auth/change-password` | Password actual incorrecta | 401, INVALID_CREDENTIALS |
| `GET /users` | Admin | 200, lista paginada |
| `GET /users` | No admin | 403, FORBIDDEN |

### 7.3. Tests Manuales (QA)

1. Registrarse con datos válidos → verificar que se crea usuario y retorna token.
2. Registrarse con email existente → debe fallar con EMAIL_ALREADY_EXISTS.
3. Iniciar sesión con credenciales correctas → verificar que retorna tokens.
4. Iniciar sesión con credenciales incorrectas → debe fallar con INVALID_CREDENTIALS.
5. Esperar expiración de access token → verificar que refresh funciona automáticamente.
6. Cerrar sesión → verificar que el refresh token ya no funciona.
7. Actualizar perfil → verificar que los cambios persisten.
8. Cambiar contraseña → verificar que se puede login con la nueva.
9. Como admin: listar usuarios → verificar paginación y filtros.
10. Intentar acceder a ruta protegida sin token → debe redirigir a login.

### 7.4. Criterios de Aceptación del Módulo

- [ ] Todos los endpoints documentados en Swagger
- [ ] Cobertura de tests unitarios >= 70%
- [ ] Registro con validación de email único y contraseña
- [ ] Login con generación de access y refresh tokens
- [ ] Refresh token con rotación y revocación
- [ ] Logout invalida refresh token correctamente
- [ ] Middleware de autenticación protege rutas
- [ ] Middleware de autorización verifica roles
- [ ] Perfil de usuario editable (nombre, email)
- [ ] Cambio de contraseña funcional
- [ ] Listado de usuarios con paginación y filtros (solo admin)
- [ ] Frontend responsivo con Bootstrap
- [ ] Rate limiting configurado en endpoints de auth
- [ ] Refresh token en cookie httpOnly
