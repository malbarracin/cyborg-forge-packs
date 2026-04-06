# SKILL: MULTI-FEATURE ORCHESTRATOR (Cursor)

Orquestador principal que **SOLO COORDINA** subagents. **NUNCA ejecuta tareas directamente.**

---

## 🛑🛑🛑 REGLA FUNDAMENTAL - LEER ANTES DE HACER CUALQUIER COSA 🛑🛑🛑

```
╔══════════════════════════════════════════════════════════════════════════════╗
║                                                                              ║
║   EL ORQUESTADOR NO EJECUTA NADA. SOLO DELEGA A SUBAGENTES.                  ║
║                                                                              ║
║   Si estás por ejecutar un comando (mvn, npm, docker, etc.) → STOP           ║
║   Si estás por crear un archivo .java → STOP                                 ║
║   Si estás por ejecutar tests → STOP                                         ║
║   Si estás por compilar → STOP                                               ║
║                                                                              ║
║   DELEGA AL SUBAGENTE CORRESPONDIENTE USANDO Task()                          ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

## ❌ PROHIBIDO PARA EL ORQUESTADOR (VIOLACIÓN = ERROR GRAVE)

| ACCIÓN | PROHIBIDO | QUIÉN LO HACE |
|--------|-----------|---------------|
| Ejecutar `mvn`, `gradle`, `npm` | ❌ PROHIBIDO | → `verifier` o `feature-executor` |
| Ejecutar tests | ❌ PROHIBIDO | → `verifier` o `test-generator` |
| Compilar código | ❌ PROHIBIDO | → `verifier` o `feature-executor` |
| Crear archivos `.java` | ❌ PROHIBIDO | → `feature-executor` o `scaffold-generator` |
| Crear archivos `.xml`, `.yml` | ❌ PROHIBIDO | → `scaffold-generator` o `feature-merger` |
| Modificar código existente | ❌ PROHIBIDO | → `feature-executor` |
| Ejecutar `docker` | ❌ PROHIBIDO | → `verifier` |
| Analizar coverage | ❌ PROHIBIDO | → `coverage-analyzer` |
| Generar documentación | ❌ PROHIBIDO | → `doc-generator` |

---

## ✅ PERMITIDO PARA EL ORQUESTADOR (SOLO ESTO)

```
✅ Leer el archivo .input.md (Read tool)
✅ Llamar MCP de memoria (mem_context, mem_save, mem_session_start, etc.)
✅ Delegar tareas via Task(subagent_type="...", prompt="...")
✅ Mostrar mensajes al usuario
✅ Hacer preguntas interactivas
✅ Esperar resultados de subagentes
✅ Pasar información entre subagentes
```

**SI NO ESTÁ EN ESTA LISTA → NO LO HAGAS. DELEGA.**

---

## 📢 MENSAJE OBLIGATORIO ANTES DE CADA DELEGACIÓN

**ANTES de cada llamada a `Task()`, SIEMPRE mostrar este mensaje:**

```
════════════════════════════════════════════════════════════════
  DELEGANDO!!!!
  → Agente: [nombre-del-subagente]
  → Tarea: [descripción breve de lo que hará]
════════════════════════════════════════════════════════════════
```

**Ejemplo:**

```
════════════════════════════════════════════════════════════════
  DELEGANDO!!!!
  → Agente: scaffold-generator
  → Tarea: Crear estructura base del proyecto
════════════════════════════════════════════════════════════════
```

**ESTO ES OBLIGATORIO. NO OMITIR.**

---

## 🚨 FLUJO OBLIGATORIO

```
PASO 1: CONECTAR A MEMORIA (mem_context + mem_session_start)
PASO 2: Validar Input → Task(input-validator)
PASO 3: Scaffold → Task(scaffold-generator)
PASO 4: Features → Task(feature-executor) x N
PASO 5: Merge → Task(feature-merger)
PASO 6: Coverage → Task(coverage-analyzer) + Task(test-generator)
PASO 7: Docs → Task(doc-generator)
PASO 8: Dashboard → Task(dashboard-generator)
PASO 9: Optimización → Task(optimization-generator)
PASO 10: Verificación → Task(verifier)  ← ESTE EJECUTA mvn verify, NO EL ORQUESTADOR
PASO 11: Guardar memoria (mem_save + mem_save_stats + mem_session_end)
PASO 12: Output final
```

## ⚠️ PREREQUISITO: ARCHIVO .input.md

```
╔══════════════════════════════════════════════════════════════════════════════╗
║  ESTE ORQUESTADOR REQUIERE UN ARCHIVO .input.md                              ║
║                                                                              ║
║  Si el usuario NO proporciona un .input.md:                                  ║
║  → Indicarle que use el skill "analyst" para generarlo                       ║
║  → O que use el template en .cursor/templates/multi-feature-input.template.md║
║                                                                              ║
║  NO generar el input aquí. Esa es responsabilidad del ANALYST.               ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

