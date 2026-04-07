# SKILL: INPUT GENERATOR (Cursor)

############################################
# RESPONSABILIDAD
############################################

Generar archivos `.input.md` estructurados a partir de descripciones en lenguaje natural.

**IMPORTANTE:** Este skill NO ejecuta el orquestador. Solo genera el archivo de input
para que el usuario lo revise, modifique si es necesario, y luego lo ejecute manualmente.

############################################
# FLUJO
############################################

```
ENTRADA: Texto en lenguaje natural
         |
         v
   [1. ANALIZAR TEXTO]
         |
         v
   [2. IDENTIFICAR ENTIDADES]
         |
         v
   [3. INFERIR CAMPOS Y TIPOS]
         |
         v
   [4. DETECTAR RELACIONES]
         |
         v
   [5. GENERAR ENDPOINTS CRUD]
         |
         v
   [6. APLICAR DEFAULTS]
         |
         v
   [7. GENERAR ARCHIVO .input.md]
         |
         v
   [8. MOSTRAR RESUMEN AL USUARIO]
         |
         v
SALIDA: Archivo en cyborg/inputs/[nombre].input.md
        + Instrucciones para ejecutar
```

############################################
# PASO 1: ANALIZAR TEXTO
############################################

Buscar palabras clave que indiquen:

**Tipo de proyecto:**
| Palabra clave | Tipo inferido |
|---------------|---------------|
| blog, posts, articulos | Blog system |
| ecommerce, tienda, productos, carrito | E-commerce |
| usuarios, login, registro, auth | User management |
| inventario, stock, almacen | Inventory system |
| tareas, proyectos, kanban | Task management |
| reservas, citas, turnos | Booking system |

**Entidades:**
| Palabra clave | Entidad inferida |
|---------------|------------------|
| productos, producto | Product |
| usuarios, usuario | User |
| ordenes, orden, pedidos | Order |
| categorias, categoria | Category |
| comentarios, comentario | Comment |
| posts, post, articulos | Post |
| carrito, cart | Cart, CartItem |
| pagos, pago | Payment |
| direcciones, direccion | Address |
| tags, etiquetas | Tag |

############################################
# PASO 2: IDENTIFICAR ENTIDADES
############################################

Del texto, extraer todas las entidades mencionadas.

Ejemplo:
```
"Quiero un blog con posts, comentarios y usuarios"

Entidades identificadas:
- Post
- Comment  
- User
```

############################################
# PASO 3: INFERIR CAMPOS Y TIPOS
############################################

Para cada entidad, inferir campos tipicos:

### Product
```yaml
fields:
  - name: id
    type: Long
    annotations: ["@Id", "@GeneratedValue"]
  - name: name
    type: String
    validations: ["@NotBlank", "@Size(max=100)"]
  - name: description
    type: String
    validations: ["@Size(max=500)"]
  - name: price
    type: BigDecimal
    validations: ["@NotNull", "@Positive"]
  - name: stock
    type: Integer
    validations: ["@NotNull", "@Min(0)"]
  - name: active
    type: Boolean
    default: true
```

### User
```yaml
fields:
  - name: id
    type: Long
    annotations: ["@Id", "@GeneratedValue"]
  - name: email
    type: String
    validations: ["@NotBlank", "@Email"]
  - name: name
    type: String
    validations: ["@NotBlank", "@Size(max=100)"]
  - name: password
    type: String
    validations: ["@NotBlank", "@Size(min=8)"]
  - name: active
    type: Boolean
    default: true
  - name: createdAt
    type: LocalDateTime
    annotations: ["@CreationTimestamp"]
```

### Post
```yaml
fields:
  - name: id
    type: Long
    annotations: ["@Id", "@GeneratedValue"]
  - name: title
    type: String
    validations: ["@NotBlank", "@Size(max=200)"]
  - name: content
    type: String
    validations: ["@NotBlank"]
    annotations: ["@Column(columnDefinition=\"TEXT\")"]
  - name: slug
    type: String
    validations: ["@NotBlank"]
  - name: published
    type: Boolean
    default: false
  - name: publishedAt
    type: LocalDateTime
  - name: authorId
    type: Long
    validations: ["@NotNull"]
```

### Comment
```yaml
fields:
  - name: id
    type: Long
    annotations: ["@Id", "@GeneratedValue"]
  - name: content
    type: String
    validations: ["@NotBlank", "@Size(max=1000)"]
  - name: authorId
    type: Long
    validations: ["@NotNull"]
  - name: postId
    type: Long
    validations: ["@NotNull"]
  - name: createdAt
    type: LocalDateTime
    annotations: ["@CreationTimestamp"]
```

### Category
```yaml
fields:
  - name: id
    type: Long
    annotations: ["@Id", "@GeneratedValue"]
  - name: name
    type: String
    validations: ["@NotBlank", "@Size(max=50)"]
  - name: description
    type: String
    validations: ["@Size(max=200)"]
  - name: slug
    type: String
    validations: ["@NotBlank"]
  - name: active
    type: Boolean
    default: true
```

