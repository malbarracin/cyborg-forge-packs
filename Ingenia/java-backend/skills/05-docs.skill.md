# SKILL: DOCS

Objetivo: Generar documentación completa y consistente del proyecto.

**⚠️ IMPORTANTE: Esta documentación debe ser IDENTICA a la que genera el orquestador.**

############################################
# README.md (en raíz del proyecto)
############################################

Debe incluir TODAS estas secciones:

## 1. Descripción del proyecto
- Nombre del proyecto
- Qué hace
- Stack tecnológico

## 2. Requisitos previos
- Java version
- Maven
- Docker (para la base de datos)

## 3. Cómo levantar la base de datos
```bash
docker-compose up -d
```
- Verificar que el contenedor está corriendo
- Puerto y credenciales

## 4. Cómo correr la aplicación
```bash
./mvnw spring-boot:run
```
- URL base: http://localhost:8080

## 5. Cómo correr tests
```bash
./mvnw test
```
- Para coverage: `./mvnw verify`

## 6. URLs importantes
- Swagger UI: http://localhost:8080/swagger-ui.html
- OpenAPI JSON: http://localhost:8080/v3/api-docs
- Health check: http://localhost:8080/actuator/health

## 7. Ejemplos curl

Por cada endpoint principal, incluir:

### Ejemplo OK:
```bash
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{
    "username": "jperez",
    "email": "jperez@example.com",
    "firstName": "Juan",
    "lastName": "Perez"
  }'
```

### Ejemplo Error 400:
```bash
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{
    "username": "",
    "email": "invalid-email"
  }'
```

### Ejemplo Error 404:
```bash
curl -X GET http://localhost:8080/api/users/999999
```

############################################
# POSTMAN COLLECTION
############################################

Archivo: `delivery/postman/[project].postman_collection.json`

Requisitos:
- 1 request OK por cada endpoint
- 1 request error (400 o 404) por módulo
- Variables: {{baseUrl}} = http://localhost:8080
- Headers: Content-Type: application/json
- Body con ejemplos REALES (no "string", no placeholders)

Estructura:
```
[Project] API
├── Users
│   ├── Create User (POST)
│   ├── Get User by ID (GET)
│   ├── List Users (GET)
│   ├── Update User (PUT)
│   ├── Delete User (DELETE)
│   └── Error - Invalid User (POST 400)
├── [Otro módulo]
│   └── ...
```

############################################
# SWAGGER/OPENAPI
############################################

## Dependencia en pom.xml:
```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

## Controllers - Anotaciones requeridas:

```java
@Tag(name = "Users", description = "Gestión de usuarios")
@RestController
@RequestMapping("/api/users")
public class UserController {

    @Operation(
        summary = "Crear usuario",
        description = "Crea un nuevo usuario en el sistema"
    )
    @ApiResponses({
        @ApiResponse(
            responseCode = "201",
            description = "Usuario creado exitosamente",
            content = @Content(
                mediaType = "application/json",
                schema = @Schema(implementation = UserResponse.class),
                examples = @ExampleObject(value = """
                    {
                        "id": "123",
                        "username": "jperez",
                        "email": "jperez@example.com"
                    }
                    """)
            )
        ),
        @ApiResponse(
            responseCode = "400",
            description = "Error de validación"
        )
    })
    @PostMapping
    public ResponseEntity<UserResponse> create(@RequestBody UserRequest request) {
        // ...
    }
}
```

## DTOs - Anotaciones requeridas:

```java
@Schema(description = "Request para crear usuario")
public record UserRequest(
    @Schema(description = "Nombre de usuario", example = "jperez", requiredMode = REQUIRED)
    @NotBlank
    String username,
    
    @Schema(description = "Email del usuario", example = "jperez@example.com", requiredMode = REQUIRED)
    @Email
    String email
) {}
```

## Checklist Swagger:

```
[ ] springdoc-openapi en pom.xml
[ ] @Tag en cada Controller
[ ] @Operation en cada endpoint
[ ] @ApiResponses con códigos 200/201, 400, 404
[ ] @ExampleObject con valores REALES
[ ] @Schema en DTOs con description y example
[ ] Swagger UI funciona en /swagger-ui.html
[ ] Ejemplos NO tienen "string", "0", "" o placeholders
```

############################################
# REGLAS
############################################

- No inventar comandos: usar los reales del repo
- No hardcodear URLs o puertos (usar los del application.yml)
- Ejemplos deben ser REALISTAS (nombres reales, emails reales, precios reales)
- Mantener consistencia entre README, Postman y Swagger

############################################
# OUTPUT
############################################

```
README.md                                         # En raíz del proyecto
delivery/postman/[project].postman_collection.json
```

############################################
# VERIFICACION FINAL
############################################

Antes de marcar como DONE:

1. Abrir http://localhost:8080/swagger-ui.html → Funciona
2. Ejecutar curl de ejemplo del README → Funciona
3. Importar Postman Collection → Todos los requests funcionan
4. Verificar que los ejemplos son REALES, no placeholders
