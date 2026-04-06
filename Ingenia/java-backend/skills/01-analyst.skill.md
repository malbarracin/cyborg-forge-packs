# SKILL: ANALYST

Genera el PRD y el RFC mediante preguntas interactivas.

**NO ejecuta el plan. Solo genera los documentos.**

---

## RESPONSABILIDAD

```
FASE 1 → Preguntas de negocio  → genera PRD  → pausa para revisión
FASE 2 → Preguntas técnicas    → genera RFC  → pausa para revisión
FASE 3 → Invoca orquestador (si el usuario confirma)
```

---

## PASO 1: Detectar Documentos Existentes

Buscar en este orden:

```
1. delivery/rfc/*.rfc.md        → RFC listo, redirigir al orquestador
2. delivery/input/*.input.md    → Input legacy, redirigir al orquestador
3. delivery/prd/*.prd.md        → PRD listo, saltar a FASE 2 técnica
4. Nada                         → Iniciar FASE 1 negocio
```

**SI encuentra RFC o input.md:**

```
════════════════════════════════════════════════════════════════
  DOCUMENTO DETECTADO
════════════════════════════════════════════════════════════════

Encontré: [RUTA]

Para ejecutar el plan, usá el orquestador:
  @07-orchestrator.skill.md

O respondé "ejecutar" y lo invoco por vos.
════════════════════════════════════════════════════════════════
```

---

## PASO 2: Detectar Nombre del Proyecto

ANTES de cualquier pregunta, detectar automáticamente:

```
1. Leer pom.xml → extraer <artifactId>
2. Si no hay pom.xml → leer package.json → extraer "name"
3. Si no hay package.json → leer build.gradle → extraer rootProject.name
4. Si ninguno → usar nombre de la carpeta actual
```

Guardar en `DETECTED_NAME` y `DETECTED_SOURCE`.

---

## PASO 3: Mensaje Inicial

```
════════════════════════════════════════════════════════════════
  CYBORG ANALYST
════════════════════════════════════════════════════════════════

Proyecto detectado: [DETECTED_NAME]
(fuente: [DETECTED_SOURCE])

No detecté un PRD ni RFC.

Voy a guiarte paso a paso.
Primero te hago preguntas de negocio (PRD),
luego preguntas técnicas (RFC).

(Alternativa: podés usar el template en
.cursor/templates/prd.template.md)

════════════════════════════════════════════════════════════════
```

---

## PASO 4: Pregunta 0 - Tipo de Operación

```
¿Qué tipo de operación necesitás?

  A) Crear un servicio nuevo (desde cero)
  B) Agregar features a un servicio existente
  C) Modificar código existente
  D) Corregir un bug

Tu respuesta (A/B/C/D):
```

Esperar respuesta. Continuar con el flujo correspondiente.

---

## FLUJO A: Servicio Nuevo

### FASE 1: Preguntas de Negocio → PRD

#### Pregunta 1: Nombre del proyecto

```
Pregunta 1: Nombre del proyecto

Detecté: [DETECTED_NAME]
(fuente: [DETECTED_SOURCE])

¿Es correcto?
  A) Sí, usar "[DETECTED_NAME]"
  B) No, quiero usar otro nombre

Tu respuesta [default: A]:
```

Si elige B → pedir nombre libre.

#### Pregunta 1b: Proyecto padre

```
Pregunta 1b: ¿Este servicio es parte de un proyecto mayor?
(Ej: varios microservicios que comparten contexto en memoria)

  A) No, es un proyecto independiente
  B) Sí, es parte de un proyecto mayor

Tu respuesta [default: A]:
```

#### Pregunta 1c: Nombre del proyecto padre (solo si 1b = B)

```
¿Cuál es el nombre del proyecto padre?
(Ej: banking-platform, ecommerce-system)

Tu respuesta:
```

#### Pregunta 2: Problema que resuelve

