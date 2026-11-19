# LifeOS — Cloud, DevOps, seguridad y despliegue profesional

## 1. Objetivo de aprendizaje

Aprender un despliegue más profesional que Render/Vercel sin sobredimensionar el proyecto. Ruta recomendada: AWS con contenedores, red privada, base de datos administrada, secretos, observabilidad e infraestructura como código.

## 2. Arquitectura AWS propuesta

```text
                         Internet
                            |
                    Route 53 (DNS)
                            |
                 CloudFront + HTTPS
                    /             \
            Frontend estático    /api/*
             S3 privado             |
                              Application Load Balancer
                                      |
                                ECS Fargate
                             Spring Boot container
                                      |
                    +-----------------+----------------+
                    |                                  |
             RDS PostgreSQL                       Secrets Manager
             private subnets                     OAuth/DB secrets
                    |
              Automated backups

ECS / ALB / RDS --> CloudWatch Logs, Metrics, Alarms
GitHub Actions --> ECR --> ECS deployment
Terraform --> VPC, subnets, security groups, ALB, ECS, RDS, IAM
```

### Componentes
- **Amazon VPC:** red virtual propia.
- **Subredes públicas:** ALB y NAT Gateway si se necesita salida controlada.
- **Subredes privadas:** tareas ECS y RDS.
- **ECS Fargate:** ejecutar contenedor sin administrar servidores EC2.
- **ECR:** repositorio privado de imágenes Docker.
- **RDS PostgreSQL:** base de datos administrada, backups y mantenimiento.
- **ALB:** entrada HTTPS y health checks.
- **ACM:** certificados TLS.
- **Route 53:** DNS si se compra/configura dominio.
- **S3 + CloudFront:** frontend estático y distribución CDN.
- **Secrets Manager:** credenciales DB y refresh tokens OAuth cifrados.
- **CloudWatch:** logs, métricas, alarmas y dashboards.
- **IAM:** permisos mínimos para cada servicio.
- **AWS Backup/RDS snapshots:** estrategia de recuperación.

## 3. ¿Por qué no empezar con Kubernetes?

Kubernetes/EKS es valioso para aprender orquestación avanzada, pero para una aplicación personal con un backend es costoso y agrega carga operativa. Primero dominar:
1. Docker y Docker Compose.
2. Redes, volúmenes y health checks.
3. Imágenes multi-stage y escaneo.
4. ECR + ECS Fargate.
5. RDS, IAM, VPC y observabilidad.
6. Terraform y CI/CD.
7. Luego desplegar una segunda versión en EKS como laboratorio opcional.

No desplegar Kubernetes solo por parecer más profesional. Una arquitectura simple, segura, automatizada y operable es más profesional que una compleja sin necesidad.

## 4. Estrategia de costes

AWS puede generar cobros por recursos activos, transferencia, NAT Gateway, RDS, balanceadores, logs y almacenamiento. No prometer que el despliegue será gratuito.

Para aprender con presupuesto controlado:
- Definir región cercana y revisar disponibilidad/precios antes de crear recursos.
- Configurar AWS Budgets y alertas de coste antes del despliegue.
- Usar tamaños mínimos compatibles y apagar entornos no productivos.
- Evaluar RDS single-AZ para laboratorio, entendiendo el riesgo.
- Evitar NAT Gateway si el patrón de red puede resolverse de manera segura y más económica; no sacrificar seguridad para ahorrar.
- Destruir recursos de laboratorio con Terraform cuando no se utilicen.
- Mantener backups y exportar datos antes de destruir recursos.
- Usar calculadora oficial AWS Pricing Calculator.
- No publicar base de datos directamente a Internet.

## 5. Docker local

Servicios locales:
- `frontend`: React.
- `backend`: Spring Boot.
- `postgres`: PostgreSQL.
- Opcional `mailpit`: probar correos localmente.

Requisitos:
- Dockerfile multi-stage para backend.
- Dockerfile de frontend solo si se sirve desde Nginx en local.
- Health checks.
- Volumen persistente para PostgreSQL.
- Red interna.
- Variables en `.env` ignorado por Git.
- `.env.example` sin secretos.
- Migraciones automáticas controladas al arrancar.

## 6. CI/CD con GitHub Actions

