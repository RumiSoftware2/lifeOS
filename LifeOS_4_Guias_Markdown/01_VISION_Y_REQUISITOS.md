# LifeOS — Visión, alcance y requisitos del producto

**Propósito:** construir un sistema personal y familiar de organización que reduzca la carga administrativa cotidiana, proteja el tiempo para familia y trabajo y convierta obligaciones repetitivas en rutinas, recordatorios y flujos verificables.

## 1. Principios del producto

1. **Menos carga mental:** una bandeja central para capturar pendientes sin tener que decidir inmediatamente dónde van.
2. **Automatizar con límites:** automatizar recordatorios, cálculos, recurrencias y preparación; exigir confirmación humana para movimientos de dinero, decisiones médicas, tributarias o de inversión.
3. **Calendario como ejecución:** las tareas con fecha/hora pueden sincronizarse con Google Calendar; el sistema conserva su propia base de datos como fuente de verdad.
4. **Privacidad por diseño:** minimizar datos, cifrar secretos, separar datos personales, financieros y de salud, y ofrecer exportación y eliminación.
5. **Contexto colombiano configurable:** COP, zona horaria `America/Bogota`, festivos y calendarios tributarios por jurisdicción. No asumir que todas las personas tienen las mismas obligaciones.
6. **Primero utilidad, luego complejidad:** empezar con un monolito modular bien separado; evolucionar a servicios independientes solo cuando exista una razón medible.

## 2. Áreas de vida que debe cubrir

### 2.1 Finanzas personales y familiares
- Registro manual/importación de ingresos, gastos, transferencias y deudas.
- Presupuesto mensual por categorías: vivienda, alimentación, transporte, servicios, salud, mascotas, educación, ocio, ahorro e inversión.
- Flujo de caja previsto y real; comparación presupuesto vs. ejecución.
- Metas de ahorro con objetivo, fecha, aportes y progreso.
- Deudas: saldo, tasa, cuota, fecha de pago y prioridad.
- Suscripciones y pagos recurrentes: importe esperado, proveedor, periodicidad, fecha de renovación y aviso previo.
- Cierre financiero semanal/mensual.
- Adjuntar comprobantes y clasificar gastos.
- No almacenar credenciales bancarias ni ejecutar transferencias en el MVP.

### 2.2 Hogar, aseo y mantenimiento
Crear rutinas editables por frecuencia, duración, responsable, última ejecución y próxima fecha:
- Diario: ordenar superficies, lavar loza, sacar residuos según calendario local, revisar comida/agua de mascota.
- Semanal: baños, pisos, cocina, cambio de sábanas, limpieza de nevera superficial, mercado y revisión de inventario.
- Quincenal/mensual: lavar cobijas, toallas, cortinas según necesidad; limpiar electrodomésticos, filtros y zonas de difícil acceso.
- Trimestral/semestral: mantenimiento preventivo, revisión de botiquín, vencimientos y limpieza profunda.
- Registrar duración real para ajustar la planificación.
- Listas de chequeo por habitación y tareas que pueden delegarse o compartirse.

No imponer frecuencias sanitarias universales: permitir personalizar según número de habitantes, mascotas, clima, uso y recomendaciones del fabricante.

### 2.3 Compras e inventario del hogar
- Catálogo de productos: categoría, unidad, cantidad disponible, cantidad mínima, ubicación y proveedor preferido.
- Lista de compras automática al alcanzar stock mínimo.
- Lista compartida para hogar/familia.
- Plan de mercado semanal/quincenal, presupuesto estimado y comparación estimado/real.
- Productos recurrentes, tamaños, marcas opcionales y sustitutos.
- Alertas de vencimiento para alimentos, medicamentos y productos del hogar.
- Separar consumibles, despensa, aseo, farmacia y artículos de mascotas.
- No comprar automáticamente sin aprobación explícita.