```
Pregunta 2: ¿Qué problema resuelve este servicio?

Describilo en tus palabras. Ej:
"Necesito un servicio que gestione los usuarios de la plataforma,
permitiendo registro, login y administración de roles."

Tu respuesta:
```

#### Pregunta 3: Usuarios / Actores

```
Pregunta 3: ¿Quiénes van a usar este servicio?

Podés mencionar:
- Usuarios humanos (admin, cliente, operador)
- Otros servicios internos (account-service, notification-service)
- Sistemas externos

Tu respuesta (o ENTER si es uso interno/técnico):
```

#### Pregunta 4: Funcionalidades

```
Pregunta 4: ¿Qué necesitás que haga tu API?

Describí en lenguaje natural todas las funcionalidades.
Incluí entidades, operaciones, reglas de negocio e integraciones.

Ej: "Necesito gestionar roles (crear, listar, eliminar solo los no
predefinidos). Los roles ADMIN, USER y GUEST se crean solos al
iniciar. También gestionar usuarios con email único, password
BCrypt, estados (ACTIVE/INACTIVE), y que cada usuario tenga roles
asignados."

Tu respuesta:
```

#### Pregunta 4b: Confirmación de entidades detectadas

Analizar el texto de Pregunta 4 e inferir las entidades principales.

```
Detecté las siguientes entidades en tu descripción:

  1. [Entidad1] - campos inferidos: [lista]
  2. [Entidad2] - campos inferidos: [lista]

¿Está bien?
  A) Sí, continuar
  B) No, quiero agregar/quitar/modificar

Tu respuesta [default: A]:
```

Si elige B → pedir correcciones en texto libre y actualizar.

#### Pregunta 4c: Descripción funcional por entidad

Para CADA entidad detectada:

```
Para [Entidad1]:

¿Qué operaciones necesitás?
(Ej: crear, consultar por ID, listar, actualizar, eliminar, cambiar estado, etc.)

¿Hay alguna lógica de negocio especial?
(Ej: "no se puede eliminar si tiene órdenes", "el email es único")

¿Validaciones importantes?
(Ej: "nombre obligatorio, máximo 50 caracteres", "password mínimo 8 chars")

Tu descripción para [Entidad1] (o ENTER si ya lo describiste en P4):
```

#### Pregunta 4d: Criterios de aceptación por entidad

Para CADA entidad:

```
Para [Entidad1] definiste:
"[resumen de operaciones y reglas]"

¿Cuáles son los criterios de aceptación?
(Condiciones verificables que deben cumplirse para considerar la feature completa)

Ej:
- POST /roles crea un rol nuevo (201)
- DELETE /roles/{id} retorna 400 si el rol es predefinido
- El nombre del rol debe ser único

Tus criterios para [Entidad1] (o ENTER para omitir):
```

#### Pregunta 5: Integraciones

```
Pregunta 5: ¿Este servicio se integra con otros servicios o sistemas?

Ej: "Necesito llamar a notification-service para enviar emails"
    "Consume datos de product-service"

Tu respuesta (o ENTER si no hay integraciones):
```

#### Pregunta 6: Fuera de alcance

```
Pregunta 6: ¿Qué queda fuera de esta versión?

Ej: "No incluir autenticación JWT (lo hace un API Gateway)"
    "Sin manejo de pagos por ahora"

Tu respuesta (o ENTER si no hay restricciones):
```

#### Generar PRD

1. Leer template `.cursor/templates/prd.template.md`
2. Completar con todas las respuestas
3. Guardar en `delivery/prd/[PROJECT_NAME].prd.md`

#### PAUSA 1 (OBLIGATORIA)

```
════════════════════════════════════════════════════════════════
  PRD GENERADO
════════════════════════════════════════════════════════════════

Creé el PRD en:
  delivery/prd/[PROJECT_NAME].prd.md

Por favor revisalo. Cuando estés listo:
  - "ok" o "continuar" → seguir con las preguntas técnicas
  - "modificar" → hacé cambios al archivo primero

⚠️ NO voy a continuar hasta que confirmes.
════════════════════════════════════════════════════════════════
```

