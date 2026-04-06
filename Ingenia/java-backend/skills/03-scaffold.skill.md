# SKILL: SCAFFOLD

## Objetivo (CREATE_FROM_ZERO)

Crear el proyecto base completo:
- Maven wrapper (si aplica)
- pom.xml (con configuracion segun optimization)
- application.yml
- docker-compose.yml (segun database del stack)
- main class
- estructura paquetes (segun architecture: mvc o hexagonal)
- README inicial
- Archivos de optimizacion (si aplica)

## Objetivo (CHANGE_EXISTING)

Solo tocar scaffold si el cambio lo requiere.

## Parametros de entrada

El skill recibe del orquestador:
- PROJECT_NAME: nombre del proyecto
- BASE_PACKAGE: paquete base Java
- GROUP_ID: group id Maven
- ARTIFACT_ID: artifact id Maven
- JAVA_VERSION: version de Java (default: 21)
- SPRING_VERSION: version Spring Boot (default: 3.3.x)
- DATABASE: tipo de base de datos (postgresql, mysql, mongodb, h2)
- BUILD_TOOL: maven o gradle
- ARCHITECTURE: mvc o hexagonal
- OPTIMIZATION: none, cds, aot, native (default: none)

## Estructura de paquetes

### Si ARCHITECTURE = mvc

```
src/main/java/[BASE_PACKAGE]/
├── [PROJECT]Application.java
├── config/
├── controller/
├── service/
├── repository/
├── entity/
├── dto/
│   ├── request/
│   └── response/
├── exception/
└── mapper/
```

### Si ARCHITECTURE = hexagonal

```
src/main/java/[BASE_PACKAGE]/
├── [PROJECT]Application.java
├── domain/
│   ├── model/
│   ├── port/
│   │   ├── in/
│   │   └── out/
│   └── service/
├── application/
│   ├── usecase/
│   └── dto/
└── adapter/
    ├── in/
    │   └── web/
    └── out/
        └── persistence/
```

## Configuracion de pom.xml segun OPTIMIZATION

### Si OPTIMIZATION = none

pom.xml estandar sin configuracion adicional.

### Si OPTIMIZATION = cds

pom.xml estandar. CDS no requiere cambios en pom.xml.
Generar scripts:
- `generate-cds.sh`
- `generate-cds.bat`

### Si OPTIMIZATION = aot

Agregar a pom.xml en spring-boot-maven-plugin:

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <executions>
        <execution>
            <id>process-aot</id>
            <goals>
                <goal>process-aot</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

Generar script:
- `run-aot.sh`
- `run-aot.bat`

### Si OPTIMIZATION = native

Agregar a pom.xml:

1. Plugin native-maven-plugin:
```xml
<plugin>
    <groupId>org.graalvm.buildtools</groupId>
    <artifactId>native-maven-plugin</artifactId>
    <configuration>
        <buildArgs>
            <buildArg>--no-fallback</buildArg>
            <buildArg>-H:+ReportExceptionStackTraces</buildArg>
        </buildArgs>
    </configuration>
</plugin>
```

2. Profile native:
```xml
<profiles>
    <profile>
        <id>native</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.graalvm.buildtools</groupId>
                    <artifactId>native-maven-plugin</artifactId>
                    <executions>
                        <execution>
                            <id>build-native</id>
                            <goals>
                                <goal>compile-no-fork</goal>
                            </goals>
                            <phase>package</phase>
                        </execution>
                    </executions>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
```

3. Generar archivos:
- `src/main/resources/META-INF/native-image/reflect-config.json`
- Clase `NativeHints.java` en config/

## Archivos de optimizacion a generar

### generate-cds.sh (para OPTIMIZATION = cds)

