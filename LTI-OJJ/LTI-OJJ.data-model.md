# LTI - Modelo de Datos - Documentación Completa

## 📊 Visión General del Modelo

El modelo de datos de LTI está diseñado siguiendo principios de:
- ✅ **Normalización** para evitar redundancia
- ✅ **Escalabilidad** para soportar millones de registros
- ✅ **Multi-tenancy** (cada organización aislada)
- ✅ **Auditabilidad** (timestamps y tracking de cambios)
- ✅ **GDPR compliance** (consentimientos y derecho al olvido)

---

## 🏢 ENTIDADES PRINCIPALES

### 1. ORGANIZATION (Organización)

**Descripción**: Representa cada empresa cliente de LTI (multi-tenancy).

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `name` | STRING | Nombre de la organización |
| `slug` | STRING | URL-friendly identifier (ej: "acme-corp") |
| `industry` | STRING | Sector/industria |
| `employee_count` | INTEGER | Tamaño de la empresa |
| `website` | STRING | URL del sitio web |
| `billing_info` | JSON | Datos de facturación y pago |
| `subscription_plan` | STRING | Plan contratado (Starter/Pro/Enterprise) |
| `created_at` | DATETIME | Fecha de creación |
| `updated_at` | DATETIME | Última actualización |
| `is_active` | BOOLEAN | Estado de la cuenta |

**Relaciones**:
- 1:N con USER (una org tiene muchos usuarios)
- 1:N con JOB_POSTING (una org crea muchas vacantes)
- 1:N con DEPARTMENT (una org tiene múltiples departamentos)

---

### 2. USER (Usuario)

**Descripción**: Usuarios del sistema (reclutadores, hiring managers, admins).

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `organization_id` | UUID | Organización a la que pertenece (FK) |
| `department_id` | UUID | Departamento al que pertenece (FK) |
| `email` | STRING | Email (único por organización) |
| `password_hash` | STRING | Contraseña encriptada |
| `first_name` | STRING | Nombre |
| `last_name` | STRING | Apellido |
| `role` | STRING | Rol (admin, recruiter, hiring_manager, interviewer) |
| `avatar_url` | STRING | URL de foto de perfil |
| `permissions` | JSON | Permisos granulares |
| `preferences` | JSON | Configuración personal (idioma, notificaciones) |
| `last_login` | DATETIME | Último acceso |
| `created_at` | DATETIME | Fecha de creación |
| `updated_at` | DATETIME | Última actualización |
| `is_active` | BOOLEAN | Estado del usuario |

**Relaciones**:
- N:1 con ORGANIZATION
- N:1 con DEPARTMENT
- 1:N con JOB_POSTING (crea vacantes)
- 1:N con INTERVIEW (participa en entrevistas)
- 1:N con COMMENT (escribe comentarios)

---

### 3. DEPARTMENT (Departamento)

**Descripción**: Departamentos dentro de una organización.

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `organization_id` | UUID | Organización (FK) |
| `name` | STRING | Nombre del departamento (ej: "Engineering") |
| `description` | STRING | Descripción |
| `manager_id` | UUID | Usuario responsable del departamento (FK) |
| `created_at` | DATETIME | Fecha de creación |
| `updated_at` | DATETIME | Última actualización |

**Relaciones**:
- N:1 con ORGANIZATION
- 1:N con USER
- 1:N con JOB_POSTING

---

### 4. JOB_POSTING (Vacante/Oferta de Empleo)

**Descripción**: Posiciones abiertas que la empresa quiere cubrir.

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `organization_id` | UUID | Organización (FK) |
| `department_id` | UUID | Departamento (FK) |
| `created_by` | UUID | Usuario que creó la vacante (FK) |
| `title` | STRING | Título del puesto |
| `description` | TEXT | Descripción detallada |
| `requirements` | TEXT | Requisitos y cualificaciones |
| `responsibilities` | TEXT | Responsabilidades del rol |
| `employment_type` | STRING | Tipo (full_time, part_time, contract, internship) |
| `location` | STRING | Ubicación del puesto |
| `remote_policy` | STRING | Política remoto (on_site, hybrid, remote) |
| `salary_range` | JSON | Rango salarial {min, max, currency, period} |
| `status` | STRING | Estado (draft, published, closed, on_hold) |
| `custom_fields` | JSON | Campos personalizados por organización |
| `published_at` | DATETIME | Fecha de publicación |
| `closes_at` | DATETIME | Fecha de cierre |
| `created_at` | DATETIME | Fecha de creación |
| `updated_at` | DATETIME | Última actualización |