### 2.4 Impuestos y obligaciones administrativas en Colombia
- Catálogo configurable de obligaciones: renta DIAN si aplica, predial y vehículo según municipio, impuesto de vehículos, SOAT, revisión técnico-mecánica, seguros, matrículas y renovaciones.
- Cada obligación debe almacenar entidad, jurisdicción, periodo gravable, fecha límite, fuente oficial, fecha de última verificación, estado, importe estimado/real y comprobante.
- Alertas escalonadas: 30, 15, 7, 3 y 1 día antes, configurables.
- Adjuntar documentos y comprobantes de pago.
- Panel de obligaciones próximas y vencidas.
- El sistema no determina por sí solo si alguien está obligado a declarar renta. Debe mostrar criterios informativos, pedir validación del usuario y enlazar a DIAN/asesor.
- Fechas tributarias deben ser versionadas por año y verificadas en fuente oficial. Nunca reutilizar fechas de años anteriores como si fueran vigentes.
- Bogotá: contemplar Secretaría Distrital de Hacienda para predial/vehículos cuando aplique; otras ciudades deben poder configurarse.

### 2.5 Salud y bienestar
- Agenda de citas médicas, odontología, controles, exámenes y seguimientos.
- Recordatorios de citas, preparación y documentos necesarios.
- Registro opcional de medicamentos: nombre, pauta prescrita por profesional, horario, duración y recordatorio. No recomendar dosis ni cambiar tratamientos.
- Registro de vacunas y controles preventivos, con fuente y fecha.
- Hábitos: sueño, hidratación, pausas activas, alimentación y actividad física.
- Actividad física como planificación personal: la guía general de Minsalud para adultos recomienda acumular al menos 150 minutos semanales de actividad moderada o 75 de vigorosa, y fortalecimiento muscular al menos dos veces por semana; adaptar siempre a la condición individual y orientación profesional.
- Datos de salud son altamente sensibles: permitir desactivar el módulo, limitar campos y no compartirlos con servicios externos sin consentimiento explícito.
- No diagnosticar, prescribir ni reemplazar a profesionales de salud.

### 2.6 Mascotas
- Perfil por mascota: especie, nombre, fecha de nacimiento aproximada, veterinario y contactos de emergencia.
- Calendario de vacunas, desparasitación, baños, peluquería, controles y medicamentos prescritos.
- Inventario de alimento y alertas de reposición.
- Rutinas: paseos, limpieza de cama, lavado de cobijas/toallas, higiene de recipientes.
- Historial de citas, gastos y documentos.
- No recomendar tratamientos ni dosis veterinarias.

### 2.7 Familia, trabajo y estudio
- Calendario de compromisos familiares, cumpleaños, actividades y tiempo de calidad.
- Bloques protegidos de concentración y descanso.
- Tareas laborales/estudio con proyecto, prioridad, fecha y duración estimada.
- Vista semanal que evite sobrecargar los días.
- Revisión semanal de 20–30 minutos para priorizar y reprogramar.
- Permitir compartir únicamente calendarios/listas seleccionados; no exponer finanzas o salud por defecto.

### 2.8 Documentos, garantías y renovaciones
- Registro de documento, entidad, fecha de emisión, vencimiento, ubicación segura y recordatorio.
- Garantías, pólizas, contratos, licencias, pasaportes y documentos del vehículo.
- No almacenar números completos de tarjetas, claves ni documentos innecesarios.
- Archivos privados con control de acceso y política de retención.

## 3. Automatizaciones recomendadas

