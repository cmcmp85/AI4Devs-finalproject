# Registro de Prompts y Uso de IA

> Detalla en esta sección los prompts principales utilizados durante la creación del proyecto, que justifiquen el uso de asistentes de código en todas las fases del ciclo de vida del desarrollo. 

## Índice
1. [Descripción general del producto](#1-descripción-general-del-producto)
2. [Arquitectura del sistema](#2-arquitectura-del-sistema)
3. [Modelo de datos](#3-modelo-de-datos)
4. [Especificación de la API](#4-especificación-de-la-api)
5. [Historias de usuario](#5-historias-de-usuario)
6. [Tickets de trabajo](#6-tickets-de-trabajo)
7. [Pull requests](#7-pull-requests)

---

## 1. Descripción general del producto

**Prompt 1 (Brainstorming Inicial):**
> "Actúa como PMO y Arquitecto de Soluciones de IA experto en FinTech y SaaS. Quiero hacer un brainstorming para un nuevo proyecto de \"Ticketing/Facturación Inteligente\". La idea base del flujo es: 1. Sube foto. 2. Extrae datos (OCR/LLM). 3. Clasifica y activa acciones. Propónme ideas divididas en: Casos de Uso, Arquitectura, Desafíos Técnicos y Monetización (3 a 4 ideas por categoría)."

**Prompt 2 (Delimitación del Alcance):**
> "Ayúdame a redactar el Alcance del Proyecto de manera clara y profesional. Necesito documentar todo el sistema a nivel general, pero dejando perfectamente estipulado que la implementación de esta fase se limitará únicamente al módulo de 'Gastos Generales' y a 3 Historias de Usuario. Asegúrate de marcar bien la frontera entre lo que se documenta y lo que se construye."

**Prompt 3 (Plan de Ejecución):**
> "Vamos a crear un paso a paso para implementar el proyecto cuyo contexto se encuentra en el RFP [...]. NO GENERES DOCUMENTACION, solo quiero el paso a paso para implementarlo siguiendo los requerimientos."

**💡 Nota explicativa (Criterio Humano):** 
Guié al LLM para no quedarse en una lluvia de ideas teórica, forzando una delimitación estricta entre el diseño teórico completo del RFP y el MVP ejecutable. La exigencia de separar visualmente `[MVP]` de `[DOCUMENTADO]` garantizó que las posteriores generaciones de código no alucinaran funcionalidades fuera de alcance.

---

## 2. Arquitectura del Sistema

**Prompt 1 (Selección de Stack):**
> "Vamos a centrarnos en la fase 1, pero antes elijamos el stack tecnológico [...]. En el backend si podemos utilizar nodeJs incluso NestJs, ya seria un stack conocido. Planteame opciones que cubran las especificaciones del proyecto y sus PROS y Contras."

**Prompt 2 (Especificación Técnica Core):**
> "Actúa como Arquitecto de Software senior con experiencia en AWS y sistemas de procesamiento de documentos con OCR y LLM. A partir del siguiente PRD, genera la especificación técnica del MVP con: Arquitectura general, Diseño OCR, Diseño LLM, Flujo HITL, Modelo de datos, Auditoría, S3, Entra ID y Entornos."

**Prompt 3 (Seguridad y Privacidad PII):**
> "Actúa como un desarrollador senior experto en ciberseguridad y NestJS. Acabo de implementar un servicio OCR que extrae texto bruto. Antes de enviar este texto a un LLM, necesito anonimizar PII por el RGPD. Crea un `PiiSanitizerService` que utilice Regex para reemplazar con '[REDACTED]': DNI/NIE/CIF, emails, teléfonos (formato España) e IBAN."

**💡 Nota explicativa (Criterio Humano):**
No dejé la elección de la arquitectura al azar. Validé que el stack propuesto incluyera NestJS (aprovechando experiencia previa) y garanticé la privacidad de datos "by design" obligando al agente a crear una capa de sanitización (Regex) antes de exponer cualquier string al servicio de LLM multimodal.

---

## 3. Modelo de Datos

**Prompt 1 (Corrección de Relaciones - Many-to-Many):**
> "El modelo actual trata el 'Rol' como un campo simple, pero los roles deberían ser una entidad propia en la base de datos, y un usuario debe poder tener MÚLTIPLES roles asignados al mismo tiempo (relación Many-to-Many). Realiza la refactorización en el modelo (Entidades), Lógica de Negocio, Middlewares y Migraciones."

**Prompt 2 (Auditoría e Inmutabilidad):**
> "Revisa todos los documentos de especificación existentes en el workspace. Verifica si el ciclo de vida de estados está modelado de forma estricta. Reglas: 5. Bloqueo Definitivo (Inmutabilidad): En el instante en que el ticket pasa a estado APPROVED, la entidad queda bloqueada. Las especificaciones deben documentar la prohibición de cualquier edición o eliminación posterior."

**💡 Nota explicativa (Criterio Humano):**
El agente propuso inicialmente un diseño de base de datos ingenuo (roles como strings y entidades mutables). Intervine directamente para refactorizar la base de datos hacia una relación transaccional madura (N:M en PostgreSQL) e impuse reglas de inmutabilidad financiera en los estados, asegurando que el diseño de BD reflejara reglas de negocio reales.

---

## 4. Especificación de la API

**Prompt 1 (Diseño de Endpoints y Roles):**
> "Lee detenidamente las especificaciones. Basándote en el stack, implementa la gestión de usuarios básica. Incluye endpoints CRUD, registro/login con JWT y establece la regla de negocio de que el registro por Entra ID asigne por defecto el rol de 'Empleado'. Escribe código modular."

**Prompt 2 (Depuración del Flujo API - OCR):**
> "Tengo un bug en el flujo de subida: cuando subo un PDF, el sistema devuelve un concepto estático. Haz un rastreo (trace) del flujo: 1. Envío desde el Frontend (FormData). 2. Controlador (Multer). 3. Servicio OCR/LLM. 4. Integración. Identifica en qué línea se está rompiendo y devuelve el dato real."

**Prompt 3 (Refactor del Flujo HITL - Modales):**
> "Al hacer clic en 'Modificar', el sistema debe realizar una petición GET para obtener los datos más recientes. Una vez obtenidos, abre un Modal. El botón Confirmar ejecuta un UPDATE (PATCH) hacia `/opsx-update`. Si es exitoso, cierra el popup y refresca la lista."

**💡 Nota explicativa (Criterio Humano):**
En lugar de aceptar código de API a ciegas, utilicé el agente como una herramienta de rastreo y depuración profunda. Definí estrictamente el contrato HTTP de los endpoints (GET para hidratar modales, PATCH para mutaciones) y obligué al asistente a revisar la traza completa de la petición cuando se detectaron respuestas estáticas hardcodeadas.

---

## 5. Historias de Usuario

**Prompt 1 (Generación de HUs core):**
> "Crea la propuesta de especificación para la historia HU-01, tal como está definida en la sección 5 de @readme.md. Genera la propuesta para HU-02 y HU-03."

**Prompt 2 (Ajuste manual de visibilidad por rol):**
> "He revisado el comportamiento y necesito hacer aclaraciones sobre visibilidad. Filtro por Rol: Empleado solo ve sus propios gastos. Aprobador / Admin: Visibilidad total de todos los usuarios. Aplica esto a Gastos Aprobados, Rechazados y Pendientes."

**Prompt 3 (Inclusión de alcance extendido):**
> "Quita el DOCUMENTADA, vamos a implementarlo aunque no esté en el MVP. Crea la propuesta de especificación para la historia HU-07 (Informes), tal como está definida en el readme."

**💡 Nota explicativa (Criterio Humano):**
El asistente tendía a mezclar permisos y no aislar la información. Apliqué correcciones manuales para garantizar que las historias de usuario reflejaran un control de acceso basado en roles (RBAC) estricto. Además, amplié el MVP original integrando flujos adicionales (Combustible e Informes) gestionando los deltas a través de un flujo estructurado con OpenSpec en el IDE.

---

## 6. Tickets de Trabajo

**Prompt 1 (Ticket: Extracción JSON Estricta):**
> "Eres un sistema de extracción de datos de documentos financieros. Analiza la imagen adjunta y extrae los campos en formato JSON estricto. Si no está presente, devuelve null. Indica el nivel de confianza (alto/medio/bajo). Devuelve únicamente el JSON, sin bloques de código markdown."

**Prompt 2 (Ticket: Reglas de Validación de Negocio):**
> "Valida el siguiente JSON extraído. Aplica las siguientes reglas y devuelve un JSON: 1. base_imponible + iva_importe = importe_total (±0.02). 2. Fecha no futura ni >5 años. 3. Importe positivo. 4. Si forma_pago es null, pendiente revisión humana. Devuelve: valido (boolean), errores, advertencias, campos_pendientes."

**Prompt 3 (Ticket: Lógica de Duplicados):**
> "Compara el siguiente ticket con el historial del usuario. Determina duplicado basado en: Mismo importe (±0.01), misma fecha, mismo NIF proveedor. Devuelve JSON con: es_duplicado, confianza, registro_coincidente_id, motivo."

**💡 Nota explicativa (Criterio Humano):**
Para los tickets relacionados con el motor de IA, no pedí implementaciones genéricas. Especifiqué la entrada y salida exacta (contratos JSON estrictos sin markdown) y definí las tolerancias matemáticas (±0.02€) para evitar que el LLM rompiera la integración del backend con alucinaciones de formato.

---

## 7. Pull Requests

**Prompt 1 (Diseño de la Suite E2E):**
> "Analiza la especificación actual y diseña los casos de prueba E2E necesarios para validar los cambios. Los tests deberán implementarse con playwright y organizarse bajo tests/e2e/<test-case>/<scenario>."

**Prompt 2 (Control de Calidad de Tests):**
> "He revisado el reporte y todos los tests e2e salen como 'skipped', un test 'skipped' NO es válido. Proporciona la evidencia de que la ejecución es completa."

**Prompt 3 (UI UX Refactor - Separación Lectura/Escritura):**
> "Explora el rediseño de la UI de lectura/escritura: /captura y /combustible como pantallas de lectura (tabs con conteo, estado persistido en URL) y la escritura bajo demanda (sheet full-screen). Genera la propuesta del cambio improve-ui-read-write-flow con los deltas de spec."

**💡 Nota explicativa (Criterio Humano):**
Actué como un "gatekeeper" de calidad. Cuando el agente generó tests en Playwright que se saltaban escenarios (skipped), rechacé el PR virtual exigiendo pruebas e2e ejecutables en verde reales. Además, el rediseño de UX/UI se guio paso a paso utilizando un plan de iteraciones, asegurando el ritmo humano en las validaciones antes de dar por cerrado el refactor.