**Relaciones**:
- N:1 con ORGANIZATION
- N:1 con DEPARTMENT
- N:1 con USER (created_by)
- 1:N con APPLICATION
- 1:N con PIPELINE_STAGE
- 1:N con INTERVIEW_STAGE

---

### 5. CANDIDATE (Candidato)

**Descripción**: Personas que aplican a vacantes. Un candidato puede aplicar a múltiples vacantes.

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `email` | STRING | Email del candidato (único global) |
| `phone` | STRING | Teléfono |
| `first_name` | STRING | Nombre |
| `last_name` | STRING | Apellido |
| `linkedin_url` | STRING | Perfil de LinkedIn |
| `portfolio_url` | STRING | Portfolio/website personal |
| `resume_text` | TEXT | CV parseado como texto plano |
| `resume_parsed` | JSON | Datos estructurados del CV (skills, experiencia, educación) |
| `current_company` | STRING | Empresa actual |
| `current_title` | STRING | Título actual |
| `years_experience` | INTEGER | Años de experiencia |
| `skills` | JSON | Array de skills |
| `languages` | JSON | Idiomas y niveles |
| `location` | STRING | Ubicación actual |
| `willing_relocate` | BOOLEAN | Dispuesto a relocalizarse |
| `salary_expectations` | JSON | Expectativas salariales |
| `avatar_url` | STRING | Foto de perfil |
| `created_at` | DATETIME | Primera aplicación |
| `updated_at` | DATETIME | Última actualización |
| `gdpr_consent` | BOOLEAN | Consentimiento GDPR |
| `gdpr_consent_date` | DATETIME | Fecha del consentimiento |

**Relaciones**:
- 1:N con APPLICATION (un candidato puede aplicar a múltiples vacantes)
- 1:N con CANDIDATE_TAG

---

### 6. APPLICATION (Candidatura/Aplicación)

**Descripción**: Relación entre un CANDIDATE y un JOB_POSTING. Es la entidad central del ATS.

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `candidate_id` | UUID | Candidato que aplica (FK) |
| `job_posting_id` | UUID | Vacante a la que aplica (FK) |
| `stage_id` | UUID | Etapa actual del pipeline (FK) |
| `source` | STRING | Canal de origen (linkedin, referral, indeed, website) |
| `ai_score` | INTEGER | Puntuación de 0-100 generada por IA |
| `ai_analysis` | JSON | Análisis detallado de la IA (match de skills, experiencia) |
| `cover_letter` | TEXT | Carta de presentación |
| `status` | STRING | Estado (new, reviewing, interview, offer, hired, rejected) |
| `custom_answers` | JSON | Respuestas a preguntas personalizadas |
| `applied_at` | DATETIME | Fecha de aplicación |
| `last_activity` | DATETIME | Última actividad registrada |
| `created_at` | DATETIME | Fecha de creación |
| `updated_at` | DATETIME | Última actualización |
| `assigned_to` | UUID | Reclutador asignado (FK) |

**Relaciones**:
- N:1 con CANDIDATE
- N:1 con JOB_POSTING
- N:1 con PIPELINE_STAGE
- N:1 con USER (assigned_to)
- 1:N con INTERVIEW
- 1:N con EVALUATION
- 1:N con ACTIVITY
- 1:N con DOCUMENT
- 1:N con COMMENT

---

### 7. PIPELINE_STAGE (Etapa del Pipeline)

**Descripción**: Etapas configurables del proceso de selección (ej: "Nuevo", "Screening", "Entrevista Técnica", "Oferta").

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `job_posting_id` | UUID | Vacante a la que pertenece (FK) |
| `name` | STRING | Nombre de la etapa |
| `type` | STRING | Tipo (applied, screening, interview, offer, hired, rejected) |
| `order_index` | INTEGER | Orden en el pipeline |
| `automation_rules` | JSON | Reglas de automatización al entrar/salir |
| `requires_evaluation` | BOOLEAN | Requiere evaluación formal |
| `created_at` | DATETIME | Fecha de creación |
| `updated_at` | DATETIME | Última actualización |

**Relaciones**:
- N:1 con JOB_POSTING
- 1:N con APPLICATION

---

