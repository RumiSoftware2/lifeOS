# LifeOS — Arquitectura de software y modelo técnico

## 1. Decisión arquitectónica

Construir inicialmente un **monolito modular (modular monolith)** con Spring Boot. No iniciar con microservicios: para un proyecto personal aumentan el coste operativo, despliegue, observabilidad y complejidad de transacciones. Mantener límites de dominio claros para poder extraer servicios en el futuro.

### Stack acordado
- Backend: Java 21 LTS o versión LTS vigente compatible, Spring Boot 3.x/4.x según compatibilidad validada, Spring Web, Spring Security, Spring Data JPA, Bean Validation, Actuator.
- Frontend: React + JavaScript (no TypeScript en esta primera versión) + CSS puro. No usar Tailwind, Bootstrap ni librerías de estilos.
- Base de datos: PostgreSQL.
- Migraciones: Flyway.
- Contenedores: Docker y Docker Compose para desarrollo.
- Pruebas: JUnit 5, Mockito, Testcontainers; Vitest/React Testing Library para frontend si se adopta.
- API: REST documentada con OpenAPI/Swagger.
- Integración: Google Calendar API mediante OAuth 2.0.
- Infraestructura objetivo de aprendizaje: AWS.

## 2. Vista lógica

```text
[Browser: React + CSS]
        |
      HTTPS
        |
[Load Balancer / HTTPS]
        |
[Spring Boot API - modular monolith]
   |        |          |
   |        |          +--> Google Calendar API (OAuth)
   |        +-------------> Scheduler / Job Processor
   +----------------------> PostgreSQL (RDS)
        |
[Logs / Metrics / Alerts]
```

En una primera versión de bajo coste se puede ejecutar backend en una sola tarea/contenedor y PostgreSQL administrado. Añadir cola cuando los trabajos asíncronos lo justifiquen.

## 3. Módulos de dominio del backend

Organizar por funcionalidad, no por capas globales gigantes:

```text
com.lifeos
  identity/
  dashboard/
  planning/
  tasks/
  routines/
  household/
  shopping/
  finance/
  obligations/
  health/
  pets/
  documents/
  calendar/
  notifications/
  automation/
  audit/
  shared/
```

Cada módulo puede contener:
- `api`: controllers y DTOs.
- `application`: casos de uso y servicios.
- `domain`: entidades, value objects y reglas.
- `infrastructure`: repositorios JPA, clientes externos y adaptadores.

No exponer entidades JPA directamente desde los controllers. Usar DTOs, validación, manejo centralizado de errores y paginación.

## 4. Modelo de datos inicial

Entidades sugeridas:
- `users`: identidad, email, zona horaria y preferencias.
- `households`, `household_members`: hogar y miembros/roles (fase posterior).
- `tasks`: título, descripción, categoría, prioridad, estado, vencimiento, duración, owner.
- `recurrence_rules`: frecuencia, intervalo, días, fecha fin, zona horaria y siguiente ejecución.
- `routine_templates`: plantilla de rutina; `routine_occurrences`: ocurrencias concretas.
- `calendar_connections`: proveedor, Google account subject, scopes y estado.
- `calendar_sync_records`: id local, provider event id, etag, estado y última sincronización.
- `financial_accounts`: alias y tipo; nunca guardar credenciales bancarias.
- `transactions`: fecha, importe decimal, moneda, tipo, categoría y descripción.
- `budgets`, `budget_categories`, `savings_goals`, `debts`, `recurring_bills`.
- `obligations`: tipo, entidad, jurisdicción, periodo, fecha límite, fuente y fecha de verificación.
- `shopping_items`, `inventory_items`, `shopping_lists`, `shopping_list_items`.
- `appointments`, `medication_reminders` (datos mínimos y consentimiento).
- `pets`, `pet_care_events`.
- `documents`: metadatos y referencia a almacenamiento privado, no URL pública.
- `automation_rules`, `automation_runs`.
- `notification_preferences`, `notifications`.
- `audit_events`.

### Convenciones
- UUID como identificador público.
- `created_at`, `updated_at`, y `deleted_at` cuando aplique.
- Importes con `NUMERIC(19,4)` y moneda ISO `COP`; no usar `float/double` para dinero. En Java usar `BigDecimal`.
- Fechas-hora como `TIMESTAMPTZ`; eventos de todo el día como `DATE`.
- Guardar zona horaria IANA por usuario, por defecto `America/Bogota`.
- Índices en `user_id`, `due_at`, `status`, `next_run_at`.
- Aislamiento de datos por usuario en cada consulta; no confiar solo en el frontend.

## 5. Diseño de recurrencias y motor de tareas