**ESPERAR respuesta. NO continuar automáticamente.**

---

### FASE 2: Preguntas Técnicas → RFC

#### Pregunta T1: Autor

```
Pregunta T1: Autor del proyecto

¿Cuál es tu nombre y email? (para metadata del RFC)
Ej: Juan Perez, juan@example.com

Tu respuesta (o ENTER para omitir):
```

#### Pregunta T2: Identificadores Maven

Antes de mostrar la pregunta, inferir sugerencias basadas en lo recolectado:

```
INFERIR:
  - SUGGESTED_GROUP_ID:
      SI existe proyecto padre (Ej: "whatsapp-banking-api")
        → tomar dominio invertido del contexto (Ej: la.ingenia)
      SI se detectó pom.xml con groupId
        → usar ese valor
      SINO
        → usar "com.example" como placeholder

  - SUGGESTED_ARTIFACT_ID:
      → usar PROJECT_NAME tal como fue confirmado en P1
        (Ej: "user-services-api")

  - SUGGESTED_PACKAGE:
      → SUGGESTED_GROUP_ID + "." + módulo principal del proyecto
        (Ej: la.ingenia + users → "la.ingenia.users")
        El módulo se infiere del PROJECT_NAME:
          "user-services-api"    → users
          "product-catalog-api"  → products
          "notification-service" → notification
          "account-services"     → accounts
```

Luego mostrar la pregunta con las sugerencias:

```
Pregunta T2: Identificadores Maven/Gradle

Basándome en lo que me contaste, sugiero:

  Group ID:    [SUGGESTED_GROUP_ID]
  Artifact ID: [SUGGESTED_ARTIFACT_ID]
  Package base: [SUGGESTED_PACKAGE]

¿Está bien?
  A) Sí, usar estos valores
  B) Modificar Group ID
  C) Modificar todos

Tu respuesta [default: A]:
```

Si elige B → pedir solo el Group ID y recalcular Package base automáticamente.
Si elige C → pedir los 3 valores en formato: `group_id, artifact_id, package_base`.

#### Pregunta T3: Stack Tecnológico

**ACCIÓN OBLIGATORIA:** Leer y mostrar el contenido completo de `rules/10-stack.mdc`.

```
Pregunta T3: Stack Tecnológico

El stack por defecto del proyecto es:

════════════════════════════════════════════════════════════════
[CONTENIDO COMPLETO DE rules/10-stack.mdc]
════════════════════════════════════════════════════════════════

¿Qué querés hacer?
  A) Usar este stack por defecto
  B) Personalizar (cambiar DB, versiones, agregar dependencias)

Tu respuesta [default: A]:
```

Si elige B → preguntar qué quiere cambiar en texto libre.

#### Pregunta T4: Arquitectura

```
Pregunta T4: Arquitectura

  A) MVC - Controller → Service → Repository [default]
     Más simple, ideal para CRUD y APIs REST estándar

  B) Hexagonal - Domain → Application → Adapter
     Más compleja, ideal para lógica de negocio rica

Tu respuesta [default: A]:
```

#### Pregunta T5: Documentación

```
Pregunta T5: Documentación a generar

  A) Swagger + Postman Collection [default]
  B) Swagger + Postman + Javadoc
  C) Solo Swagger
  D) Ninguna

Tu respuesta [default: A]:
```

#### Pregunta T6: Coverage

```
Pregunta T6: Coverage mínimo de tests

  A) 80% [default, recomendado]
  B) 70%
  C) Otro porcentaje (ingresá el número)
  D) No verificar coverage

Tu respuesta [default: A]:
```

#### Pregunta T7: Optimización de startup