### 8. INTERVIEW_STAGE (Tipo de Entrevista)

**Descripción**: Template de entrevistas configuradas para una vacante (ej: "Screening de 30 min", "Técnica de 60 min").

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `job_posting_id` | UUID | Vacante (FK) |
| `name` | STRING | Nombre del tipo de entrevista |
| `type` | STRING | Tipo (phone, video, onsite, technical, behavioral) |
| `duration_minutes` | INTEGER | Duración estimada |
| `order_index` | INTEGER | Orden en el proceso |
| `interview_guide` | JSON | Guía de preguntas sugeridas |
| `scorecard_template` | JSON | Template de evaluación |
| `created_at` | DATETIME | Fecha de creación |
| `updated_at` | DATETIME | Última actualización |

**Relaciones**:
- N:1 con JOB_POSTING
- 1:N con INTERVIEW

---

### 9. INTERVIEW (Entrevista)

**Descripción**: Entrevistas específicas programadas con candidatos.

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `application_id` | UUID | Candidatura (FK) |
| `interview_stage_id` | UUID | Tipo de entrevista (FK) |
| `scheduled_at` | DATETIME | Fecha y hora programada |
| `duration_minutes` | INTEGER | Duración |
| `location` | STRING | Ubicación (dirección física o "remote") |
| `video_url` | STRING | Link de videoconferencia (Zoom, Meet, Teams) |
| `status` | STRING | Estado (scheduled, completed, cancelled, no_show) |
| `notes` | TEXT | Notas generales de la entrevista |
| `recording_url` | JSON | URL de grabación (si aplica) |
| `started_at` | DATETIME | Hora real de inicio |
| `completed_at` | DATETIME | Hora real de finalización |
| `created_at` | DATETIME | Fecha de creación |
| `updated_at` | DATETIME | Última actualización |

**Relaciones**:
- N:1 con APPLICATION
- N:1 con INTERVIEW_STAGE
- 1:N con INTERVIEW_PARTICIPANT
- 1:N con EVALUATION

---

### 10. INTERVIEW_PARTICIPANT (Participante de Entrevista)

**Descripción**: Relación many-to-many entre INTERVIEW y USER (múltiples entrevistadores por entrevista).

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `interview_id` | UUID | Entrevista (FK) |
| `user_id` | UUID | Usuario entrevistador (FK) |
| `role` | STRING | Rol (interviewer, observer, organizer) |
| `is_required` | BOOLEAN | Si su asistencia es obligatoria |
| `status` | STRING | Estado (pending, accepted, declined, attended) |
| `confirmed_at` | DATETIME | Fecha de confirmación |

**Relaciones**:
- N:1 con INTERVIEW
- N:1 con USER

---

### 11. EVALUATION (Evaluación)

**Descripción**: Evaluaciones y feedback de entrevistadores sobre candidatos.

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `application_id` | UUID | Candidatura evaluada (FK) |
| `interview_id` | UUID | Entrevista relacionada (FK, nullable) |
| `evaluator_id` | UUID | Usuario que evalúa (FK) |
| `scores` | JSON | Puntuaciones por criterio {technical: 4, communication: 5, ...} |
| `strengths` | TEXT | Fortalezas observadas |
| `weaknesses` | TEXT | Áreas de mejora |
| `overall_feedback` | TEXT | Comentarios generales |
| `recommendation` | STRING | Recomendación (strong_yes, yes, maybe, no, strong_no) |
| `is_submitted` | BOOLEAN | Si fue enviada o está en borrador |
| `submitted_at` | DATETIME | Fecha de envío |
| `created_at` | DATETIME | Fecha de creación |
| `updated_at` | DATETIME | Última actualización |

**Relaciones**:
- N:1 con APPLICATION
- N:1 con INTERVIEW (opcional)
- N:1 con USER (evaluator)

---

### 12. ACTIVITY (Actividad/Timeline)

**Descripción**: Log de todas las acciones realizadas sobre una candidatura (auditoría y timeline).

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `application_id` | UUID | Candidatura (FK) |
| `user_id` | UUID | Usuario que realizó la acción (FK, nullable para acciones automáticas) |
| `activity_type` | STRING | Tipo (stage_changed, email_sent, interview_scheduled, note_added, etc.) |
| `description` | TEXT | Descripción legible de la actividad |
| `metadata` | JSON | Datos adicionales específicos del tipo |
| `created_at` | DATETIME | Fecha de la actividad |

