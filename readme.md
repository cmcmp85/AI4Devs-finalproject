# GastOS — Plataforma de Gestión de Gastos Internos 

> Proyecto Final AI4Devs — Entrega 1: Documentación técnica.

**Leyenda usada en este documento:**
- **[MVP]** → se documenta y **se implementa** en este proyecto.
- **[DOCUMENTADO]** → forma parte de la especificación completa del RFP y se documenta aquí para dar contexto de producto completo, pero **no se implementa** en el alcance de este proyecto (queda fuera, como referencia de lo que existiría en una versión completa).

## Índice

0. [Ficha del proyecto](#0-ficha-del-proyecto)
1. [Descripción general del producto](#1-descripción-general-del-producto)
2. [Arquitectura del sistema](#2-arquitectura-del-sistema)
3. [Modelo de datos](#3-modelo-de-datos)
4. [Especificación de la API](#4-especificación-de-la-api)
5. [Historias de usuario](#5-historias-de-usuario)
6. [Tickets de trabajo](#6-tickets-de-trabajo)
7. [Pull requests](#7-pull-requests)

---

## 0. Ficha del proyecto

### **0.1. Tu nombre completo:**
Carmen Margarita Carro Martinez Pinillo

### **0.2. Nombre del proyecto:**
GastOS

### **0.3. Descripción breve del proyecto:**
GastOS es una plataforma interna que sustituye la gestión manual de gastos (dietas, estancias, desplazamientos, combustible) basada en plantillas Excel, por un flujo digital con captura automatizada de tickets (OCR + LLM), revisión humana (HITL), evaluación de política de aprobación y conciliación, con trazabilidad completa e informes operativos y de dirección.

### **0.4. URL del proyecto:**
No disponible todavía. No existe un entorno desplegado (fuera del alcance de este proyecto de demostración); en su lugar se puede arrancar el stack completo en local siguiendo las instrucciones de la sección 1.4.

### 0.5. URL o archivo comprimido del repositorio
El repositorio es local (`GastOSApp/`, monorepo) y la entrega se materializa como commits en la rama `master`. No hay remoto configurado (`git remote`) ni URL pública; `backend/`, `frontend/` y el resto del proyecto están en la raíz del repositorio.

---

## 1. Descripción general del producto

### **1.1. Objetivo:**

La Empresa gestiona hoy el gasto interno (dietas, estancias, desplazamientos, combustible de flota) mediante plantillas Excel cumplimentadas manualmente por cada empleado. Esto genera falta de trazabilidad, riesgo de pérdida de justificantes, control solo reactivo y ninguna visibilidad agregada sin consolidar ficheros a mano.

GastOS automatiza la captura, clasificación, revisión y aprobación del gasto, y centraliza el archivo de justificantes con retención garantizada para auditoría, aportando valor a tres perfiles:

- **Empleados/Conductores:** capturan un gasto en segundos subiendo una foto del ticket, sin rellenar formularios largos.
- **Aprobadores (Administración) y Dirección:** disponen de un flujo de aprobación estructurado y trazable, en lugar de creación y revisión manual de Excel.
- **Administración/Contabilidad y Administrador de plataforma:** obtienen informes operativos y de dirección fiables, y un archivo centralizado con retención auditable.

### **1.2. Características y funcionalidades principales:**

| Funcionalidad | Estado en este proyecto | Referencia RFP |
|---|---|---|
| Módulo **Gastos generales** (captura, OCR+LLM, HITL, aprobación, conciliación) | **[MVP]** | 3.1, 3.2.2, 3.4 |
| Módulo **Combustible** (repostaje, km, tarjeta Cepsa/Visa) | **Implementado (fuera del MVP de 3 HUs)** | 3.1, 3.2.1 |
| Vía de entrada "sin ticket" (entrada manual + anotación textual obligatoria) | **[MVP]** (como parte de la revisión HITL) | 3.3 |
| Extracción automática: OCR (Tesseract) + clasificación LLM (Ollama) | **[MVP]** | 3.4, 5.2 |
| Revisión HITL con registro de cada edición | **[MVP]** | 3.4 |
| Motor de política de aprobación (umbral parametrizable, "vuelo" siempre) | **[MVP]** | 3.5 |
| Flujo de aprobación/rechazo con motivo | **[MVP]** | 3.4, 3.5 |
| Roles y permisos completos (5 roles) | **[DOCUMENTADO]** — MVP: Empleado, Aprobador y Admin (Admin resuelve y puede autoaprobar) | 3.9 |
| Períodos de liquidación configurables + detección "fuera de período" | **[DOCUMENTADO]** | 3.6.1 |
| Notificaciones (urgentes vs. ordinarias) | **[DOCUMENTADO]** | 3.6.2, 3.7 |
| Auditoría inmutable completa (todas las operaciones) | **[MVP parcial]** — trazas de las 3 historias implementadas; el resto de eventos quedan documentados | 3.8 |
| Informe de detalle operativo + exportación a Excel por plantilla | **Implementado (fuera del MVP de 3 HUs)** | 3.10.1 |
| Informes agregados (nivel umbral / desglose) | **[DOCUMENTADO]** | 3.10.2 |
| SSO con Microsoft Entra ID | **[DOCUMENTADO]** — MVP usa un mock de sesión, ver nota en 1.4 | 5.1 |
| Retención documental ≥5 años | **[DOCUMENTADO]** | 5.4 |
| Fase 2 completa (responsable de equipo, gráficos, comparativos, presupuestos, alertas, agrupación de registros) | **[DOCUMENTADO]** — fuera de alcance de este proyecto | Sección 4 |

> Esta tabla es la única fuente de verdad sobre qué se construye realmente. El resto de secciones (historias, tickets, API, modelo de datos) documentan el producto completo, pero desarrollan en detalle únicamente las 3 historias Must-Have marcadas como **[MVP]** en la sección 5.

### **1.3. Diseño y experiencia de usuario:**

El flujo de pantallas del MVP implementado es:

1. **Login** (sesión local en MVP; SSO Entra ID en versión completa). Tras un login correcto, Empleado aterriza en **Captura**; Aprobador y Admin aterrizan en **Bandeja**.
2. **Captura** (`/captura`) → **pantalla de lectura**: tres tabs accesibles (ARIA tablist) **Pendientes / Rechazados / Aprobados**, cada una con su conteo en la etiqueta y la tab activa sincronizada con la URL (`?tab=`, persiste tras recarga; valor inválido o ausente = Pendientes). Con el asistente cerrado la pantalla **no renderiza ningún formulario de alta**; las antiguas tarjetas de métricas se sustituyen por los conteos de las tabs.
3. **Captura de gasto (escritura bajo demanda)** → el botón "+ Capturar gasto" abre un **sheet** (100% del viewport en móvil, panel de scroll propio en desktop) con el asistente: elegir "Gastos generales", subir imagen/PDF o marcar "sin ticket" (anotación obligatoria). El sheet gestiona el foco como los demás diálogos (focus trap, Escape devuelve el foco al botón). El draft OCR/HITL se guarda en `sessionStorage`: al volver a `/captura` el sheet **se reabre automáticamente** con los datos restaurados sin volver a subir el fichero; cerrarlo sin confirmar conserva el draft.
4. **OCR + HITL en el sheet** → los campos extraídos (fecha, concepto, importe, forma de pago y área sugerida) se revisan en el mismo sheet. Acciones: **Confirmar** (PATCH + envío a política; **no aprueba**) y **Modificar** (edición, igual para Empleado, Aprobador y Admin, también sobre gasto ajeno en alcance). Admin autor ve además **Autoaprobar**. Tras el envío, un toast accesible (`role="status"`) informa del resultado y el sheet se cierra; estas acciones NO piden confirmación adicional porque el flujo ya incluye revisión HITL.
5. **Tabs de lectura de Captura** (vía `GET /expenses`, alcance por rol): Pendientes (`borrador`, `pendiente_aprobacion`), Rechazados (`rechazado`+`conciliado`) y Aprobados del mes en curso (filtro de fechas editable). Empleado solo ve los suyos; Aprobador ve autores Empleado y Aprobador (no Admin); Admin ve todos. En **Pendientes**, cada fila tiene **Modificar** (GET extraction → modal de Revisión HITL; Confirmar = PATCH; Cancelar cierra sin mutar). Aprobador y Admin ven además **Aprobar** y **Rechazar** en filas que el API permite resolver (no autoaprobación del Aprobador ni rechazo de registro propio); **Rechazar** abre un **diálogo de motivo obligatorio bajo demanda** (ya no hay input de motivo embebido en la fila). Cada fila ocupa una única línea de acciones compactas.
6. **Confirmación de política** → si no requiere aprobación humana, el registro queda conciliado; si sí, pasa a `pendiente_aprobacion`.
7. **Bandeja de aprobación** (solo roles Aprobador o Admin) → lista todos los gastos generales **no finales** (`borrador` y `pendiente_aprobacion`) con el **nombre del solicitante**, **Modificar** (mismo modal HITL) y, en cada fila resoluble, **Aprobar** y **Rechazar** en una única línea por fila. El rechazo exige motivo en el diálogo (Escape lo cancela devolviendo el foco a la fila sin llamar a la API); Aprobar mantiene su confirmación explícita antes de llamar a la API y ambos flujos muestran toast de éxito. Un `borrador` se rechaza con `POST /expenses/:id/reject`. El rol solo Empleado no ve Bandeja en la navegación y `/bandeja-aprobacion` lo redirige a Captura. Las pantallas muestran los estados con **etiquetas humanizadas** (Borrador, En revisión, Aprobado, Rechazado, Conciliado) desde un mapeo único (`frontend/src/lib/expense-status.ts`), cada página fija su `<title>` dinámico ("Captura de tickets · GastOS", …) y el header no expone la URL de la API.
8. **Combustible** (`/combustible`) → también pantalla de lectura: la tabla **"Mis repostajes"** (fecha, vehículo, km, importe, ticket adjunto, estado) se alimenta de `GET /fuel-expenses` con el alcance por rol (sin columna Solicitante si todo es propio); el botón "+ Nuevo repostaje" abre un **dialog centrado** con el alta (ticket opcional, sin OCR). El éxito cierra el dialog, avisa con toast y **refresca la tabla sin recargar**. Los repostajes no recorren el flujo de aprobación (permanecen en `borrador`), por lo que esta vista es una única tabla sin tabs de estado.

Este flujo está implementado y validado end-to-end mediante una suite E2E (Playwright) organizada por caso de prueba y escenario en `frontend/tests/e2e/<caso>/<escenario>.spec.ts` (9 casos, 33 specs, variantes success/error/cancel/access-denied, incluido el caso `read-write-flow` con tabs/`?tab=`, teclado del tablist, accesibilidad del sheet y lectura de combustible por rol), con grabación de vídeo revisable (pausas y scroll) en `frontend/test-results/`. La bandeja consume pendientes en cada pasada (aprobar/rechazar los retiran), por lo que `global-setup.ts` **siembra 3 gastos "En revisión" vía API** para que la suite sea determinista y repetible.

### **1.4. Instrucciones de instalación:**

**Stack:** Backend NestJS (Node/TypeScript) · Frontend React + TypeScript · PostgreSQL · AWS S3 (adjuntos) · Tesseract (OCR) · Ollama (LLM, infraestructura propia) · Microsoft Entra ID (SSO, documentado, mock en MVP).

```bash
# 1. Clonar el repositorio (monorepo)
git clone <url-del-repositorio>
cd GastOSApp

# 2. Backend
cd backend
npm install
cp .env.example .env        # completar variables, ver tabla abajo
npx prisma migrate dev      # o el ORM elegido (TypeORM/Prisma)
npm run start:dev

# 3. Frontend (en otra terminal)
cd frontend
npm install
cp .env.example .env
npm run dev

# 4. Servicios de IA en local
#    - Tesseract: apt-get install tesseract-ocr (o vía contenedor Docker)
#    - Ollama: instalar, ejecutar `ollama serve` y descargar el modelo elegido
```

**Variables de entorno principales** (sin valores reales — solo referencia, nunca se versionan secretos):

| Variable | Descripción |
|---|---|
| `DATABASE_URL` | Cadena de conexión a PostgreSQL |
| `AWS_S3_BUCKET` / `AWS_REGION` | Almacenamiento de adjuntos |
| `OLLAMA_BASE_URL` | URL del servicio Ollama local |
| `OCR_SERVICE_URL` | Endpoint del servicio Tesseract |
| `APPROVAL_THRESHOLD_DEFAULT` | Umbral de importe por defecto para requerir aprobación |
| `AZURE_AD_CLIENT_ID` / `AZURE_AD_TENANT_ID` | *(documentado, no usado en MVP)* — integración Entra ID futura |

> **[SENSIBLE]** Ningún dato real de tickets, empleados o tarjetas debe usarse en entornos de desarrollo o pruebas con servicios externos de IA. Para probar el pipeline OCR+LLM usar tickets sintéticos o anonimizados; cualquier uso de datos reales fuera de la infraestructura propia requiere aprobación previa de `ai.seguridad@fake.com`.

### Arranque local del scaffold

El monorepo ya incluye `backend/` (NestJS) y `frontend/` (React + Vite). En este recorte no hay migraciones de dominio ni `prisma migrate`: Prisma solo hace un ping a PostgreSQL.

```bash
# PostgreSQL (contenedor local; coincide con backend/.env.example)
docker compose up -d
# postgresql://postgres:postgres@localhost:5432/gastos

# Backend
cd backend
cp .env.example .env    # DATABASE_URL, JWT_SECRET y ADMIN_BOOTSTRAP_* obligatorios
npx prisma generate
npm run start:dev       # http://localhost:3000  — GET /health  POST /auth/login  GET /auth/me

# Frontend (otra terminal)
cd frontend
cp .env.example .env    # VITE_API_BASE_URL=http://localhost:3000
npm run dev             # http://localhost:5173
# Rutas: /login  /captura  /combustible  /bandeja-aprobacion (Aprobador|Admin)  /informe-detalle (Admin)
# /revision-hitl/:id redirige a /captura (HITL inline)
```

Sin `DATABASE_URL` el backend no arranca. Tesseract, Ollama y S3 no se usan en el scaffold.

---

## 2. Arquitectura del Sistema

### **2.1. Diagrama de arquitectura:**

```mermaid
graph TB
    subgraph Cliente
        FE["Frontend Web<br/>React + TypeScript"]
    end

    subgraph Identidad["[DOCUMENTADO] — mock en MVP"]
        ENTRA["Microsoft Entra ID<br/>SSO / OIDC"]
    end

    subgraph Backend["Backend — NestJS"]
        API["API REST<br/>Controllers"]
        AUTHM["Módulo Auth"]
        POLICY["Motor de Política<br/>de Aprobación · MVP"]
        AUDIT["Módulo de Auditoría · MVP parcial"]
        NOTIF["Módulo de Notificaciones<br/>[DOCUMENTADO]"]
        EXPORT["Módulo de Exportación · HU-07"]
    end

    subgraph IA["Capa de Extracción / Clasificación · MVP (desacoplable)"]
        OCR["Servicio OCR<br/>Tesseract"]
        LLM["Servicio LLM<br/>Ollama · infra propia"]
    end

    subgraph Datos
        DB[("PostgreSQL")]
        S3[("AWS S3<br/>Adjuntos")]
    end

    FE -- HTTPS/REST --> API
    FE -. Login OIDC .-> ENTRA
    ENTRA -. Token validado .-> AUTHM
    AUTHM --> API
    API --> POLICY
    API --> AUDIT
    API -. futuro .-> NOTIF
    API --> EXPORT
    API -- Imagen del ticket --> OCR
    OCR -- Texto en bruto + traza --> LLM
    LLM -- Datos estructurados + confianza --> API
    API --> DB
    API --> S3
    AUDIT --> DB
```

**Patrón elegido:** monolito modular (módulos NestJS con límites claros) con la capa de extracción (OCR/LLM) tratada como servicios desacoplados detrás de una interfaz interna, no como librerías embebidas. Esto cumple el requisito 5.2 del RFP ("la arquitectura debe permitir sustituir cualquier capa sin rediseño del conjunto").

- **Por qué este patrón:** con un presupuesto de ~30h y un equipo de una persona, una arquitectura de microservicios completa añadiría sobrecarga operativa (despliegue, red, observabilidad distribuida) sin beneficio real a esta escala. Un monolito modular da velocidad de desarrollo y mantiene los límites de dominio (auth, política de aprobación, auditoría, extracción) como módulos independientes, facilitando una futura extracción a servicios si el proyecto creciera.
- **Beneficios:** rapidez de implementación, despliegue único y simple, coherencia transaccional al escribir registro + traza de auditoría en la misma unidad de trabajo.
- **Déficits asumidos:** no hay escalado independiente por módulo (aceptable para una herramienta interna de un solo departamento); el acoplamiento al mismo proceso Node exige cuidado para no romper el aislamiento entre módulos.

### **2.2. Descripción de componentes principales:**

| Componente | Tecnología | Estado |
|---|---|---|
| Frontend | React + TypeScript | MVP |
| API Backend | NestJS (Node/TypeScript) | MVP |
| Motor de política de aprobación | Módulo NestJS (lógica de dominio) | MVP |
| Módulo de auditoría | Interceptors de NestJS + tabla de eventos | MVP parcial |
| Servicio OCR | Tesseract (proceso/contenedor separado) | MVP |
| Servicio LLM | Ollama, infraestructura propia | MVP |
| Base de datos | PostgreSQL | MVP |
| Almacenamiento de adjuntos | AWS S3 | MVP |
| Identidad | Microsoft Entra ID (SSO/OIDC) | Documentado (mock en MVP) |
| Notificaciones | Servicio de notificaciones (email / canal urgente diferenciado) | Documentado |
| Exportación de informes | Generación Excel (`exceljs`) por plantilla de módulo | Implementado (HU-07, detalle operativo) |

### **2.3. Descripción de alto nivel del proyecto y estructura de ficheros**

Estructura de **monorepo** con carpetas `backend/` y `frontend/` independientes en el mismo repositorio:

```
GastOSApp/                        # raíz del repositorio (monorepo)
├── backend/                      # API NestJS + Prisma + tests (Unit/integ./e2e)
│   ├── src/modules/
│   │   ├── auth/                 # Login email/contraseña, JWT, RBAC (Entra ID documentado, mock)
│   │   ├── expenses/             # Registro de gasto general (MVP) + auditoría (getAuditTrail)
│   │   ├── extraction/           # Orquestación OCR + LLM (Tesseract/Wrapper)
│   │   ├── approval-policy/      # Motor de política de aprobación (MVP)
│   │   ├── audit/                # Trazas de auditoría (MVP parcial)
│   │   ├── fuel/                 # Módulo Combustible (HU-04: POST alta + GET /fuel-expenses lectura con alcance por rol)
│   │   ├── reports/              # Informe detalle operativo + Excel (HU-07)
│   │   └── notifications/        # Documentado, no implementado
│   ├── prisma/                   # schema + seed (usuarios demo argon2)
│   └── test/                     # e2e-spec backend
├── frontend/                     # React + TypeScript (Vite) + tests + E2E Playwright
│   ├── src/pages/                # Captura (lectura con tabs + sheet), Bandeja, Combustible (lectura + dialog), Informe detalle
│   ├── src/components/           # UI (tabs ARIA, dialog/sheet), CaptureWizard, RejectReasonDialog, modal Revisión HITL, listas
│   ├── src/services/             # Cliente API
│   └── tests/e2e/                # Suite Playwright por caso/escenario (9 casos + _helpers, 33 specs)
├── openspec/                     # Especificaciones OpenSpec (specs canónicas + archive)
├── postman/                      # Colecciones/entorno de pruebas HTTP
├── scripts/                      # Utilidades de desarrollo
├── tickets_Test/                 # Ejemplares de tickets para pruebas de OCR
├── docker-compose.yml            # Orquestación postgres + backend + frontend (nginx)
├── readme.md
├── prompts.md
└── .github/workflows/ci.yml      # Pipeline: test (matrix) + build + e2e
```

---

## 3. Modelo de datos

Modelo de datos completo del dominio (documentado). Las entidades MVP de Gastos generales más las de HU-04 Combustible se usan en código; el resto se modela para coherencia del RFP.

```mermaid
erDiagram
    USUARIO ||--o{ REGISTRO_GASTO : crea
    USUARIO }o--|| DEPARTAMENTO : pertenece_a
    USUARIO ||--o{ APROBACION : resuelve
    USUARIO ||--o{ EVENTO_AUDITORIA : origina
    REGISTRO_GASTO ||--o| ADJUNTO : tiene
    REGISTRO_GASTO ||--o{ EVENTO_AUDITORIA : genera
    REGISTRO_GASTO |o--o| APROBACION : requiere
    REGISTRO_GASTO }o--|| PERIODO_LIQUIDACION : pertenece_a
    REGISTRO_GASTO }o--o| AREA_GASTO : clasificado_en
    REGISTRO_GASTO }o--o| VEHICULO : referencia
    REGISTRO_GASTO }o--o| TARJETA_PAGO : pagado_con

    USUARIO {
        uuid id PK
        string nombre
        string email
        enum rol "Empleado|Aprobador|Direccion|Administracion|AdminPlataforma"
        uuid departamento_id FK
    }
    DEPARTAMENTO {
        uuid id PK
        string nombre
    }
    REGISTRO_GASTO {
        uuid id PK
        enum tipo "combustible|general"
        date fecha
        decimal importe
        decimal kilometraje "solo combustible"
        string forma_pago
        string concepto
        string anotacion_textual
        enum estado "borrador|pendiente_aprobacion|aprobado|rechazado|conciliado|fuera_de_periodo"
        uuid autor_id FK
        uuid periodo_id FK
        uuid area_id FK
        uuid vehiculo_id FK
        uuid tarjeta_id FK
    }
    AREA_GASTO {
        uuid id PK
        string nombre "comida|estancia|vuelo|parking|otros"
        boolean requiere_aprobacion_siempre
    }
    VEHICULO {
        uuid id PK
        string matricula
    }
    TARJETA_PAGO {
        uuid id PK
        string tipo "Cepsa|Visa corporativa"
    }
    PERIODO_LIQUIDACION {
        uuid id PK
        date fecha_inicio
        date fecha_fin
    }
    ADJUNTO {
        uuid id PK
        uuid registro_id FK
        string url_s3
        string tipo_archivo
        timestamp fecha_subida
    }
    APROBACION {
        uuid id PK
        uuid registro_id FK
        uuid aprobador_id FK
        enum resultado "pendiente|aprobado|rechazado"
        string motivo_rechazo
        decimal umbral_vigente
        timestamp fecha_evaluacion
    }
    EVENTO_AUDITORIA {
        uuid id PK
        uuid registro_id FK
        uuid usuario_id FK
        string tipo_evento "ingesta|ocr|clasificacion_llm|edicion|aprobacion|rechazo|conciliacion"
        json valor_anterior
        json valor_nuevo
        timestamp timestamp
    }
```

**Uso en el MVP:** `USUARIO` (roles Empleado, Aprobador y Admin), `REGISTRO_GASTO` (solo `tipo = general`), `AREA_GASTO`, `ADJUNTO`, `APROBACION`, `EVENTO_AUDITORIA`.
**Usadas también por HU-04 (Combustible, fuera del MVP de 3 HUs):** `VEHICULO`, `TARJETA_PAGO`, `REGISTRO_GASTO` (`tipo = combustible`, campo `kilometraje`).
**Solo documentadas:** `DEPARTAMENTO`, `PERIODO_LIQUIDACION` (informes y notificaciones — fuera de alcance de esta entrega).

### Ciclo de vida del registro (Gastos generales) **[MVP — especificado]**

```mermaid
stateDiagram-v2
    [*] --> borrador: POST /expenses
    borrador --> borrador: HITL PATCH / attachment
    borrador --> pendiente_aprobacion: submit (requiere humano)
    borrador --> conciliado: submit (umbral)
    pendiente_aprobacion --> pendiente_aprobacion: PATCH autor
    pendiente_aprobacion --> aprobado: approve
    pendiente_aprobacion --> rechazado: reject
    pendiente_aprobacion --> conciliado: re-submit umbral
    rechazado --> borrador: PATCH correccion
    rechazado --> pendiente_aprobacion: submit
    rechazado --> conciliado: submit umbral
    aprobado --> [*]
    conciliado --> [*]
```

**Matriz de mutabilidad (autor del registro):**

| Estado | PATCH / attachment* | submit | DELETE | Notas |
|---|---|---|---|---|
| `borrador` | sí | sí | no | Fase HITL previa a política |
| `pendiente_aprobacion` | sí | sí (reevalúa) | no | Editable hasta resolución |
| `rechazado` | sí | sí | no | Vuelta al autor |
| `aprobado` | **409** | **409** | no | Inmutable |
| `conciliado` | **409** | **409** | no | Inmutable (mismo bloqueo) |

\*attachment solo si el registro admite ticket.

**Visibilidad:** el autor y un Aprobador/Admin en alcance de listado pueden consultar el `estado` (p. ej. vía `GET /expenses/:id/extraction`). `GET /expenses` lista gastos generales según el rol: Empleado solo propios; Aprobador, los de autores Empleado|Aprobador (no Admin); Admin, todos. Cada fila puede incluir `approvalId`. Captura muestra pendientes, rechazados (`rechazado`+`conciliado`) y aprobados. Bandeja lista no-finales (`borrador`, `pendiente_aprobacion`). El historial de transiciones se retiene en `EVENTO_AUDITORIA`; el contrato HTTP `GET /expenses/:id/audit` implementa la consulta de la trazabilidad (eventos ordenados por timestamp DESC). El combustible **no recorre este ciclo de aprobación** (permanece en `borrador`) y **no se mezcla** en `GET /expenses`: se consulta con su propio endpoint `GET /fuel-expenses`, con la misma jerarquía de visibilidad por rol y un booleano `ticketAdjunto` por fila (capability `fuel-expense-read`).

**Resolución:** roles `Aprobador` o `Admin`. Autoaprobación: bloqueada para `Aprobador` autor; **permitida** para `Admin` autor. Tras `aprobado` o `conciliado` la UI MUST mostrar solo lectura (sin edición ni borrado).

**Fuente de detalle:** OpenSpec change `expense-lifecycle-immutability`.

 ---

## 4. Especificación de la API

| Método | Endpoint | Descripción | Estado |
|---|---|---|---|
| `POST` | `/expenses` | Crea un registro de gasto general (con o sin ticket) | **MVP** (HU-01) |
| `GET` | `/expenses` | Lista gastos generales según rol (Empleado: propios; Aprobador: autores Empleado/Aprobador, no Admin; Admin: todos); query `estado`, `fechaDesde`, `fechaHasta`; incluye `autorNombre` y `approvalId` si hay pendiente | **MVP** (listas de Captura y Bandeja) |
| `POST` | `/expenses/:id/attachment` | Sube la imagen del ticket, dispara OCR + LLM | **MVP** (HU-01) |
| `GET` | `/expenses/:id/extraction` | Consulta campos propuestos/actuales, `estado` y `areaId` persistido; autor o Aprobador/Admin en alcance de listado | **MVP** (HU-02; `estado` alineado en lifecycle) |
| `PATCH` | `/expenses/:id` | Edita campos HITL y puede persistir `areaId` si estado mutable (`borrador` \| `pendiente_aprobacion` \| `rechazado`); mismo alcance que extracción; 409 si `aprobado` \| `conciliado` | **MVP** (HU-02 + lifecycle) |
| `POST` | `/expenses/:id/submit` | Envía a evaluación de política; mismo alcance que extracción; 409 si `aprobado` \| `conciliado` | **MVP** (HU-03) |
| `POST` | `/expenses/:id/reject` | Rechaza un `borrador` (Aprobador/Admin, motivo obligatorio; 403 propio o Empleado) | **MVP** (bandeja no-final) |
| `GET` | `/approvals` | Lista pendientes (roles Aprobador o Admin); incluye `autorNombre` del solicitante | **MVP** (HU-03) |
| `POST` | `/approvals/:id/approve` | Aprueba; Aprobador no autoaprueba; Admin sí puede autoaprobar; registro queda inmutable | **MVP** (HU-03 + lifecycle) |
| `POST` | `/approvals/:id/reject` | Rechaza con motivo obligatorio | **MVP** (HU-03) |
| `DELETE` | `/expenses/:id` | **No expuesto** — prohibido en todos los estados | **Documentado (prohibición)** |
| `GET` | `/expenses/:id/audit` | Consulta la traza de auditoría de un registro (eventos ordenados por timestamp DESC) | **MVP** |
| `POST` | `/fuel-expenses` | Crea un registro de combustible | **Implementado (HU-04)** |
| `GET` | `/fuel-expenses` | Lista repostajes (`tipo=combustible`) con la misma jerarquía por rol que `GET /expenses`; query `estado` (CSV), `fechaDesde`, `fechaHasta`; filas con matrícula, tarjeta, `ticketAdjunto` (booleano, sin exponer adjuntos); orden descendente por fecha | **Implementado (HU-04, lectura)** |
| `GET` | `/settlement-periods` | Consulta/gestiona períodos de liquidación | Documentado |
| `GET` | `/reports/detail` | Informe de detalle operativo con filtros y exportación Excel (`format=json|xlsx`) | **Implementado (HU-07)** |
| `GET` | `/reports/aggregate` | Informes agregados (nivel umbral / desglose) | Documentado |
| `GET` | `/auth/login` | Redirección SSO Entra ID | Documentado (mock en MVP) |

---

## 5. Historias de usuario

> Backlog completo del dominio, documentado por trazabilidad. Se implementan **HU-01, HU-02, HU-03** (MVP), **HU-04** (Combustible) y **HU-07** (informe de detalle operativo), fuera del recorte Must-Have original cuando aplica.

### HU-01 — Captura y extracción automática de gasto general **[MVP]**
*Ref. RFP: 3.2.2, 3.3, 3.4, 5.2*

**Como** Empleado, **quiero** subir la imagen de un ticket de gasto general (o indicar que no tengo ticket), **para** que el sistema proponga los datos estructurados automáticamente y no tener que rellenarlos todos a mano.

**Criterios de aceptación (BDD):**
```gherkin
Dado que el usuario ha seleccionado el módulo "Gastos generales"
Cuando sube una imagen de ticket (foto o PDF)
Entonces el sistema aplica OCR y clasificación LLM
Y propone fecha, concepto, importe y forma de pago para revisión
Y genera una traza de auditoría independiente para la ingesta, el OCR y la clasificación LLM

Dado que el usuario no dispone de ticket
Cuando marca la opción "sin ticket"
Entonces el sistema exige una anotación textual obligatoria antes de continuar
```

### HU-02 — Revisión HITL y confirmación del registro **[MVP]**
*Ref. RFP: 3.4*

**Como** Empleado, **quiero** revisar y corregir los datos propuestos antes de confirmarlos, **para** asegurarme de que el registro final es correcto.

**Criterios de aceptación (BDD):**
```gherkin
Dado que existen campos propuestos por OCR/LLM para un registro
Cuando el usuario edita uno o varios campos
Entonces cada edición queda registrada (campo, valor anterior, valor nuevo, usuario, timestamp)

Dado que el usuario ha revisado y confirmado el registro
Cuando pulsa "Confirmar"
Entonces el registro pasa a evaluación de política de aprobación
```

### HU-03 — Evaluación de política y aprobación/rechazo **[MVP]**
*Ref. RFP: 3.4, 3.5*

**Como** Aprobador (Administración), **quiero** que el sistema me muestre solo los registros que realmente requieren mi aprobación y poder aprobarlos o rechazarlos con motivo, **para** mantener el control sin revisar manualmente todo el gasto.

**Criterios de aceptación (BDD):**
```gherkin
Dado un registro de área "vuelo"
Cuando se evalúa la política de aprobación
Entonces requiere aprobación siempre, independientemente del importe

Dado un registro que no es "vuelo" y su importe es menor que el umbral configurado
Cuando se evalúa la política de aprobación
Entonces se concilia directamente sin aprobación

Dado un registro pendiente de aprobación
Cuando el Aprobador lo rechaza
Entonces debe indicar un motivo obligatorio
Y el registro vuelve al autor para corrección

Dado un registro creado por el propio Aprobador
Cuando requiere aprobación
Entonces el sistema impide que el mismo Aprobador lo apruebe
```

---

### HU-04 — Registro de repostaje de combustible **[Implementado — fuera del MVP de 3 HUs]**
*Ref. RFP: 3.2.1*

**Como** Empleado, **quiero** registrar un repostaje de combustible (vehículo, km, tarjeta, ticket opcional), **para** dejar constancia digital del gasto de flota sin Excel.

**Criterios de aceptación (BDD):**
```gherkin
Dado que el Empleado elige el módulo Combustible
Cuando indica vehículo, kilometraje, tarjeta (Cepsa o Visa corporativa), fecha e importe
Entonces el sistema crea un registro tipo combustible en borrador
Y permite omitir el ticket sin exigir anotación de Gastos generales

Dado un registro de combustible creado
Cuando el Empleado adjunta opcionalmente una foto o PDF de ticket
Entonces el sistema almacena el justificante sin exigir OCR ni LLM
```

### HU-07 — Informe de detalle operativo y exportación Excel **[Implementado — fuera del MVP de 3 HUs]**
*Ref. RFP: 3.10.1*

**Como** Administración (rol `Admin`), **quiero** generar el informe de detalle operativo con filtros y exportarlo a Excel, **para** consolidar el gasto sin plantillas manuales.

**Criterios de aceptación (BDD):**
```gherkin
Dado un Admin autenticado
Cuando consulta GET /reports/detail con filtros de fechas, tipo y/o estado
Entonces el sistema devuelve el detalle operativo en JSON
Y Empleado o Aprobador reciben 403

Dado el mismo conjunto de filtros
Cuando solicita format=xlsx
Entonces el sistema descarga un Excel con hojas "Gastos generales" y "Combustible"
Y las columnas respetan la plantilla canónica por módulo
```

### Historias documentadas (fuera del alcance de implementación de este proyecto)

| ID | Historia | Ref. RFP |
|---|---|---|
| HU-05 | Como Administrador, configuro períodos de liquidación y el sistema detecta registros fuera de período | 3.6.1 |
| HU-06 | Como usuario, recibo notificaciones urgentes (distinguibles por canal) frente a ordinarias | 3.6.2, 3.7 |
| HU-08 | Como Dirección, consulto informes agregados en dos niveles (umbral → desglose) según mi rol | 3.10.2 |
| HU-09 | Como Administrador de plataforma, gestiono el catálogo de áreas de gasto y el umbral de aprobación | 3.9 |
| HU-10 | Como usuario, inicio sesión mediante SSO corporativo (Microsoft Entra ID) | 5.1 |

---

## 6. Tickets de trabajo

> Desglose técnico únicamente de las 3 historias MVP.

| Ticket | Historia | Tipo | Descripción |
|---|---|---|---|
| TCK-01 | HU-01 | Backend | Endpoint de ingesta: recibe imagen, la almacena en S3 y crea el registro en estado inicial |
| TCK-02 | HU-01 | Backend | Orquestación del servicio OCR (Tesseract) + traza de auditoría de la extracción en bruto |
| TCK-03 | HU-01 | Backend | Integración con servicio LLM (Ollama) para clasificar/estructurar los datos, con confianza por campo |
| TCK-04 | HU-01 | Frontend | Pantalla de captura: selección de módulo, subida de imagen o entrada manual con anotación obligatoria |
| TCK-05 | HU-02 | Frontend | Pantalla de revisión HITL: campos editables, resaltando lo propuesto por IA vs. lo corregido |
| TCK-06 | HU-02 | Backend | Endpoint de confirmación: persiste el registro final y una entrada de auditoría por cada campo editado |
| TCK-07 | HU-03 | Backend | Motor de evaluación de política: regla "vuelo siempre", umbral parametrizable, bloqueo de autoaprobación |
| TCK-08 | HU-03 | Backend | Endpoints de aprobación/rechazo (motivo obligatorio en rechazo) y transición a estado conciliado |
| TCK-09 | HU-03 | Frontend | Bandeja de aprobación: lista de pendientes, acciones de aprobar/rechazar |

### Matriz de trazabilidad

| Épica | Historia | Tickets | PR |
|---|---|---|---|
| Captura y extracción | HU-01 | TCK-01, TCK-02, TCK-03, TCK-04 | *(pendiente, ver sección 7)* |
| Revisión humana | HU-02 | TCK-05, TCK-06 | *(pendiente)* |
| Aprobación | HU-03 | TCK-07, TCK-08, TCK-09 | *(pendiente)* |

---

## 7. Pull Requests

Cambios entregados por el coding agent durante la última iteración (Entrega 2), documentados como unidades de integración. El repositorio no tiene remoto configurado, por lo que la integración se materializó como commits en `master` y queda listo para abrir los PR frente al repositorio real / ramas feature cuando se configure un remoto.

| # | Commit (hash corto) | Área | Descripción del cambio | Cobertura/verificación |
|---|---|---|---|---|
| PR-1 | `873f4e8` | Proyecto (MVP core) | Base del monorepo: backend NestJS + frontend React/Vite + Docker Compose all-in-one + specs OpenSpec | `npm test` backend/frontend |
| PR-2 | `0f098fa` | CI/CD | Pipeline GitHub Actions (`test` matrix, `build` imágenes Docker, `e2e` con Postgres service) | Workflow YAML / `oxlint` |
| PR-3 | `73ee43d` | E2E | Suite Playwright segmentada por escenario y variante con ritmo humano (capture, approval, fuel/report/audit, role-guards) | `npx playwright test e2e/` → 14 tests OK |
| PR-4 | `e41cb48` | Backend (auditoría/TDD) | `GET /expenses/:id/audit` devuelve 404 para combustible; fixtures de seed; eliminado código muerto `requireOwnRegistro` | 141 tests backend + 51 frontend OK |
| PR-5 | `c042bc3`+`27169a7` | UX/a11y + E2E | Cambio OpenSpec `improve-ui-feedback-a11y`: toasts de éxito (`FlashToast` con `role="status"`), confirmación antes de aprobar/rechazar en Bandeja (Captura exenta, ya hay HITL), títulos `<title>` dinámicos por ruta, header sin URL de API, estados humanizados centralizados en `statusLabel()`, radios con label accesible, Login con `<h1>`/`<main>`, focus trap + restauración de foco en el modal HITL. Suite E2E reorganizada a `frontend/tests/e2e/<caso>/<escenario>.spec.ts` con 13 escenarios nuevos según `e2e-test-plan.md` | 29 E2E OK (reporter JSON: `expected: 29, skipped: 0`) + 63 unit frontend; `tsc -b` y `oxlint` sin avisos nuevos |
| PR-6 | `7531ddc..ebb173d` (merge `00fa506`) | UX lectura/escritura + Backend (HU-04) | Cambio OpenSpec `improve-ui-read-write-flow`: separación lectura/escritura de la UI. `/captura` pasa a pantalla de lectura con tabs ARIA (Pendientes/Rechazados/Aprobados con conteo, `?tab=` persistido en URL, sin tarjetas de métricas) y escritura en sheet full-screen (`CaptureWizard` con draft auto-reabrible); `/combustible` lectura "Mis repostajes" + dialog "Nuevo repostaje" con refresco sin recargar; rechazo con diálogo de motivo obligatorio bajo demanda (sustituye al input inline y al `window.confirm` del rechazo); nuevo `GET /fuel-expenses` con alcance por rol, filtros y `ticketAdjunto`; E2E: 4 specs nuevos `read-write-flow` + adaptación de captura/aprobación/combustible + siembra determinista de pendientes en `global-setup` | 33 E2E OK (reporter JSON: `expected: 33, skipped: 0`) + 90 unit frontend + 148 backend; `tsc` y lint verdes; recorrido Playwright desktop/móvil × 3 roles |

> Nota metodológica: se siguió TDD — los tests que fallan fieles a la especificación se escribieron primero (ej. `combustible → 404`), y la implementación se ajustó después para cumplir (`expenses.service.ts`). La suite E2E se dividió en 4 archivos por escenario con variantes success/error/access-denied y se se generó vídeo con pausas y scroll para su revisión humana. En la iteración final (`improve-ui-feedback-a11y`) la suite se rediseñó desde la especificación (`e2e-test-plan.md`: matriz escenario→test) y se segmentó a `frontend/tests/e2e/<caso>/<escenario>.spec.ts`; un E2E de focus trap destapó además un defecto real de producción (el foco no volvía al disparador del modal HITL porque la fila se deshabilitaba durante la carga), corregido con el propio test como verificación. En la iteración `improve-ui-read-write-flow` la suite volvió a actuar como red de seguridad del refactor: detectó que la extracción del wizard a `capture-wizard.tsx` había roto el `<form>` único que envolvió siempre la revisión HITL en la página original (los selectores E2E con alcance `form.` dejaron de resolver), y forzó a restaurar la estructura original; también reveló que la bandeja es data-consumidora (cada pasada aprueba/rechaza pendientes reales), resuelto con la siembra determinista de `global-setup`. Sin remoto configurado no procede abrir Pull Requests reales; queda listo para hacerlo al añadir un `origin`.

