# SKILL: ARCHITECT

Objetivo:
- Detectar arquitectura del proyecto (project.architecture)
- Definir estructura objetivo de paquetes y componentes según arquitectura
- Definir contratos de endpoints, DTOs, errors
- Definir estrategia de tests (qué se testea y cómo)
- No escribir código si aún no hay APPROVE

############################################
# DETECCIÓN DE ARQUITECTURA
############################################

Leer project.architecture del input:

## Si architecture = "mvc" (o no definido)

Aplicar rules/11-architecture-mvc.mdc

Estructura de paquetes:
```
<base_package>/
├── api/
│   ├── dto/
│   └── error/
├── entity/
├── repository/
├── service/
└── mapper/
```

Componentes a crear:
- *Controller (REST)
- *Request / *Response (DTOs)
- *Service / *ServiceImpl
- *Repository (Spring Data)
- *Entity (MongoDB)
- *Mapper (MapStruct)

## Si architecture = "hexagonal"

Aplicar rules/11-architecture-hexagonal.mdc

Estructura de paquetes:
```
<base_package>/
├── domain/
│   ├── model/
│   ├── port/in/
│   ├── port/out/
│   ├── service/
│   └── exception/
├── application/
│   └── usecase/
└── adapter/
    ├── in/rest/
    │   ├── dto/
    │   ├── mapper/
    │   └── error/
    └── out/persistence/
        ├── entity/
        ├── mapper/
        └── repository/
```

Componentes a crear:
- Domain: Model (POJO), *UseCase (port in), *Repository (port out)
- Application: *UseCaseImpl
- Adapter In: *Controller, *Request/*Response, *RestMapper
- Adapter Out: *Entity, *MongoRepository, *RepositoryAdapter, *PersistenceMapper
- Config: UseCaseConfig (beans de use cases)

############################################
# ENTREGABLE
############################################

- PLAN detallado por feature
- Lista de archivos a tocar/crear (según arquitectura detectada)
- Diagrama de dependencias entre componentes