**Relaciones**:
- N:1 con APPLICATION
- N:1 con USER (opcional)

---

### 13. COMMENT (Comentario)

**Descripción**: Comentarios internos del equipo sobre candidatos.

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `application_id` | UUID | Candidatura (FK) |
| `user_id` | UUID | Autor del comentario (FK) |
| `parent_id` | UUID | Comentario padre (para hilos/respuestas, FK nullable) |
| `content` | TEXT | Contenido del comentario |
| `is_internal` | BOOLEAN | Si es visible solo para el equipo |
| `created_at` | DATETIME | Fecha de creación |
| `updated_at` | DATETIME | Última edición |

**Relaciones**:
- N:1 con APPLICATION
- N:1 con USER
- N:1 con COMMENT (parent, auto-referencial)

---

### 14. DOCUMENT (Documento)

**Descripción**: Archivos adjuntos (CVs, portfolios, certificados, etc.).

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `application_id` | UUID | Candidatura (FK) |
| `uploaded_by` | UUID | Usuario que subió el archivo (FK, nullable si fue el candidato) |
| `file_name` | STRING | Nombre original del archivo |
| `file_type` | STRING | MIME type (application/pdf, image/jpeg, etc.) |
| `file_size` | INTEGER | Tamaño en bytes |
| `storage_url` | STRING | URL en storage (S3, GCS, etc.) |
| `document_type` | STRING | Tipo (resume, cover_letter, portfolio, certificate, other) |
| `is_public` | BOOLEAN | Si el candidato puede verlo |
| `created_at` | DATETIME | Fecha de subida |

**Relaciones**:
- N:1 con APPLICATION
- N:1 con USER (uploaded_by, opcional)

---

### 15. EMAIL_TEMPLATE (Plantilla de Email)

**Descripción**: Plantillas reutilizables para comunicación con candidatos.

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `organization_id` | UUID | Organización (FK) |
| `name` | STRING | Nombre descriptivo de la plantilla |
| `subject` | STRING | Asunto del email (puede incluir variables) |
| `html_body` | TEXT | Cuerpo en HTML |
| `text_body` | TEXT | Cuerpo en texto plano (fallback) |
| `variables` | JSON | Variables disponibles {candidate_name, job_title, ...} |
| `template_type` | STRING | Tipo (application_received, interview_invite, rejection, offer, etc.) |
| `is_active` | BOOLEAN | Si está activa |
| `created_at` | DATETIME | Fecha de creación |
| `updated_at` | DATETIME | Última actualización |

**Relaciones**:
- N:1 con ORGANIZATION
- 1:N con EMAIL_LOG

---

### 16. EMAIL_LOG (Registro de Emails)

**Descripción**: Registro de todos los emails enviados (tracking y auditoría).

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `application_id` | UUID | Candidatura (FK) |
| `template_id` | UUID | Plantilla usada (FK, nullable si fue custom) |
| `to_email` | STRING | Email del destinatario |
| `subject` | STRING | Asunto real enviado |
| `body` | TEXT | Cuerpo real enviado |
| `status` | STRING | Estado (queued, sent, delivered, bounced, failed, opened, clicked) |
| `sent_at` | DATETIME | Fecha de envío |
| `opened_at` | DATETIME | Primera apertura (tracking) |
| `clicked_at` | DATETIME | Primer click (tracking) |
| `metadata` | JSON | Datos adicionales (provider_id, error_message, etc.) |

**Relaciones**:
- N:1 con APPLICATION
- N:1 con EMAIL_TEMPLATE (opcional)

---

### 17. TAG (Etiqueta)

**Descripción**: Etiquetas personalizables para categorizar candidatos.

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |
| `organization_id` | UUID | Organización (FK) |
| `name` | STRING | Nombre de la etiqueta |
| `color` | STRING | Color en hex (#FF5733) |
| `category` | STRING | Categoría (skill, source, priority, custom) |
| `created_at` | DATETIME | Fecha de creación |

**Relaciones**:
- N:1 con ORGANIZATION
- 1:N con CANDIDATE_TAG

---

### 18. CANDIDATE_TAG (Candidato-Etiqueta)

**Descripción**: Relación many-to-many entre candidatos y etiquetas.

| Atributo | Tipo | Descripción |
|----------|------|-------------|
| `id` | UUID | Identificador único (PK) |