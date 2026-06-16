# Spec: Informes y Agenda

## 1. Objetivo y Contexto

Este módulo gestiona la generación de informes estadísticos de eventos y la generación de agendas. Los informes proporcionan métricas de participación, asistencia, satisfacción y otros datos relevantes. La agenda del evento muestra todas las sesiones organizadas cronológicamente y puede ser exportada.

### Alcance

- Generación de informe estadístico del evento (inscripciones, asistencia, encuestas)
- Generación de agenda del evento (listado cronológico de sesiones)
- Exportación de agenda en formato imprimible
- Dashboard del organizador (resumen de todos sus eventos)

### Fuera de alcance

- Exportación de informes a Excel
- Dashboard del admin (estadísticas globales de la plataforma)
- Informes financieros

---

## 2. Historias de Usuario y Criterios de Aceptación

### HU-INF-01: Generar Informe del Evento

**Como** organizador, **quiero** generar un informe estadístico de mi evento **para** evaluar su desempeño.

**Criterios de aceptación:**

1. El informe incluye: total de inscriptos, inscriptos por estado (pendiente, aprobada, rechazada, cancelada), total de asistentes (acreditados), porcentaje de asistencia, cantidad de sesiones, promedio de satisfacción (de encuestas), total de comentarios.
2. El informe se genera para cualquier evento del organizador.
3. El informe muestra datos actualizados en tiempo real.
4. El admin puede generar informes de cualquier evento.

### HU-INF-02: Generar Agenda del Evento

**Como** participante u organizador, **quiero** ver la agenda del evento **para** conocer el cronograma de actividades.

**Criterios de aceptación:**

1. La agenda muestra todas las sesiones del evento ordenadas cronológicamente.
2. Cada sesión muestra: título, disertante, fecha/hora de inicio, fecha/hora de fin, ubicación/sala, descripción.
3. Si el evento no tiene sesiones, se indica que la agenda será publicada próximamente.
4. La agenda se puede ver desde antes de que el evento comience.
5. La agenda se puede imprimir o exportar como PDF.

### HU-INF-03: Exportar Agenda como PDF

**Como** participante, **quiero** descargar la agenda del evento en PDF **para** consultarla sin conexión.

**Criterios de aceptación:**

1. El PDF de la agenda incluye: título del evento, fechas del evento, todas las sesiones ordenadas.
2. El diseño es limpio y legible.
3. El PDF se genera on-demand (no se almacena).
4. El nombre del archivo sigue el patrón: `agenda_{titulo_evento}.pdf`.
5. [OWASP] Al renderizar datos dinámicos dentro del PDF con pdfkit (como el título del evento, nombres de sesiones o disertantes), el backend debe sanitizar rigurosamente los strings para neutralizar cualquier intento de inyección de scripts o caracteres de control que puedan corromper el motor de generación de PDF o provocar ataques de inyección si el contenido se previsualiza en el navegador.
6. [OWASP] El parámetro titulo_evento utilizado para nombrar el archivo de salida debe ser sanitizado eliminando caracteres especiales (../, \, /, null bytes) para prevenir vulnerabilidades de Path Traversal o manipulación de cabeceras HTTP (Content-Disposition).

### HU-INF-04: Dashboard del Organizador

**Como** organizador, **quiero** ver un resumen de todos mis eventos **para** tener una vista general de mi actividad.

**Criterios de aceptación:**

1. El dashboard muestra: total de eventos creados, eventos por estado (pendiente, aprobado, finalizado, cancelado), total de participantes en todos mis eventos, promedio de satisfacción general.
2. Se muestra un listado de los últimos 5 eventos con sus métricas principales.
3. Se puede acceder al informe detallado de cada evento desde el dashboard.

### HU-INF-05: Ver Informe con Filtros de Fecha

**Como** organizador, **quiero** filtrar los datos del informe por fecha **para** analizar períodos específicos.

**Criterios de aceptación:**

1. El informe permite filtrar las inscripciones por rango de fecha.
2. Se pueden ver inscripciones por período (semanal, mensual).
3. El filtro de fecha es opcional (sin filtro muestra todo el período del evento).

---

## 3. Requisitos Funcionales y Reglas de Negocio

### RF-INF-01: Acceso a Informes

- **RF-INF-01.1:** Solo el organizador del evento y los admins pueden ver informes.
- **RF-INF-01.2:** La agenda del evento es pública (cualquier usuario puede verla).

### RF-INF-02: Contenido del Informe

- **RF-INF-02.1:** El informe siempre incluye métricas de inscripciones.
- **RF-INF-02.2:** Si el evento tiene encuestas, incluye métricas de satisfacción.
- **RF-INF-02.3:** Si el evento tiene sesiones, incluye conteo de sesiones.
- **RF-INF-02.4:** Si el evento tiene comentarios, incluye conteo y promedio de rating.

### RF-INF-03: Métricas de Inscripción