```bash
#!/bin/bash
set -e

JAR_FILE=$(ls target/*.jar 2>/dev/null | head -1)

if [ -z "$JAR_FILE" ]; then
    echo "Error: No se encontro JAR. Ejecuta './mvnw clean package' primero."
    exit 1
fi

echo "=== Generando CDS para $JAR_FILE ==="

rm -f classes.lst app.jsa

echo "Paso 1/3: Generando lista de clases..."
timeout 30 java -Xshare:off -XX:DumpLoadedClassList=classes.lst \
    -Dspring.main.lazy-initialization=true \
    -jar "$JAR_FILE" || true

echo "Paso 2/3: Creando archivo CDS..."
java -Xshare:dump \
    -XX:SharedClassListFile=classes.lst \
    -XX:SharedArchiveFile=app.jsa \
    -cp "$JAR_FILE" || true

if [ -f "app.jsa" ]; then
    echo "Paso 3/3: CDS generado exitosamente!"
    echo ""
    echo "Para ejecutar con CDS:"
    echo "  java -Xshare:on -XX:SharedArchiveFile=app.jsa -jar $JAR_FILE"
else
    echo "Error: No se pudo generar el archivo CDS"
    exit 1
fi
```

### run-aot.sh (para OPTIMIZATION = aot)

```bash
#!/bin/bash
JAR_FILE=$(ls target/*.jar 2>/dev/null | head -1)

if [ -z "$JAR_FILE" ]; then
    echo "Error: No se encontro JAR. Ejecuta './mvnw clean package' primero."
    exit 1
fi

echo "Ejecutando con AOT habilitado: $JAR_FILE"
java -Dspring.aot.enabled=true \
     -XX:TieredStopAtLevel=1 \
     -Xms256m -Xmx512m \
     -jar "$JAR_FILE"
```

### reflect-config.json (para OPTIMIZATION = native)

Generar con todas las clases DTO y Entity del proyecto:

```json
[
  {
    "name": "[BASE_PACKAGE].dto.request.[Entity]Request",
    "allDeclaredConstructors": true,
    "allDeclaredMethods": true,
    "allDeclaredFields": true
  },
  {
    "name": "[BASE_PACKAGE].dto.response.[Entity]Response",
    "allDeclaredConstructors": true,
    "allDeclaredMethods": true,
    "allDeclaredFields": true
  },
  {
    "name": "[BASE_PACKAGE].entity.[Entity]",
    "allDeclaredConstructors": true,
    "allDeclaredMethods": true,
    "allDeclaredFields": true
  }
]
```

### NativeHints.java (para OPTIMIZATION = native)

```java
package [BASE_PACKAGE].config;

import org.springframework.aot.hint.MemberCategory;
import org.springframework.aot.hint.RuntimeHints;
import org.springframework.aot.hint.RuntimeHintsRegistrar;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.ImportRuntimeHints;

@Configuration
@ImportRuntimeHints(NativeHints.NativeHintsRegistrar.class)
public class NativeHints {

    public static class NativeHintsRegistrar implements RuntimeHintsRegistrar {
        @Override
        public void registerHints(RuntimeHints hints, ClassLoader classLoader) {
            // Registrar DTOs para Jackson
            // [GENERAR PARA CADA DTO]
            
            // Registrar recursos
            hints.resources()
                .registerPattern("application*.yml")
                .registerPattern("META-INF/spring/*");
        }
    }
}
```

## Actualizacion de .gitignore

Agregar segun OPTIMIZATION:

### Para cds:
```gitignore
# CDS files
classes.lst
*.jsa
```

### Para native:
```gitignore
# Native build
target/native-image/
```

## Dockerfile

Segun OPTIMIZATION, usar el template correspondiente:
- none: Dockerfile estandar
- cds: `.cursor/templates/Dockerfile.cds.template`
- aot: `.cursor/templates/Dockerfile.aot.template`
- native: `.cursor/templates/Dockerfile.native.template`

Reemplazar placeholders:
- `{{PROJECT_NAME}}` → PROJECT_NAME
- `{{JAVA_VERSION}}` → JAVA_VERSION (solo numero: 17, 21)
- `{{PORT}}` → 8080 (o el configurado)
- `{{ARTIFACT_ID}}` → ARTIFACT_ID