| Automatización | Disparador | Acción automática | Control |
|---|---|---|---|
| Recibo de servicio | Fecha de vencimiento | Crear recordatorios y tarea de pago | Usuario confirma pago |
| Lavado de cobijas/toallas | Regla recurrente | Programar tarea y Calendar | Usuario marca completada |
| Mercado | Día de planificación o stock mínimo | Generar lista agrupada | Usuario revisa y compra |
| Alimento de mascota | Stock bajo/fecha prevista | Crear compra sugerida | Aprobación manual |
| Cita médica | Fecha/hora de cita | Recordatorios y checklist | Usuario confirma asistencia |
| Medicamento | Horario configurado | Notificación | No cambia prescripción |
| Impuestos | Fecha oficial verificada | Alertas escalonadas | Usuario valida obligación y paga |
| Presupuesto | Fin de semana/mes | Resumen de gastos y desviaciones | Usuario revisa |
| Ahorro | Día de ingreso | Recordatorio de aporte/meta | Transferencia manual |
| Inversiones | Fecha de revisión | Recordatorio de revisar estrategia | Nunca ejecutar órdenes automáticamente |
| Mantenimiento | Intervalo o lectura de uso | Crear tarea | Usuario confirma ejecución |
| Revisión semanal | Día/hora elegido | Agenda de revisión | Usuario decide prioridades |

## 4. Experiencia de usuario

### Pantallas principales
1. **Hoy:** agenda, tareas del día, pagos próximos y alertas importantes.
2. **Semana:** calendario visual con bloques de trabajo, hogar, familia y descanso.
3. **Bandeja de entrada:** captura rápida de cualquier pendiente.
4. **Hogar:** rutinas, inventario, compras y mantenimiento.
5. **Finanzas:** presupuesto, movimientos, deudas, metas y obligaciones.
6. **Salud:** citas, hábitos y recordatorios.
7. **Mascotas:** cuidados, citas y suministros.
8. **Documentos:** vencimientos y archivos.
9. **Automatizaciones:** reglas, recurrencias, historial y fallos.
10. **Configuración:** zona horaria, moneda, Google, permisos, notificaciones y exportación.

### Reglas de diseño
- Responsive: móvil primero, tablet y escritorio.
- Interfaz clara, sobria, accesible y rápida.
- Mostrar prioridades sin convertir todo en una alerta.
- Estados explícitos: pendiente, programada, completada, pospuesta, cancelada, vencida.
- Cada tarea recurrente debe poder saltarse una ocurrencia, cambiar su frecuencia o pausarse.
- Mostrar la razón de cada recordatorio y permitir desactivarlo.

## 5. MVP y evolución

### MVP obligatorio
- Autenticación segura de un usuario.
- Dashboard Hoy y vista semanal.
- Tareas y rutinas recurrentes.
- Hogar: aseo, inventario básico y lista de compras.
- Pagos/obligaciones con recordatorios.
- Finanzas manuales: ingresos, gastos, presupuesto y metas.
- Integración OAuth con Google Calendar: crear, actualizar y cancelar eventos generados por LifeOS.
- Notificaciones dentro de la aplicación y correo opcional.
- Zona horaria `America/Bogota`.
- Auditoría básica, exportación y backup.

### Segunda etapa
- Salud, mascotas, documentos, deuda, suscripciones y planificación familiar.
- Reglas de automatización configurables.
- Notificaciones push/email.
- Importación CSV de extractos (sin almacenar credenciales bancarias).
- Informes mensuales.

### Etapa avanzada
- Integración bancaria solo si existe proveedor autorizado y revisión de seguridad/legal.
- OCR de facturas y recibos con revisión humana.
- Asistente inteligente para proponer agenda y detectar conflictos.
- Multiusuario/familia con roles y permisos granulares.
- Infraestructura cloud escalable y recuperación ante desastres.

## 6. Criterios de éxito
- El usuario puede ver en menos de 30 segundos qué requiere atención hoy.
- Las tareas recurrentes no se duplican al sincronizar Calendar.
- Toda obligación tiene fuente, fecha de verificación y estado.
- Ningún pago, inversión o cambio médico se ejecuta sin aprobación humana.
- Se puede exportar la información y recuperar un backup.
- El usuario puede pausar cualquier automatización.