- **RF-INF-03.1:** Total de inscripciones (todos los estados).
- **RF-INF-03.2:** Desglose por estado: pendiente, aprobada, rechazada, cancelada, pendiente_registro.
- **RF-INF-03.3:** Porcentaje de asistencia: (asistentes / inscriptos_aprobados) * 100.
- **RF-INF-03.4:** Tasa de aprobación: (aprobadas / (aprobadas + rechazadas + pendientes)) * 100.

### RF-INF-04: Métricas de Encuestas

- **RF-INF-04.1:** Promedio general de satisfacción (promedio de todas las preguntas de escala de todas las encuestas).
- **RF-INF-04.2:** Cantidad de respuestas por encuesta.
- **RF-INF-04.3:** Tasa de respuesta: (respuestas / inscriptos_aprobados) * 100.

### RF-INF-05: Métricas de Comentarios

- **RF-INF-05.1:** Total de comentarios.
- **RF-INF-05.2:** Promedio de rating de comentarios (si tienen rating).

### RF-INF-06: Agenda

- **RF-INF-06.1:** La agenda se genera a partir de las sesiones del evento.
- **RF-INF-06.2:** Las sesiones se ordenan por startTime ascending.
- **RF-INF-06.3:** Si hay sesiones superpuestas, se muestran en paralelo (indicando la superposición).
- **RF-INF-06.4:** El PDF de agenda se genera con pdfkit (mismo que certificados).

### RF-INF-07: Dashboard

- **RF-INF-07.1:** Las métricas del dashboard se calculan en base a todos los eventos del organizador.
- **RF-INF-07.2:** El promedio de satisfacción general se calcula solo sobre eventos con encuestas respondidas.
- **RF-INF-07.3:** El dashboard no incluye eventos cancelados en las métricas de participantes (pero sí en el conteo de eventos por estado).

---

## 4. Restricciones Técnicas Específicas

- **RT-INF-01:** Los informes se calculan con consultas agregadas de Prisma (groupBy, count, average).
- **RT-INF-02:** La agenda se retorna como JSON para visualización web y como PDF para descarga.
- **RT-INF-03:** No se almacenan los informes generados (se calculan on-demand).
- **RT-INF-04:** El PDF de agenda usa pdfkit con diseño tabular.
- **RT-INF-05:** Para el dashboard, las consultas deben ser optimizadas para no hacer N+1 queries.

---

## 5. Modelo de Datos

### 5.1. Modelos

Este módulo no introduce nuevos modelos. Reutiliza los modelos existentes:

- `Event` - Para datos del evento
- `EventSession` - Para la agenda
- `Enrollment` - Para métricas de inscripción
- `Attendance` - Para métricas de asistencia
- `Survey` + `SurveyResponse` + `SurveyAnswer` - Para métricas de encuestas
- `Comment` - Para métricas de comentarios
- `Certificate` - Para métricas de certificados emitidos

### 5.2. DTOs de Respuesta

#### ReportDTO

```typescript
interface ReportDTO {
  event: {
    id: number;
    title: string;
    startDate: string;
    endDate: string;
    status: string;
  };
  enrollments: {
    total: number;
    byStatus: {
      pendiente: number;
      aprobada: number;
      rechazada: number;
      cancelada: number;
      pendienteRegistro: number;
    };
    approvalRate: number; // porcentaje
  };
  attendance: {
    total: number;
    rate: number; // porcentaje sobre aprobadas
  };
  sessions: {
    total: number;
  };
  surveys: {
    totalSurveys: number;
    totalResponses: number;
    responseRate: number; // porcentaje sobre aprobadas
    averageSatisfaction: number | null; // promedio 1-5, null si no hay encuestas
  };
  comments: {
    total: number;
    averageRating: number | null; // promedio 1-5, null si no hay ratings
  };
  certificates: {
    total: number;
    byType: {
      asistencia: number;
      aprobacion: number;
      expositor: number;
    };
  };
}
```

#### AgendaDTO

```typescript
interface AgendaDTO {
  event: {
    id: number;
    title: string;
    startDate: string;
    endDate: string;
    modality: string;
    location: string | null;
    streamUrl: string | null;
  };
  sessions: {
    id: number;
    title: string;
    description: string | null;
    speaker: string;
    startTime: string;
    endTime: string;
    location: string | null;
  }[];
}
```

#### DashboardDTO

```typescript
interface DashboardDTO {
  summary: {
    totalEvents: number;
    eventsByStatus: {
      pendiente: number;
      aprobado: number;
      finalizado: number;
      cancelado: number;
      rechazado: number;
    };
    totalParticipants: number;
    averageSatisfaction: number | null;
  };
  recentEvents: {
    id: number;
    title: string;
    startDate: string;
    status: string;
    enrollments: number;
    attendance: number;
    averageSatisfaction: number | null;
  }[];
}
```

---

## 6. Plan de Tareas