No usar un simple `@Scheduled` que genere tareas sin control. Diseñar:
1. Una regla recurrente persistida.
2. Un planificador que detecte reglas vencidas.
3. Una creación idempotente de ocurrencias con clave única `(rule_id, occurrence_date)`.
4. Un registro de ejecución y resultado.
5. Reintentos con backoff para errores transitorios.
6. Una cola de fallos (dead-letter conceptual) o estado `FAILED` para revisión.
7. Zona horaria y horario de verano manejados con `ZoneId`, aunque Colombia no usa DST.
8. Operaciones de pausa, reanudación, omitir una ocurrencia y editar solo esta o todas las futuras.

Para el MVP puede usarse Spring Scheduler + tabla de trabajos y locking en base de datos. En la etapa cloud, considerar EventBridge Scheduler y SQS para procesamiento desacoplado. Evitar ejecutar dos veces la misma acción.

## 6. Google Calendar

- Flujo OAuth 2.0 Authorization Code en backend, con `state`, validación de redirect URI y protección CSRF.
- Solicitar el alcance mínimo que permita el caso de uso. Para crear/editar eventos se requiere un scope de escritura apropiado; evaluar `calendar.events` frente a permisos más amplios.
- Guardar refresh token cifrado en backend, nunca en `localStorage` ni en frontend.
- Usar `calendarId=primary` inicialmente.
- Persistir `provider_event_id`, `etag`, vínculo con tarea y estado de sincronización.
- Crear eventos con start/end, timezone, descripción, recurrencia cuando aplique y reminders.
- Implementar actualización y cancelación; no solo creación.
- Idempotencia: generar event ID determinístico válido o usar registro local/clave de idempotencia para evitar duplicados tras reintentos.
- Manejar token revocado, cuota, 401, 403, 429 y errores temporales.
- Sincronización unidireccional LifeOS → Calendar en MVP. La bidireccionalidad requiere resolver conflictos y debe ser una etapa posterior.
- Si el usuario elimina el evento directamente en Google, definir política: detectar y marcar como desvinculado o recrearlo solo con autorización.
- Proveer botón de desconectar cuenta y revocar/borrar tokens.

## 7. Seguridad y privacidad

- Spring Security; contraseñas con Argon2id o BCrypt configurado correctamente.
- Autorización en cada endpoint; propietario de recurso validado en backend.
- Cookies HttpOnly, Secure, SameSite para sesiones web o estrategia de tokens segura; documentar decisión.
- CORS restringido a dominios propios.
- TLS en tránsito y cifrado en reposo.
- Secretos en AWS Secrets Manager/Parameter Store, no en Git ni imagen Docker.
- No registrar tokens OAuth, contraseñas, información médica ni datos financieros completos.
- Auditoría de acciones críticas.
- Rate limiting, validación de entrada, límites de tamaño y protección contra CSRF/XSS/SQL injection.
- Backups cifrados y pruebas de restauración.
- Exportación y eliminación de datos del usuario.
- Aplicar minimización, finalidad, consentimiento y derechos de titulares conforme al marco colombiano de protección de datos personales. La Ley 1581 de 2012 contiene reglas generales; revisar asesoría jurídica antes de tratar datos sensibles o abrir el producto a terceros.
- Separar datos de salud y documentos sensibles con permisos más restrictivos.

## 8. API REST orientativa

```text
POST   /api/v1/auth/register
POST   /api/v1/auth/login
GET    /api/v1/dashboard/today
GET    /api/v1/tasks
POST   /api/v1/tasks
PATCH  /api/v1/tasks/{id}
POST   /api/v1/tasks/{id}/complete
GET    /api/v1/routines
POST   /api/v1/routines
POST   /api/v1/routines/{id}/pause
GET    /api/v1/finance/transactions
POST   /api/v1/finance/transactions
GET    /api/v1/finance/budgets
GET    /api/v1/obligations
POST   /api/v1/obligations
GET    /api/v1/shopping-lists
POST   /api/v1/shopping-lists
GET    /api/v1/calendar/google/connect
GET    /api/v1/calendar/google/callback
POST   /api/v1/calendar/sync
DELETE /api/v1/calendar/google/connection
GET    /api/v1/automation/rules
POST   /api/v1/automation/rules
GET    /api/v1/notifications
```

Todos los endpoints deben tener autorización, validación, respuestas consistentes y pruebas.

## 9. Observabilidad y calidad
- Spring Boot Actuator con health/readiness.
- Logs estructurados con correlation ID.
- Métricas de errores, latencia, jobs pendientes, sincronización y fallos.
- CloudWatch en AWS.
- Pruebas unitarias, integración con PostgreSQL real mediante Testcontainers y pruebas end-to-end de flujos críticos.
- CI: compilación, pruebas, análisis estático, escaneo de dependencias e imagen.