## ⚠️⚠️⚠️ PASOS CRÍTICOS QUE NO SE PUEDEN OMITIR ⚠️⚠️⚠️

```
╔══════════════════════════════════════════════════════════════════════════════╗
║  PASO 11 ES OBLIGATORIO - SIN EXCEPCIÓN                                      ║
║                                                                              ║
║  ANTES de mostrar el output final, DEBES ejecutar:                           ║
║                                                                              ║
║  1. mem_save (service_contract) → Guardar contrato del servicio              ║
║  2. mem_save (decision) → Guardar decisiones arquitectónicas                 ║
║  3. mem_save_stats → Guardar estadísticas para dashboard                     ║
║  4. mem_session_end → Cerrar sesión                                          ║
║                                                                              ║
║  SI NO EJECUTAS mem_save_stats, EL DASHBOARD NO FUNCIONARÁ                   ║
║                                                                              ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

---

## PASO 0: Verificar Input

Buscar input en este orden de prioridad:

```
1. SI el usuario adjuntó un .rfc.md    → usar RFC
2. SI el usuario adjuntó un .input.md  → usar input (legacy)
3. SI existe delivery/rfc/*.rfc.md     → usar RFC
4. SI existe delivery/input/*.input.md → usar input (legacy)
5. SI existe delivery/prd/*.prd.md     → invocar prd-to-rfc primero
6. Ninguno                             → redirigir al analyst
```

**CASOS 1-4:** Leer el archivo y continuar con PASO 1.

**CASO 5: Existe PRD pero no RFC:**

```
╔══════════════════════════════════════════════════════════════════════╗
║  STOP - ACCIÓN OBLIGATORIA                                           ║
║                                                                      ║
║  DEBES invocar la herramienta Task() AHORA.                          ║
║  NO leas prd-to-rfc.md vos mismo.                                    ║
║  NO generes el RFC vos mismo.                                        ║
║  USA la herramienta Task() para delegar a un subagente.              ║
╚══════════════════════════════════════════════════════════════════════╝
```

Invocar la herramienta **Task** con estos parámetros:

```
Task(
  subagent_type="prd-to-rfc",
  description="Convertir PRD a RFC",
  prompt="PRD: [ruta al .prd.md encontrado]
RFC destino: delivery/rfc/[nombre].rfc.md"
)
```

Esperar que el subagente termine. Luego leer el RFC generado y continuar con PASO 1.

**CASO 6: No hay ningún documento:**

```
════════════════════════════════════════════════════════════
  ORQUESTADOR - INPUT REQUERIDO
════════════════════════════════════════════════════════════

No detecté ningún documento de input.

Opciones disponibles:

1. Usar el ANALYST (recomendado):
   → Genera PRD + RFC con preguntas guiadas
   → Skill: .cursor/skills/01-analyst.skill.md

2. Escribir el PRD manualmente:
   → Template: .cursor/templates/prd.template.md
   → Guardar en: delivery/prd/[proyecto].prd.md

3. Escribir el RFC directamente:
   → Template: .cursor/templates/rfc.template.md
   → Guardar en: delivery/rfc/[proyecto].rfc.md

════════════════════════════════════════════════════════════
```

**FIN.** No continuar sin input.

---

## PASO 1: CONECTAR A MEMORIA (OBLIGATORIO)

Conectarse a memoria con los datos del input.

### 1.1 Determinar proyecto de memoria

```
SI project.parent_project existe en input:
  MEMORY_PROJECT = project.parent_project
SINO:
  MEMORY_PROJECT = project.name
```

### 1.2 Obtener contexto

```
CallMcpTool(
  server: "user-cyborg-memory",
  toolName: "mem_context",
  arguments: { "project": "[MEMORY_PROJECT]" }
)
```

**GUARDAR el resultado completo en variable `MEMORY_CONTEXT`** para pasarlo a los subagentes.

Esto recupera:
- Decisiones previas del proyecto
- Patrones de implementación
- Información de onboarding
- Contratos de servicios relacionados

### 1.3 Iniciar sesión de memoria

```
CallMcpTool(
  server: "user-cyborg-memory", 
  toolName: "mem_session_start",
  arguments: { "project": "[MEMORY_PROJECT]" }
)
```

**GUARDAR el `session_id` retornado en variable `SESSION_ID`** para pasarlo a los subagentes.

### 1.4 Variables para subagentes

Después de este paso, el orquestador tiene:
```
MEMORY_PROJECT = "[nombre del proyecto]"
MEMORY_CONTEXT = "[contexto completo recuperado de mem_context]"
SESSION_ID = "[session_id de mem_session_start]"
```

**IMPORTANTE:** Estas 3 variables DEBEN incluirse en el prompt de CADA Task que se delegue.

### 1.5 Variables para estadísticas (Dashboard)

Inicializar variables para acumular estadísticas durante la ejecución:
```
START_TIME = [timestamp actual]
STATS = {
  "features_completed": 0,
  "total_files_created": 0,
  "total_tests": 0,
  "coverage_final": 0,
  "duration_seconds": 0,
  "rules_applied": {
    "stack": [],       # 10-19
    "architecture": [], # 11, 20-29
    "code_style": [],   # 20-29
    "testing": [],      # 40-49
    "docs": [],         # 50-59
    "orchestration": [] # 60-69
  },
  "features": []  # Array de stats por feature
}
```

**IMPORTANTE:** Después de CADA Task de `feature-executor`, `scaffold-generator`, `test-generator`, `doc-generator`, extraer `rules_applied` del resultado y acumular en `STATS`.

### Formato para incluir en cada Task

```
## CONTEXTO DE MEMORIA
project: [MEMORY_PROJECT]
session_id: [SESSION_ID]

[MEMORY_CONTEXT]
```

El `[MEMORY_CONTEXT]` incluye:
- Decisiones previas del proyecto
- Patrones de implementación usados
- Información de onboarding (stack, convenciones)
- Contratos de servicios relacionados (endpoints, DTOs, ejemplos de integración)

### 1.5 Mostrar confirmación al usuario

```
════════════════════════════════════════════════════════════
  CYBORG MEMORY CONECTADA
════════════════════════════════════════════════════════════
Proyecto: [MEMORY_PROJECT]
Sesión: [SESSION_ID]
Contexto cargado: [resumen de lo recuperado]
════════════════════════════════════════════════════════════
```

**SI LA CONEXIÓN A MEMORIA FALLA:** Informar al usuario y preguntar si desea continuar sin memoria.

**NO CONTINUAR SIN EJECUTAR ESTE PASO.**

---

## PASO 1.5: Detectar Integraciones con Servicios Externos

Después de conectar a memoria y leer el RFC (o input.md), verificar si existe la sección `integrations`:

### Leer integrations del RFC

```
SI el RFC tiene:
  integrations:
    - service: "user-services-api"
      reason: "validar que el usuario existe"
      data_needed: ["userId", "userStatus"]

→ INTEGRATIONS_FOUND = true
→ INTEGRATIONS_LIST = [lista de servicios]
```

### Por cada servicio en la lista, buscar su contrato en Cyborg Memory

```
╔══════════════════════════════════════════════════════════════════════╗
║  ACCIÓN OBLIGATORIA: Buscar contratos de servicios dependientes      ║
╚══════════════════════════════════════════════════════════════════════╝
```

Para cada `integration` detectada:

```
CallMcpTool(
  server: "user-cyborg-memory",
  toolName: "mem_context",
  arguments: {
    "project": "[nombre del servicio, ej: user-services-api]",
    "query": "service contract endpoints API"
  }
)
```

Guardar el resultado en:

```
INTEGRATIONS_CONTEXT = {
  "user-services-api": {
    "reason": "validar que el usuario existe",
    "data_needed": ["userId", "userStatus"],
    "contract": "[contenido retornado por mem_context]"
  },
  ...
}
```

### Mostrar al usuario el resultado

**Si se encontraron integraciones y contratos:**
```
════════════════════════════════════════════════════════════════════
  🔗 INTEGRACIONES DETECTADAS
════════════════════════════════════════════════════════════════════
  Servicios dependientes encontrados en RFC:

  ✅ user-services-api
     Motivo: validar que el usuario existe
     Datos necesarios: userId, userStatus
     Contrato en memoria: ✅ encontrado ([N] observaciones)

  El feature-executor recibirá este contexto para generar
  los Feign Clients y la integración correspondiente.
════════════════════════════════════════════════════════════════════
```

**Si no hay contrato en memoria para algún servicio:**
```
════════════════════════════════════════════════════════════════════
  ⚠️  INTEGRACIÓN SIN CONTRATO EN MEMORIA
════════════════════════════════════════════════════════════════════
  El servicio "user-services-api" está listado en las integraciones
  pero no se encontró su contrato en Cyborg Memory.

  Opciones:
  1. Continuar sin contexto de integración (el feature-executor
     generará un Feign Client genérico)
  2. Cancelar y ejecutar primero el servicio "user-services-api"
     para que su contrato quede registrado en memoria.
════════════════════════════════════════════════════════════════════
Esperando respuesta del usuario...
```

**Si `integrations` está vacío o no existe:**
```
INTEGRATIONS_CONTEXT = {}
```
→ Continuar sin contexto de integración.

---

## PASO 2: Validar Input

Delegar al subagent `input-validator`:

```
Task(
  subagent_type="input-validator",
  description="Validar input.md",
  prompt="Valida el archivo de input: [ruta al input.md]"
)
```

**SI validation_result.status = INVALID:**
- Mostrar errores al usuario
- NO continuar

**SI validation_result.status = VALID:**
- Continuar con PASO 3

---

## PASO 3: Scaffold (si proyecto nuevo)

**SI mode = CREATE_FROM_ZERO:**

Delegar al subagent `scaffold-generator`:

```
Task(
  subagent_type="scaffold-generator",
  description="Crear estructura base",
  prompt="Genera el scaffold para:
  
project:
  name: [project.name]
  base_package: [project.base_package]
  architecture: [project.architecture]
  
docs:
  swagger: [docs.swagger]

## CONTEXTO DE MEMORIA
project: [MEMORY_PROJECT]
session_id: [SESSION_ID]

[MEMORY_CONTEXT]
  
Aplicar rules: 10-stack.mdc, 11-architecture-[arch].mdc, 14-docker-support.mdc"
)
```

**Esperar resultado antes de continuar.**

#### Verificar y compensar mem_save del scaffold

```
SI el resultado del scaffold-generator NO contiene "Memory saved" o "mem_save":

  CallMcpTool(
    server: "user-cyborg-memory",
    toolName: "mem_save",
    arguments: {
      "title": "Scaffold - [project.name]",
      "type": "inter_agent",
      "project": "[MEMORY_PROJECT]",
      "session_id": "[SESSION_ID]",
      "content": "[RESULTADO COMPLETO DEL SCAFFOLD-GENERATOR]",
      "importance": 8,
      "topic_key": "[project.name]-scaffold"
    }
  )
```

---

## PASO 4: Ejecutar Features (paralelo)

Para cada feature en el input, delegar al subagent `feature-executor`.

**Ejecutar en paralelo (máximo 4 por batch):**

```
// Feature 1
Task(
  subagent_type="feature-executor",
  description="Implementar [feature1.module]",
  prompt="Implementa el módulo [feature1.module]:
  
feature:
  name: [feature1.name]
  module: [feature1.module]
  type: [feature1.type]
  entity: [feature1.entity]
  endpoints: [feature1.endpoints]
  
project:
  name: [project.name]
  base_package: [project.base_package]
  architecture: [project.architecture]

## CONTEXTO DE MEMORIA
project: [MEMORY_PROJECT]
session_id: [SESSION_ID]

[MEMORY_CONTEXT]

## INTEGRACIONES CON SERVICIOS EXTERNOS
[SI INTEGRATIONS_CONTEXT está vacío → omitir esta sección]
[SI INTEGRATIONS_CONTEXT tiene datos → incluir:]

Este servicio depende de los siguientes servicios externos.
Debes generar los Feign Clients, DTOs de respuesta y la lógica
de integración para cada uno.

[Para cada servicio en INTEGRATIONS_CONTEXT:]
### Servicio: [nombre-del-servicio]
Motivo: [reason]
Datos necesarios: [data_needed]
Contrato disponible en memoria:
[contract - el texto completo retornado por mem_context]

Instrucciones para el Feign Client:
- Crear `[NombreServicio]Client` en el paquete `infrastructure.client`
- Crear `[NombreServicio]ClientFallback` para resiliencia
- Crear DTOs de respuesta para los campos: [data_needed]
- Configurar `@FeignClient(name="[nombre-servicio]", url="${services.[nombre].url}")`
- Agregar la URL en `application.yml` bajo `services.[nombre].url`
- Inyectar el client en el Service donde sea necesario
[fin del loop]
  
isolation:
  enabled: true
  output_dependencies: true
  output_configs: true"
)

// Feature 2, 3, 4... (mismo formato, incluir CONTEXTO DE MEMORIA e INTEGRACIONES en cada uno)
```

**Esperar que TODAS las features del batch terminen antes de continuar.**

### 4.1 Procesar resultado de cada feature-executor

Después de que **cada** `Task(feature-executor)` retorne, ejecutar este proceso:

#### Paso A: Leer el resultado

El resultado del subagente es la respuesta final que escribió. Buscar en ese texto:

```
BUSCAR en el resultado del subagente:
  - Sección "Archivos Creados" → contar archivos
  - Sección "Tests" → extraer total
  - Sección "Endpoints Creados" → contar endpoints
  - Sección "Reglas Aplicadas" → extraer lista de reglas
  - Línea "AGENT_RESULT:" → si existe, extraer valores directamente
  - Confirmación "mem_save" ejecutado → buscar "Memory saved" o "mem_save" en resultado
```

#### Paso B: Acumular estadísticas

```
STATS.features_completed += 1
STATS.total_files_created += [archivos_creados extraídos]
STATS.total_tests += [tests extraídos]

Para cada rule en [rules_applied extraídas]:
  categoria = categorizar_regla(rule)
  agregar a STATS.rules_applied[categoria] si no está ya

STATS.features.append({
  "name": [nombre de la feature],
  "module": [módulo],
  "files_created": [N],
  "tests": [N],
  "rules_applied": [lista]
})
```

**Función categorizar_regla(rule):**
```
10-19 → "stack"
20-29 → "code_style"
30-39 → "architecture"
40-49 → "testing"
50-59 → "docs"
60-69 → "orchestration"
```

#### Paso C: Verificar y compensar mem_save

```
SI el resultado del subagente NO contiene "Memory saved" o "mem_save":

  → El subagente NO guardó en memoria. El orquestador lo hace ahora.

  CallMcpTool(
    server: "user-cyborg-memory",
    toolName: "mem_save",
    arguments: {
      "title": "[feature.name] implementation - [project.name]",
      "type": "inter_agent",
      "project": "[MEMORY_PROJECT]",
      "session_id": "[SESSION_ID]",
      "content": "[RESULTADO COMPLETO DEL SUBAGENTE]",
      "importance": 8,
      "topic_key": "[project.name]-[feature.module]-impl"
    }
  )

SI el resultado SÍ contiene confirmación de mem_save:
  → No hacer nada, el subagente ya lo guardó.
```

#### Paso D: Mensaje al usuario

Después de cada feature completada, mostrar:

```
════════════════════════════════════════════════════════════════
  ✅ FEATURE COMPLETADA: [feature.name]
  📁 Archivos: [N] creados
  🧪 Tests: [N]
  💾 Guardado en memoria: [✅ por subagente / ✅ por orquestador]
  
  Progreso: [X]/[TOTAL] features
════════════════════════════════════════════════════════════════
```

---

## PASO 5: Merge de Resultados

Delegar al subagent `feature-merger`:

```
Task(
  subagent_type="feature-merger",
  description="Merge de features",
  prompt="Combina los resultados de las features ejecutadas:
  
project:
  name: [project.name]
  base_package: [project.base_package]
  architecture: [project.architecture]
  
features_results:
  - feature: [feature1]
    dependencies_required: [del feature-executor]
    configs_required: [del feature-executor]
  - feature: [feature2]
    ...

## CONTEXTO DE MEMORIA
project: [MEMORY_PROJECT]
session_id: [SESSION_ID]

[MEMORY_CONTEXT]
    
GENERAR: pom.xml actualizado, application.yml consolidado"
)
```

---

## PASO 6: Verificación de Coverage (si aplica)

**SI coverage.min_line existe en el input:**

### 6.1 Analizar coverage

```
Task(
  subagent_type="coverage-analyzer",
  description="Analizar coverage",
  prompt="Analiza el reporte JaCoCo:
  
coverage:
  min_line: [coverage.min_line]

## CONTEXTO DE MEMORIA
project: [MEMORY_PROJECT]
session_id: [SESSION_ID]

[MEMORY_CONTEXT]"
)
```

### 6.2 Si coverage < threshold, generar tests

```
Task(
  subagent_type="test-generator",
  description="Generar tests",
  prompt="Genera tests para las clases con bajo coverage:
  
threshold: [coverage.min_line]
belowThreshold: [lista del coverage-analyzer]

## CONTEXTO DE MEMORIA
project: [MEMORY_PROJECT]
session_id: [SESSION_ID]

[MEMORY_CONTEXT]"
)
```

**Repetir hasta MAX 3 iteraciones o hasta cumplir threshold.**

#### Verificar y compensar mem_save del test-generator

```
SI el resultado del test-generator NO contiene "Memory saved" o "mem_save":

  CallMcpTool(
    server: "user-cyborg-memory",
    toolName: "mem_save",
    arguments: {
      "title": "Tests generados - [project.name]",
      "type": "inter_agent",
      "project": "[MEMORY_PROJECT]",
      "session_id": "[SESSION_ID]",
      "content": "[RESULTADO COMPLETO DEL TEST-GENERATOR]",
      "importance": 7,
      "topic_key": "[project.name]-tests-gen"
    }
  )
```

Actualizar estadísticas:
```
STATS.total_tests += [tests generados extraídos del resultado]
```

---

## PASO 7: Documentación (paralelo)

**SI docs está habilitado en el input:**

Para cada feature, delegar al subagent `doc-generator`:

```
Task(
  subagent_type="doc-generator",
  description="Documentar [feature.name]",
  prompt="Genera documentación para:
  
feature:
  name: [feature.name]
  module: [feature.module]
  
files_created: [lista]
tests_created: [cantidad]
coverage: [porcentaje]

## CONTEXTO DE MEMORIA
project: [MEMORY_PROJECT]
session_id: [SESSION_ID]

[MEMORY_CONTEXT]

GENERAR: dod.md, handoff.md, pr.md en delivery/features/[NN]-[feature]/"
)
```

---

## PASO 8: Dashboard Global

Delegar al subagent `dashboard-generator`:

```
Task(
  subagent_type="dashboard-generator",
  description="Generar dashboard",
  prompt="Genera el dashboard global:
  
project:
  name: [project.name]
  architecture: [project.architecture]
  
features: [lista con stats de cada feature]

## CONTEXTO DE MEMORIA
project: [MEMORY_PROJECT]
session_id: [SESSION_ID]

[MEMORY_CONTEXT]

USAR template: .cursor/templates/stats-dashboard.template.html
GENERAR en: delivery/reports/dashboard/"
)
```

---

## PASO 9: Optimización (si aplica)

**SI project.optimization != "none":**

```
Task(
  subagent_type="optimization-generator",
  description="Generar optimización",
  prompt="Genera configuración de optimización:
  
project:
  name: [project.name]
  optimization: [cds|aot|native]

## CONTEXTO DE MEMORIA
project: [MEMORY_PROJECT]
session_id: [SESSION_ID]

[MEMORY_CONTEXT]
  
APLICAR rule: 45-optimization-[tipo].mdc"
)
```

---

## PASO 10: Verificación Final

Delegar al subagent `verifier`:

```
Task(
  subagent_type="verifier",
  description="Verificar proyecto",
  prompt="Verifica que todo funciona:
  
project:
  name: [project.name]
  path: [project_path]
  
coverage:
  min_line: [del input o null]

## CONTEXTO DE MEMORIA
project: [MEMORY_PROJECT]
session_id: [SESSION_ID]

[MEMORY_CONTEXT]
  
EJECUTAR: ./mvnw clean verify
APLICAR checklist: rules/62-validation-checklist.mdc"
)
```

---

## PASO 11: Guardar en Memoria

### 11.1 Guardar resumen de sesión

```
Llamar MCP tool: mem_session_summary
Argumentos: {
  "session_id": "[SESSION_ID]",
  "summary": "Proyecto [project.name]: [N] features implementadas, coverage [X]%"
}
```

### 11.2 Guardar decisiones importantes

```
Llamar MCP tool: mem_save
Argumentos: {
  "content": "[decisiones arquitectónicas tomadas]",
  "type": "decision",
  "project": "[MEMORY_PROJECT]",
  "session_id": "[SESSION_ID]",
  "topic_key": "[project.name]-decisions",
  "importance": 8
}
```

### 11.3 Guardar contrato del servicio

```
Llamar MCP tool: mem_save
Argumentos: {
  "title": "[project.name]",
  "content": "## Servicio: [project.name]

### Base URL
http://[project.name]:8080

### Endpoints
[lista de endpoints generados]

### DTOs
[lista de DTOs]

### Ejemplo Feign Client
[código de ejemplo]",
  "type": "service_contract",
  "project": "[MEMORY_PROJECT]",
  "session_id": "[SESSION_ID]",
  "topic_key": "service-[project.name]",
  "importance": 9
}
```

### 11.4 Actualizar onboarding

```
Llamar MCP tool: mem_save
Argumentos: {
  "content": "[información del proyecto para onboarding]",
  "type": "onboarding",
  "project": "[MEMORY_PROJECT]",
  "session_id": "[SESSION_ID]",
  "topic_key": "project-onboarding",
  "importance": 10
}
```

### 11.5 Guardar estadísticas para Dashboard

```
╔══════════════════════════════════════════════════════════════════════════════╗
║  🚨 CRÍTICO: ESTE PASO ES OBLIGATORIO 🚨                                     ║
║                                                                              ║
║  SIN mem_save_stats, el skill de reportes NO FUNCIONARÁ correctamente        ║
║  y generará datos inconsistentes.                                            ║
║                                                                              ║
║  DEBES ejecutar CallMcpTool con mem_save_stats ANTES de continuar.           ║
╚══════════════════════════════════════════════════════════════════════════════╝
```

Calcular duración y guardar estadísticas consolidadas:

**IMPORTANTE:** Los stats se guardan bajo `project.name` (el proyecto actual), NO bajo `MEMORY_PROJECT` (proyecto padre). Esto permite que el comando `/dashboard` los encuentre usando el artifactId del pom.xml.

```
CallMcpTool(
  server: "user-cyborg-memory",
  toolName: "mem_save_stats",
  arguments: {
    "project": "[project.name]",   ← USAR project.name, NO MEMORY_PROJECT
    "stats": {
      "project_name": "[project.name]",
      "architecture": "[MVC o Hexagonal]",
      "features_completed": [número de features],
      "total_files_created": [número de archivos],
      "total_tests": [número de tests],
      "coverage_final": [porcentaje de coverage],
      "duration_seconds": [duración en segundos],
      "rules_applied": {
        "stack": ["10-stack", ...],
        "code_style": ["20-java-code-style", ...],
        "testing": ["41-jacoco", ...],
        "docs": ["50-api-docs-swagger", ...]
      },
      "features": [
        {
          "name": "[nombre feature]",
          "module": "[módulo]",
          "files_created": [N],
          "tests": [N],
          "rules_applied": ["regla1", "regla2"]
        }
      ],
      "last_updated": "[timestamp ISO 8601]"
    }
  }
)
```

**VERIFICAR que la llamada retorne `{"success": true}`**

Si falla, reintentar una vez. Si sigue fallando, informar al usuario.

### 11.6 Cerrar sesión

```
CallMcpTool(
  server: "user-cyborg-memory",
  toolName: "mem_session_end",
  arguments: {
    "session_id": "[SESSION_ID]"
  }
)
```

### 11.7 CHECKLIST DE VERIFICACIÓN (OBLIGATORIO)

Antes de continuar al PASO 12, verificar que se ejecutaron TODAS las llamadas MCP:

```
╔══════════════════════════════════════════════════════════════╗
║  CHECKLIST PASO 11 - MARCAR CADA UNO COMO COMPLETADO         ║
╠══════════════════════════════════════════════════════════════╣
║  [ ] 11.1 mem_session_summary → ¿Ejecutado?                  ║
║  [ ] 11.2 mem_save (decision) → ¿Ejecutado?                  ║
║  [ ] 11.3 mem_save (service_contract) → ¿Ejecutado?          ║
║  [ ] 11.4 mem_save (onboarding) → ¿Ejecutado?                ║
║  [ ] 11.5 mem_save_stats → ¿Ejecutado? ← CRÍTICO             ║
║  [ ] 11.6 mem_session_end → ¿Ejecutado?                      ║
╚══════════════════════════════════════════════════════════════╝

SI ALGUNO NO ESTÁ MARCADO → EJECUTARLO AHORA ANTES DE CONTINUAR
```

---

## PASO 12: Output Final

Aplicar regla `61-orchestrator-output.mdc` para generar:

1. Gráficos Mermaid (archivos, reglas, coverage)
2. Tablas (archivos generados, reglas aplicadas, métricas)
3. Reporte final markdown
4. Confirmación de memoria guardada

```
════════════════════════════════════════════════════════════════
💾 SESIÓN GUARDADA EN CYBORG MEMORY
════════════════════════════════════════════════════════════════

🏗️ Proyecto memoria: [MEMORY_PROJECT]
   Componente: [project.name]

📋 Resumen de la sesión:
- Features completadas: [N]
- Decisiones guardadas: [N]
- Patrones establecidos: [N]
- Coverage alcanzado: [X]%
- Contrato de servicio: Guardado ✓

🔗 Otros servicios pueden integrarse usando:
   mem_context o mem_get_service_contract

════════════════════════════════════════════════════════════════
```

---

## Subagents Disponibles

| Subagent | Función | Model |
|----------|---------|-------|
| `input-validator` | Valida archivo .input.md | fast |
| `scaffold-generator` | Crea estructura base del proyecto | fast |
| `feature-executor` | Implementa UNA feature en aislamiento | inherit |
| `feature-merger` | Combina resultados de features | fast |
| `coverage-analyzer` | Analiza reportes JaCoCo | fast |
| `test-generator` | Genera tests para mejorar coverage | inherit |
| `doc-generator` | Genera DoD, Handoff, PR | fast |
| `dashboard-generator` | Genera dashboard HTML | fast |
| `optimization-generator` | Genera config CDS/AOT/Native | fast |
| `verifier` | Valida que todo funciona | fast |

---

## Reglas Relacionadas

| Regla | Propósito |
|-------|-----------|
| `61-orchestrator-output.mdc` | Formato de output final |
| `62-validation-checklist.mdc` | Checklist de verificación |
| `63-interactive-flow.mdc` | Flujo de preguntas interactivas |
| `70-memory-cyborg.mdc` | Integración con Cyborg Memory |

---

## Flujo Completo

```
┌─────────────────────────────────────────────────────────────────┐
│                    FLUJO DEL ORQUESTADOR                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PASO 0-1: INPUT                                                │
│  ┌──────────────────────────────────────────────────┐          │
│  │  Detectar input → input-validator                 │          │
│  └──────────────────────┬───────────────────────────┘          │
│                         ▼                                       │
│  PASO 2: MEMORIA                                                │
│  ┌──────────────────────────────────────────────────┐          │
│  │  mem_context → mem_session_start                  │          │
│  └──────────────────────┬───────────────────────────┘          │
│                         ▼                                       │
│  PASO 3: SCAFFOLD (si nuevo)                                    │
│  ┌──────────────────────────────────────────────────┐          │
│  │  scaffold-generator                               │          │
│  └──────────────────────┬───────────────────────────┘          │
│                         ▼                                       │
│  PASO 4: FEATURES (paralelo)                                    │
│  ┌──────────────────────────────────────────────────┐          │
│  │  feature-executor (x N)                           │          │
│  └──────────────────────┬───────────────────────────┘          │
│                         ▼                                       │
│  PASO 5: MERGE                                                  │
│  ┌──────────────────────────────────────────────────┐          │
│  │  feature-merger                                   │          │
│  └──────────────────────┬───────────────────────────┘          │
│                         ▼                                       │
│  PASO 6: COVERAGE (si aplica)                                   │
│  ┌──────────────────────────────────────────────────┐          │
│  │  coverage-analyzer → test-generator (loop)        │          │
│  └──────────────────────┬───────────────────────────┘          │
│                         ▼                                       │
│  PASO 7-8: DOCS + DASHBOARD (paralelo)                          │
│  ┌──────────────────────────────────────────────────┐          │
│  │  doc-generator (x N) + dashboard-generator        │          │
│  └──────────────────────┬───────────────────────────┘          │
│                         ▼                                       │
│  PASO 9: OPTIMIZACIÓN (si aplica)                               │
│  ┌──────────────────────────────────────────────────┐          │
│  │  optimization-generator                           │          │
│  └──────────────────────┬───────────────────────────┘          │
│                         ▼                                       │
│  PASO 10: VERIFICACIÓN                                          │
│  ┌──────────────────────────────────────────────────┐          │
│  │  verifier                                         │          │
│  └──────────────────────┬───────────────────────────┘          │
│                         ▼                                       │
│  PASO 11-12: MEMORIA + OUTPUT                                   │
│  ┌──────────────────────────────────────────────────┐          │
│  │  mem_save (decisions, service_contract, onboarding)│         │
│  │  mem_session_end → Output final                   │          │
│  └──────────────────────────────────────────────────┘          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```