### Tarea 1: Servicio de informes
- Implementar `ReportService` con métodos para calcular cada métrica
- Implementar consultas agregadas con Prisma (count, groupBy, average)
- Implementar cálculo de porcentajes y tasas
- Implementar ensamblado del ReportDTO

### Tarea 2: Servicio de agenda
- Implementar `AgendaService` para obtener sesiones ordenadas
- Implementar generación de PDF de agenda con pdfkit
- Diseñar layout del PDF (tabla con columnas: hora, sesión, disertante, ubicación)

### Tarea 3: Servicio de dashboard
- Implementar `DashboardService` para calcular métricas globales del organizador
- Implementar consultas optimizadas (evitar N+1)
- Implementar ensamblado del DashboardDTO

### Tarea 4: Endpoints de informes
- Implementar `GET /events/:id/report` (generar informe del evento)
- Implementar `GET /events/:id/agenda` (obtener agenda en JSON)
- Implementar `GET /events/:id/agenda/pdf` (descargar agenda en PDF)
- Implementar `GET /organizer/dashboard` (dashboard del organizador)
- Implementar query params opcionales para filtros de fecha en reportes

### Tarea 5: Frontend - Informe del evento
- Crear página "Informe" en panel del organizador
- Mostrar métricas en tarjetas (cards de Bootstrap)
- Mostrar desglose de inscripciones con gráfico de barras (opcional)
- Mostrar métricas de encuestas y comentarios

### Tarea 6: Frontend - Agenda del evento
- Crear página de agenda en detalle del evento
- Mostrar sesiones en formato timeline/tabla cronológica
- Indicar sesiones paralelas
- Botón de descarga PDF

### Tarea 7: Frontend - Dashboard del organizador
- Crear página "Dashboard" en panel del organizador
- Mostrar resumen con tarjetas de métricas
- Mostrar tabla de últimos eventos
- Links a informes individuales

### Tarea 8: Frontend - Filtros de fecha en informes
- Agregar selector de rango de fechas en página de informe
- Recalcular métricas con filtro aplicado
- Botones rápidos: esta semana, este mes, todo

### Tarea 9: Tests
- Tests unitarios de cálculo de métricas
- Tests unitarios de porcentajes y tasas
- Tests de integración de endpoints de informes
- Tests de generación de PDF de agenda (mock)

### Tarea 10: Documentación Swagger
- Agregar JSDoc swagger a todos los endpoints de informes

---

## 7. Estrategia de Verificación

### 7.1. Tests Unitarios

| Componente | Qué verificar |
|---|---|
| `calculateEnrollmentMetrics` | Conteo por estado correcto, tasas de aprobación |
| `calculateAttendanceRate` | (asistentes / aprobados) * 100, maneja división por cero |
| `calculateSurveyMetrics` | Promedio de satisfacción, tasa de respuesta |
| `calculateCommentMetrics` | Total comentarios, promedio de rating |
| `calculateDashboardSummary` | Agregación correcta de todos los eventos |

### 7.2. Tests de Integración

| Endpoint | Escenario | Resultado esperado |
|---|---|---|
| `GET /events/:id/report` | Organizador de evento con datos | 200, ReportDTO completo |
| `GET /events/:id/report` | No organizador | 403, FORBIDDEN |
| `GET /events/:id/report` | Evento sin encuestas | 200, surveys.averageSatisfaction = null |
| `GET /events/:id/report` | Con filtro de fecha | 200, métricas filtradas |
| `GET /events/:id/agenda` | Evento con sesiones | 200, AgendaDTO con sesiones ordenadas |
| `GET /events/:id/agenda` | Evento sin sesiones | 200, AgendaDTO con sessions vacío |
| `GET /events/:id/agenda/pdf` | Evento con sesiones | 200, application/pdf |
| `GET /organizer/dashboard` | Organizador con eventos | 200, DashboardDTO completo |
| `GET /organizer/dashboard` | Organizador sin eventos | 200, DashboardDTO con todo en 0 |

### 7.3. Tests Manuales (QA)

1. Como organizador: ver informe de evento con inscripciones, asistencia, encuestas → verificar que todos los números coinciden con los datos reales.
2. Como organizador: ver informe con filtro de fecha → verificar que las métricas se recalculan.
3. Ver agenda de evento con sesiones → verificar orden cronológico.
4. Descargar agenda como PDF → verificar formato y contenido.
5. Ver dashboard como organizador con múltiples eventos → verificar resumen correcto.
6. Ver dashboard como organizador sin eventos → verificar que muestra ceros.

### 7.4. Criterios de Aceptación del Módulo

- [ ] Todos los endpoints documentados en Swagger
- [ ] Cobertura de tests unitarios >= 70%
- [ ] Informe del evento calcula todas las métricas correctamente
- [ ] Agenda muestra sesiones ordenadas cronológicamente
- [ ] PDF de agenda generado con formato legible
- [ ] Dashboard del organizador muestra resumen correcto
- [ ] Filtros de fecha en informes funcionan correctamente
- [ ] Frontend responsivo con Bootstrap