```
Pregunta T7: Optimización de startup

  A) Ninguna [default]
  B) CDS - Class Data Sharing
     ~20% reducción en tiempo de arranque
     Sin cambios en código, fácil de implementar

  C) AOT - Ahead Of Time Compilation
     ~50% reducción en tiempo de arranque
     Compatible con Spring Boot 3.2+, complejidad media

  D) Native Image (GraalVM)
     Arranque en 50-200ms, ~80% menos RAM
     Requiere GraalVM, alta complejidad, restricciones de reflexión

Tu respuesta [default: A]:
```

#### Pregunta T8: Git automático

```
Pregunta T8: Gestión de Git

¿Querés que el cyborg gestione Git automáticamente?
(crear branches por feature y commits al finalizar)

  A) Sí, automático [default]
  B) No, lo gestiono yo

Tu respuesta [default: A]:
```

#### Pregunta T9: Reportes al finalizar

```
Pregunta T9: Reportes al finalizar la ejecución

  A) Dashboard + Coverage Report [default]
  B) Solo Dashboard
  C) Solo Coverage Report
  D) Ninguno

Tu respuesta [default: A]:
```

#### Pregunta T10: Confirmación resumen técnico

```
RESUMEN TÉCNICO
══════════════════════════════════════════════════════════════

Proyecto: [PROJECT_NAME]
Proyecto padre: [PARENT_PROJECT o "independiente"]
Group ID: [GROUP_ID]
Artifact ID: [ARTIFACT_ID]
Package base: [BASE_PACKAGE]
Arquitectura: [ARCHITECTURE]
Optimización: [OPTIMIZATION]
Documentación: [DOCS]
Coverage: [COVERAGE]
Git automático: [GIT]
Reportes: [OUTPUT]

══════════════════════════════════════════════════════════════

¿Confirmás? (si/no/modificar):
```

#### Generar RFC via prd-to-rfc

```
╔══════════════════════════════════════════════════════════════════════╗
║  STOP - ACCIÓN OBLIGATORIA                                           ║
║                                                                      ║
║  DEBES invocar la herramienta Task() AHORA.                          ║
║  NO leas prd-to-rfc.md vos mismo.                                    ║
║  NO generes el RFC vos mismo.                                        ║
║  USA la herramienta Task() para delegar a un subagente.              ║
║                                                                      ║
║  Si estás leyendo esto y pensás en hacerlo vos → STOP. USA Task().  ║
╚══════════════════════════════════════════════════════════════════════╝
```

**⚠️ FALLBACK OBLIGATORIO — Solo si Task() no está disponible:**

Si por alguna razón técnica no podés invocar la herramienta `Task()`, entonces:

```
╔══════════════════════════════════════════════════════════════════════╗
║  FALLBACK: GENERAR RFC MANUALMENTE                                   ║
║                                                                      ║
║  1. LEER obligatoriamente: .cursor/templates/rfc.template.md        ║
║  2. Usar ese template como base — NO inventar formato               ║
║  3. El RFC DEBE ser YAML (no Markdown con tablas)                   ║
║  4. Respetar TODAS las secciones del template:                      ║
║     operation_type, meta, author, project, integrations,            ║
║     optimization, docs, coverage, git, output, features[]           ║
║  5. Cada feature DEBE incluir: entity.fields[], endpoints[]         ║
║     con request_body y responses[] con ejemplos JSON reales         ║
╚══════════════════════════════════════════════════════════════════════╝
```

Invocar la herramienta **Task** con estos parámetros:

```
Task(
  subagent_type="prd-to-rfc",
  description="Transformar PRD en RFC técnico",
  prompt="PRD a transformar: delivery/prd/[PROJECT_NAME].prd.md
RFC a generar:     delivery/rfc/[PROJECT_NAME].rfc.md

Datos técnicos para completar el RFC:
author:
  name: [AUTHOR_NAME]
  email: [AUTHOR_EMAIL]
project:
  name: [PROJECT_NAME]
  group_id: [GROUP_ID]
  artifact_id: [ARTIFACT_ID]
  base_package: [BASE_PACKAGE]
  architecture: [ARCHITECTURE]
  parent_project: [PARENT_PROJECT]
optimization: [OPTIMIZATION]
docs:
  swagger: [SWAGGER]
  postman: [POSTMAN]
  javadoc: [JAVADOC]
coverage:
  min_line: [COVERAGE]
git:
  auto_branch: [GIT_BRANCH]
  auto_commit_message: [GIT_COMMIT]
output:
  dashboard: [DASHBOARD]
  coverage_report: [COVERAGE_REPORT]"
)
```