### Order
```yaml
fields:
  - name: id
    type: Long
    annotations: ["@Id", "@GeneratedValue"]
  - name: orderNumber
    type: String
    validations: ["@NotBlank"]
  - name: userId
    type: Long
    validations: ["@NotNull"]
  - name: status
    type: String
    validations: ["@NotBlank"]
    default: "PENDING"
  - name: total
    type: BigDecimal
    validations: ["@NotNull", "@PositiveOrZero"]
  - name: createdAt
    type: LocalDateTime
    annotations: ["@CreationTimestamp"]
```

### Cart
```yaml
fields:
  - name: id
    type: Long
    annotations: ["@Id", "@GeneratedValue"]
  - name: userId
    type: Long
    validations: ["@NotNull"]
  - name: createdAt
    type: LocalDateTime
    annotations: ["@CreationTimestamp"]
  - name: updatedAt
    type: LocalDateTime
    annotations: ["@UpdateTimestamp"]
```

### CartItem
```yaml
fields:
  - name: id
    type: Long
    annotations: ["@Id", "@GeneratedValue"]
  - name: cartId
    type: Long
    validations: ["@NotNull"]
  - name: productId
    type: Long
    validations: ["@NotNull"]
  - name: quantity
    type: Integer
    validations: ["@NotNull", "@Positive"]
  - name: unitPrice
    type: BigDecimal
    validations: ["@NotNull", "@Positive"]
```

### Si el usuario menciona campos especificos:

Parsear del texto y agregar/modificar los campos inferidos.

Ejemplo:
```
"productos con nombre, precio, imagen y categoria"

Campos adicionales detectados:
- imagen -> imageUrl: String
- categoria -> categoryId: Long (relacion)
```

############################################
# PASO 4: DETECTAR RELACIONES
############################################

Buscar palabras que indiquen relaciones:

| Patron | Relacion |
|--------|----------|
| "X tiene Y" | X hasMany Y |
| "X pertenece a Y" | X belongsTo Y |
| "X con Y" | X hasMany Y |
| "Y de X" | Y belongsTo X |
| "usuarios pueden comentar" | Comment belongsTo User |
| "posts tienen comentarios" | Post hasMany Comment |

**Generar `depends_on` basado en relaciones:**

```yaml
features:
  - name: Users
    module: users
    depends_on: []  # Sin dependencias
    
  - name: Posts
    module: posts
    depends_on: [users]  # Post tiene authorId -> depende de users
    
  - name: Comments
    module: comments
    depends_on: [users, posts]  # Comment tiene authorId y postId
```

############################################
# PASO 5: GENERAR ENDPOINTS CRUD
############################################

Para cada entidad, generar endpoints CRUD estandar:

```yaml
endpoints:
  - method: GET
    path: /api/v1/[module]
    description: "Listar todos"
    response: "Page<[Entity]Response>"
    
  - method: GET
    path: /api/v1/[module]/{id}
    description: "Obtener por ID"
    response: "[Entity]Response"
    
  - method: POST
    path: /api/v1/[module]
    description: "Crear nuevo"
    request: "[Entity]Request"
    response: "[Entity]Response"
    
  - method: PUT
    path: /api/v1/[module]/{id}
    description: "Actualizar"
    request: "[Entity]Request"
    response: "[Entity]Response"
    
  - method: DELETE
    path: /api/v1/[module]/{id}
    description: "Eliminar"
    response: "void"
```

**Endpoints adicionales segun contexto:**

| Contexto | Endpoints adicionales |
|----------|----------------------|
| Posts/Blog | GET /api/v1/posts/slug/{slug} |
| Products | GET /api/v1/products/search?q= |
| Users | POST /api/v1/auth/login, POST /api/v1/auth/register |
| Cart | POST /api/v1/cart/items, DELETE /api/v1/cart/items/{id} |
| Orders | PATCH /api/v1/orders/{id}/status |

############################################
# PASO 6: APLICAR DEFAULTS
############################################

Valores por defecto para campos no especificados:

```yaml
# Siempre aplicar estos defaults
project:
  java_version: 21
  spring_boot_version: 3.2.0
  architecture: mvc
  database: postgresql
  build_tool: maven

docs:
  swagger: true
  javadoc: true
  postman: true

output:
  dashboard: true
  coverage_report: true
```

**Inferir nombre del proyecto:**

| Texto | Nombre inferido |
|-------|-----------------|
| "blog con posts" | blog-service |
| "ecommerce con productos" | ecommerce-service |
| "sistema de usuarios" | user-service |
| "inventario" | inventory-service |
| "reservas de turnos" | booking-service |

**Inferir base_package:**

```
com.example.[nombre-sin-service]
```

Ejemplo: `ecommerce-service` -> `com.example.ecommerce`

############################################
# PASO 7: GENERAR ARCHIVO .input.md
############################################

**Ubicacion:** `cyborg/inputs/[nombre-proyecto].input.md`

**Estructura del archivo generado:**