Pipeline propuesto:
1. Checkout.
2. Configurar Java y Node.
3. Ejecutar pruebas backend/frontend.
4. Ejecutar análisis estático y auditoría de dependencias.
5. Construir artefacto e imagen Docker.
6. Escanear imagen.
7. Autenticarse con AWS mediante OIDC (evitar claves AWS permanentes).
8. Publicar imagen en ECR.
9. Desplegar nueva revisión de ECS.
10. Esperar health checks y verificar endpoint.
11. Si falla, detener promoción y documentar rollback.

Separar ambientes `dev` y `prod` con cuentas/recursos y permisos diferenciados cuando el proyecto crezca.

## 7. Seguridad operativa
- MFA para cuenta raíz AWS y no usar root para tareas diarias.
- IAM roles con mínimo privilegio.
- OIDC GitHub Actions a AWS.
- RDS privado y security groups restrictivos.
- Cifrado TLS y cifrado de almacenamiento.
- Secrets Manager para secretos.
- Rotación de credenciales.
- Logs sin datos sensibles.
- WAF opcional cuando haya exposición y riesgo que lo justifique.
- Escaneo de dependencias e imágenes.
- Parches y actualizaciones programadas.
- Backups automáticos y prueba de restauración.
- Alertas de errores, CPU/memoria, conexiones DB, jobs fallidos y costes.
- Plan de respuesta ante pérdida de cuenta OAuth o filtración de secreto.

## 8. Recuperación y continuidad
Definir:
- RPO: cuánto dato se puede perder.
- RTO: cuánto tiempo se acepta estar fuera de servicio.
- Frecuencia y retención de backups.
- Prueba trimestral de restauración.
- Exportación manual de datos.
- Procedimiento para rotar secretos.
- Procedimiento para revocar Google OAuth.
- Runbook de despliegue y rollback.

## 9. Alternativas de despliegue para comparar

| Alternativa | Qué aprendes | Cuándo usar |
|---|---|---|
| AWS ECS Fargate + RDS | Contenedores administrados, VPC, IAM, ALB, observabilidad | Recomendado para este proyecto |
| AWS EC2 + Docker Compose | Linux, SSH, firewall, reverse proxy, operación de VM | Laboratorio de administración de servidores |
| AWS EKS | Kubernetes, pods, deployments, ingress, autoscaling | Etapa avanzada/laboratorio |
| VPS + Docker + Caddy | Linux, DNS, TLS, firewall y contenedores | Alternativa económica, más operación propia |

No se necesita VPN para que el usuario acceda a su aplicación pública. Una VPN puede servir para administración privada o acceso a recursos internos, pero no debe agregarse por moda. Para administrar AWS, priorizar IAM, MFA, SSM Session Manager y redes privadas.

## 10. Fuentes oficiales de aprendizaje
- AWS Well-Architected Framework: https://docs.aws.amazon.com/wellarchitected/latest/userguide/waf.html
- AWS Well-Architected — pilares: https://docs.aws.amazon.com/wellarchitected/latest/framework/the-pillars-of-the-framework.html
- AWS ECS: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html
- AWS Fargate: https://docs.aws.amazon.com/AmazonECS/latest/userguide/what-is-fargate.html
- Amazon RDS for PostgreSQL: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html
- AWS Secrets Manager: https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html
- GitHub Actions OIDC con AWS: https://docs.github.com/en/actions/security-for-github-actions/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services
- Terraform AWS provider: https://registry.terraform.io/providers/hashicorp/aws/latest/docs
- Google Calendar API — crear eventos: https://developers.google.com/workspace/calendar/api/guides/create-events
- Google Calendar API — autorización: https://developers.google.com/workspace/calendar/api/auth

## 11. Fuentes de contexto colombiano
- DIAN — calendario tributario 2026: https://www.dian.gov.co/Calendarios/Calendario_Tributario_2026.pdf
- Secretaría Distrital de Hacienda de Bogotá: https://www.haciendabogota.gov.co/
- Minsalud — actividad física: https://www.minsalud.gov.co/salud/publica/HS/Paginas/actividad-fisica.aspx
- Fondo Nacional del Ahorro — finanzas personales: https://www.fna.gov.co/educacion-financiera/finanzas-personales
- Superintendencia Financiera — ahorro e inversión: https://www.superfinanciera.gov.co/
- Ley 1581 de 2012 — protección de datos personales: https://www.suin-juriscol.gov.co/

Las normas, fechas, umbrales y calendarios deben comprobarse nuevamente en los portales oficiales al implementar cada funcionalidad. Esta guía no es asesoría tributaria, financiera, médica ni legal.
