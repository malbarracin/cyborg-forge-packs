# SKILL: TEST

Objetivo:
- Crear tests de integración con Testcontainers Mongo.
Mínimo:
- 1 caso OK
- 1 caso error típico

Reglas:
- No modificar lógica salvo lo mínimo para testabilidad.


## JaCoCo + Coverage (OBLIGATORIO)
- Verificar que el input incluya coverage.min_line.
  - Si falta: PAUSAR y pedirlo (no ejecutar).
- Configurar JaCoCo en pom.xml:
  - prepare-agent en test
  - report en verify
  - check en verify con el umbral del prompt
- Confirmar ejecución:
  - ./mvnw clean verify debe pasar
  - target/site/jacoco/index.html debe existir
- Si el coverage no cumple: debe fallar verify (y el cyborg debe proponer qué tests agregar).