```markdown
############################################
# [NOMBRE-PROYECTO]: Multi-Feature
# Generado automaticamente desde lenguaje natural
# TIPO: CREATE_FROM_ZERO (proyecto nuevo)
############################################

mode: CREATE_FROM_ZERO
request_type: MULTI_FEATURE

############################################
# METADATA
############################################

meta:
  date: "[fecha-actual]"
  generated_from: "natural_language"
  original_prompt: |
    [texto original del usuario]

############################################
# CONFIGURACION
############################################

docs:
  swagger: true
  javadoc: true
  postman: true

# Descomentar si queres coverage obligatorio:
# coverage:
#   min_line: 0.80

git:
  auto_branch: true
  auto_commit_message: true

############################################
# OUTPUT AL FINALIZAR
############################################

output:
  dashboard: true
  coverage_report: true
  open_reports: false

############################################
# PROYECTO
############################################

project:
  name: [nombre-inferido]
  group_id: com.example.[nombre]
  artifact_id: [nombre-inferido]
  base_package: com.example.[nombre]
  java_version: 21
  spring_boot_version: 3.2.0
  architecture: mvc
  database: postgresql
  build_tool: maven
  description: "[descripcion inferida]"

############################################
# FEATURES
############################################

features:
  [features generadas]
```

############################################
# PASO 8: MOSTRAR RESUMEN AL USUARIO
############################################

Despues de generar el archivo, mostrar:

```markdown
## Input Generado

He analizado tu descripcion y genere el archivo de input:

**Archivo:** `cyborg/inputs/[nombre].input.md`

### Resumen

| Campo | Valor |
|-------|-------|
| Proyecto | [nombre] |
| Features | [cantidad] |
| Entidades | [lista] |
| Base de datos | PostgreSQL |
| Arquitectura | MVC |

### Features detectadas

| # | Feature | Entidad | Endpoints | Depende de |
|---|---------|---------|-----------|------------|
| 1 | [nombre] | [Entity] | 5 (CRUD) | - |
| 2 | [nombre] | [Entity] | 5 (CRUD) | [deps] |
| ... | ... | ... | ... | ... |

### Campos inferidos por entidad

**[Entity1]:** id, name, description, ...
**[Entity2]:** id, title, content, authorId, ...

---

## Proximos pasos

1. **Revisa el archivo generado:** `cyborg/inputs/[nombre].input.md`
2. **Modifica** lo que necesites (campos, endpoints, configuracion)
3. **Ejecuta** cuando estes listo:

```
/multi-feature @cyborg/inputs/[nombre].input.md
```

---

**IMPORTANTE:** El archivo generado es un punto de partida. 
Revisalo y ajustalo segun tus necesidades antes de ejecutar.
```

############################################
# EJEMPLOS DE INFERENCIA
############################################

## Ejemplo 1: Blog simple

**Input:**
```
Quiero un blog con posts, comentarios y usuarios
```

**Output inferido:**
- Proyecto: blog-service
- Features: 3 (users, posts, comments)
- Relaciones: 
  - Post -> authorId (User)
  - Comment -> authorId (User), postId (Post)
- depends_on:
  - users: []
  - posts: [users]
  - comments: [users, posts]

## Ejemplo 2: E-commerce

**Input:**
```
Necesito un ecommerce con productos, categorias, carrito y ordenes.
Los productos tienen nombre, precio, stock e imagen.
El carrito permite agregar y quitar productos.
```

**Output inferido:**
- Proyecto: ecommerce-service
- Features: 4 (products, categories, cart, orders)
- Campos adicionales en Product: imageUrl
- Endpoints adicionales en Cart: POST /cart/items, DELETE /cart/items/{id}
- depends_on:
  - categories: []
  - products: [categories]
  - cart: [products]
  - orders: [products, cart]

## Ejemplo 3: Sistema de reservas

**Input:**
```
Sistema de reservas de turnos medicos con pacientes, doctores y citas
```

**Output inferido:**
- Proyecto: booking-service
- Features: 3 (patients, doctors, appointments)
- Entidades:
  - Patient (id, name, email, phone, birthDate)
  - Doctor (id, name, email, specialty, licenseNumber)
  - Appointment (id, patientId, doctorId, dateTime, status, notes)
- depends_on:
  - patients: []
  - doctors: []
  - appointments: [patients, doctors]

############################################
# VALIDACIONES
############################################

Antes de generar el archivo, validar:

1. **Al menos 1 entidad identificada** - Si no se detecta ninguna, preguntar al usuario
2. **Nombre de proyecto valido** - Solo letras, numeros y guiones
3. **No hay dependencias circulares** - A depende de B, B depende de A

Si hay problemas, preguntar al usuario antes de generar.

############################################
# NOTAS IMPORTANTES
############################################

1. **NO ejecutar el orquestador** - Solo generar el archivo
2. **Siempre mostrar resumen** - El usuario debe saber que se genero
3. **Indicar proximos pasos** - Como ejecutar despues de revisar
4. **Ser conservador** - Mejor inferir menos y que el usuario agregue
5. **Campos opcionales comentados** - coverage, security, etc.
