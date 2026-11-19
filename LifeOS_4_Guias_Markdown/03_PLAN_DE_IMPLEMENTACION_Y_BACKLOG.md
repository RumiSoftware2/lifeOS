# LifeOS — Plan de implementación, backlog y criterios de aceptación

## 1. Método de trabajo

Desarrollar por incrementos verticales: cada fase debe entregar una funcionalidad utilizable de frontend a base de datos. No construir todos los modelos primero ni generar cientos de archivos sin probar.

Antes de modificar el repositorio:
1. Inspeccionar el estado actual, rama, estructura y README.
2. Confirmar URL remota y rama principal. El usuario debe proporcionar la URL del repositorio GitHub si no está configurada.
3. Crear `docs/`, ADRs y estructura mínima.
4. Presentar plan y decisiones antes de cambios estructurales grandes.
5. Implementar una fase, ejecutar pruebas y documentar.
6. No borrar ni sobrescribir archivos existentes sin autorización.

## 2. Fases

### Fase 0 — Descubrimiento y base técnica
- Verificar Java instalado, Maven/Gradle y Node.
- Elegir Maven inicialmente salvo razón justificada.
- Crear backend Spring Boot y frontend React con JavaScript.
- Configurar Docker Compose para PostgreSQL.
- Configurar `.env.example`, `.gitignore`, README y health check.
- Agregar Flyway y OpenAPI.
- Primer pipeline CI en GitHub Actions.

**Aceptación:** `docker compose up` levanta PostgreSQL; backend inicia, frontend carga y endpoint health responde; tests básicos pasan.

### Fase 1 — Identidad, tareas y dashboard
- Registro/login.
- Perfil y zona horaria.
- CRUD de tareas, categorías, prioridades y estados.
- Dashboard Hoy y vista semanal.
- Bandeja de entrada y filtros.
- Protección de endpoints y aislamiento por usuario.

**Aceptación:** un usuario no puede leer ni modificar tareas de otro; tareas aparecen ordenadas por vencimiento; validaciones y pruebas cubren casos normales y errores.

### Fase 2 — Rutinas y automatización doméstica
- Plantillas de aseo.
- Recurrencias y ocurrencias.
- Acciones completar, posponer, omitir y pausar.
- Inventario básico y lista de mercado.
- Rutinas de mascotas.

**Aceptación:** una regla semanal genera una sola ocurrencia por semana; reiniciar el backend no duplica tareas; cambiar una regla futura no altera historial completado.

### Fase 3 — Finanzas y obligaciones
- Ingresos/gastos manuales.
- Categorías y presupuestos.
- Metas de ahorro y deudas.
- Servicios recurrentes y vencimientos.
- Obligaciones tributarias configurables con fuente oficial y año.
- Comprobantes y cierre mensual.

**Aceptación:** cálculos con `BigDecimal`; el saldo y presupuesto se calculan desde movimientos; fechas de impuestos requieren jurisdicción, año y fuente; no se realizan pagos automáticos.

### Fase 4 — Google Calendar
- Configurar Google Cloud project, habilitar Calendar API y OAuth consent screen.
- Implementar conexión y desconexión.
- Crear/actualizar/cancelar eventos asociados a tareas.
- Registro de sincronización, idempotencia y reintentos.
- Selector de qué categorías sincronizar.

**Aceptación:** conectar cuenta, crear una tarea programada y verla en Google Calendar; editarla actualiza el evento; cancelar la tarea cancela el evento; reintento no duplica.

### Fase 5 — Salud, familia y documentos
- Citas y recordatorios.
- Hábitos y ejercicio.
- Datos de mascota ampliados.
- Documentos y vencimientos.
- Permisos granulares para miembros del hogar.
- Exportación de datos.

**Aceptación:** consentimiento explícito para datos sensibles; el usuario puede exportar y eliminar información; salud no se comparte por defecto.

### Fase 6 — Infraestructura AWS
- Construir imagen Docker multi-stage.
- Publicar imagen en Amazon ECR.
- Desplegar backend en ECS Fargate.
- RDS PostgreSQL en subred privada.
- Application Load Balancer y HTTPS.
- Secrets Manager, CloudWatch, backups y alertas.
- Frontend servido por S3 + CloudFront o integrado detrás de CDN.
- Infraestructura como código con Terraform o AWS CDK (elegir uno; Terraform facilita aprendizaje multi-cloud).

**Aceptación:** despliegue reproducible desde CI/CD; base de datos no expuesta a Internet; secretos fuera de repositorio; backup y restauración documentados.

## 3. Backlog priorizado

### P0 — imprescindible
- Autenticación y autorización.
- Tareas y recurrencias.
- Vista Hoy/Semana.
- Rutinas del hogar.
- Obligaciones/pagos recordables.
- Finanzas manuales y presupuesto.
- Google Calendar unidireccional.
- Backup, exportación y seguridad básica.

### P1 — valor alto
- Inventario y mercado.
- Mascotas.
- Suscripciones.
- Notificaciones email/push.
- Informes mensuales.
- Documentos y garantías.
- Salud y citas.

### P2 — avanzado
- OCR de recibos con confirmación.
- Integraciones financieras autorizadas.
- Recomendaciones inteligentes de agenda.
- Multiusuario con roles.
- Motor de reglas más sofisticado.
- Cola distribuida y servicios desacoplados.

## 4. Casos de uso que deben probarse

1. Crear tarea “Lavar toallas” cada sábado.
2. Posponer una ocurrencia sin cambiar todas las siguientes.
3. Marcar una tarea completada y registrar cuándo.
4. Crear “Pagar internet” con vencimiento y avisos previos.
5. Crear lista de mercado y descontar inventario manualmente.
6. Registrar gasto en COP y comprobar presupuesto.
7. Registrar fecha de vencimiento de SOAT y recibir recordatorio.
8. Registrar obligación tributaria solo tras verificar su aplicabilidad.
9. Conectar Google Calendar, crear evento y sincronizar actualización.
10. Revocar Google OAuth y comprobar que el sistema deja de sincronizar.
11. Reiniciar backend y confirmar que las reglas no generan duplicados.
12. Restaurar backup en entorno separado.
13. Intentar consultar un recurso de otro usuario y recibir 403/404.
14. Desactivar automatización y comprobar que no genera nuevas tareas.

## 5. Definición de terminado

Una historia está terminada cuando:
- El flujo funciona en UI responsive.
- Backend valida y autoriza.
- Migración Flyway incluida.
- Pruebas relevantes pasan.
- Errores se muestran claramente.
- No hay secretos en código.
- README/API docs actualizados.
- Se registra decisión arquitectónica si hay trade-off importante.
- El usuario entiende cómo ejecutar y verificar la funcionalidad.

## 6. Datos iniciales de ejemplo

Permitir cargar datos demo opcionales, claramente marcados:
- Rutina semanal: limpiar baño.
- Rutina semanal: cambiar sábanas.
- Rutina quincenal: lavar toallas.
- Rutina mensual: lavar cobijas.
- Pago mensual: servicio de internet.
- Pago mensual: energía/agua/gas según servicios reales del hogar.
- Revisión semanal: lista de mercado.
- Mascota: alimento, paseo, baño y control veterinario.
- Salud: cita de ejemplo sin datos médicos reales.

No asumir días, importes, fechas, nombres de proveedores ni obligaciones reales del usuario. El onboarding debe pedirlos o permitir editarlos.