#### PAUSA 2 (OBLIGATORIA)

```
════════════════════════════════════════════════════════════════
  RFC GENERADO
════════════════════════════════════════════════════════════════

Creé el RFC en:
  delivery/rfc/[PROJECT_NAME].rfc.md

Por favor revisalo. Cuando estés listo:
  - "ejecutar" → comenzar la ejecución del plan
  - "modificar" → hacé cambios al RFC primero

⚠️ NO voy a continuar hasta que confirmes.
════════════════════════════════════════════════════════════════
```

**ESPERAR respuesta. NO continuar automáticamente.**

---

## PASO 5: Invocar Orquestador (si confirma "ejecutar")

```
╔══════════════════════════════════════════════════════════════════════╗
║  STOP - ACCIÓN OBLIGATORIA                                           ║
║                                                                      ║
║  DEBES invocar la herramienta Task() AHORA.                          ║
║  NO ejecutes el orquestador vos mismo.                               ║
║  USA la herramienta Task() para delegar a un subagente.              ║
╚══════════════════════════════════════════════════════════════════════╝
```

Invocar la herramienta **Task** con estos parámetros:

```
Task(
  subagent_type="generalPurpose",
  description="Ejecutar orquestador",
  prompt="Leer y ejecutar el skill .cursor/skills/07-orchestrator.skill.md

RFC: delivery/rfc/[PROJECT_NAME].rfc.md

Leer el skill y ejecutar el plan completo."
)
```

---

## FLUJO B: Agregar Features

### FASE 1 - Negocio

```
P1.  Detectar entidades/módulos existentes en el proyecto
     (leer src/main/java/**/*Controller.java para inferir módulos)
P2.  ¿Qué features nuevas necesitás?
P2b. Descripción funcional por feature nueva (operaciones, lógica, validaciones)
P2c. Criterios de aceptación por feature
P3.  ¿Integraciones nuevas con otros servicios?
P4.  ¿Qué queda fuera de esta iteración?

→ PAUSA 1: usuario revisa PRD
```

### FASE 2 - Técnico (simplificada)

```
T1.  Leer pom.xml para confirmar stack existente
T2.  ¿Cambiar algo del stack? (default: No)
T3.  ¿Agregar optimización? (si no la tiene, default: No)

→ Generar RFC de features nuevas
→ PAUSA 2: usuario revisa RFC
```

---

## FLUJO C: Modificar Código

```
P1.  ¿Qué archivo/clase necesita cambios?
P2.  ¿Qué cambio necesitás?
P3.  ¿Comportamiento esperado?

→ Genera RFC simplificado (sin PRD)
→ PAUSA: usuario confirma
```

---

## FLUJO D: Corregir Bug

```
P1.  ¿Comportamiento actual (lo que falla)?
P2.  ¿Comportamiento esperado?
P3.  ¿Mensaje de error o stacktrace? (pegalo o ENTER para omitir)

→ Genera RFC simplificado (sin PRD)
→ PAUSA: usuario confirma
```

---

## REGLAS

1. Hacer UNA pregunta a la vez
2. Esperar respuesta antes de continuar
3. Validar respuestas y ofrecer defaults claros
4. Mostrar resumen antes de confirmar
5. **SIEMPRE pausar después de generar PRD**
6. **SIEMPRE pausar después de generar RFC**
7. **NUNCA ejecutar el plan sin confirmación explícita**
8. **NUNCA inferir depends_on preguntando al usuario — inferirlo del PRD**
