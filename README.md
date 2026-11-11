# Laboratorio Nro 6 – Servicios Web con GraphQL con Spring Boot

Tecnológico de Antioquia · Ingeniería de Software
Profesor: Ph.D. Diego José Luis Botia Valderrama

**Integrantes:** Daniel Salgado Mazo, Duvan Pinilla y Juan David Castro

## Descripción

Este repositorio contiene dos proyectos que exponen servicios web con **GraphQL** usando **Spring Boot**, con base de datos **MySQL**, ejecución en contenedores con **Docker** y monitoreo con **Spring Boot Actuator, Prometheus y Grafana**.

| Proyecto | Descripción |
|---|---|
| `tdeavuelo` | Práctica guiada: gestión de vuelos y reservas. |
| `tdeabanco` | Ejercicio: API de créditos para un banco en línea (otorgar créditos, consultar su estado y listarlos por categoría). |

El informe completo del laboratorio está en `Informe_Laboratorio_6_GraphQL.docx`.

## Tecnologías

- Java 25 y Spring Boot 4.0.9 (Maven)
- Spring for GraphQL, Spring Data JPA (Hibernate) y Lombok
- MySQL 8.4
- Docker y Docker Compose
- Spring Boot Actuator, Micrometer, Prometheus y Grafana

## Estructura del repositorio

```
.
├── tdeavuelo/
│   ├── src/
│   ├── grafana/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   ├── prometheus.yml
│   └── pom.xml
├── tdeabanco/
│   └── (misma estructura)
├── Informe_Laboratorio_6_GraphQL.docx
└── README.md
```

## Requisitos

- **Docker Desktop** en ejecución (en Windows, con WSL 2 activado).
- No hace falta instalar Java ni Maven: la aplicación se compila dentro del contenedor.

## Cómo ejecutar

Los dos proyectos usan los mismos puertos, por lo que **solo debe estar activo uno a la vez**.

1. Abrir una terminal en la carpeta del proyecto (`tdeavuelo` o `tdeabanco`).
2. Levantar todo:

```
docker compose up --build
```

3. Esperar a que aparezca en la consola el mensaje `Started ... Application` (la primera vez tarda varios minutos).
4. Para detener y limpiar los contenedores:

```
docker compose down
```

## Servicios disponibles

| Servicio | URL |
|---|---|
| GraphiQL (cliente para probar el API) | http://localhost:8088/graphiql |
| Endpoint GraphQL | http://localhost:8088/graphql |
| Estado de la aplicación (Actuator) | http://localhost:8088/actuator/health |
| Prometheus | http://localhost:9091/targets |
| Grafana | http://localhost:3000 (usuario `admin`, contraseña `admin`) |

En Grafana se crea automáticamente el dashboard **tdeavuelo - monitoreo** o **tdeabanco - monitoreo**, según el proyecto que esté en ejecución.

## Proyecto `tdeavuelo`

**Operaciones del API:**

| Tipo | Operación | Descripción |
|---|---|---|
| Query | `allFlights` | Lista todos los vuelos. |
| Query | `flightById(id)` | Consulta un vuelo con sus reservas. |
| Mutation | `addFlight(...)` | Crea un vuelo. |
| Mutation | `reserveFlight(flightId, passengerName, seatNumber)` | Crea una reserva y descuenta un asiento. |

**Ejemplo:**

```graphql
mutation {
  addFlight(
    flightNumber: "AV543", seatsAvailable: 180,
    origin: "BOG", destination: "MED",
    departureTime: "2026-11-20T10:00:00",
    arrivalTime: "2026-11-20T10:30:00"
  ) { id flightNumber origin destination }
}
```

## Proyecto `tdeabanco`

**Operaciones del API:**

| Tipo | Operación | Descripción |
|---|---|---|
| Query | `creditById(id)` | Consulta un crédito y su estado. |
| Query | `creditsByCategory(category)` | Lista los créditos de una categoría. |
| Query | `allCredits` | Lista todos los créditos. |
| Query | `allCustomers` | Lista los clientes con sus créditos. |
| Mutation | `createCustomer(...)` | Registra un cliente. |
| Mutation | `grantCredit(customerId, category, amount, termMonths)` | Otorga un crédito. |
| Mutation | `updateCreditStatus(creditId, status)` | Cambia el estado de un crédito. |

**Categorías y tasa de interés anual:** CONSUMO 18 %, VIVIENDA 11 %, VEHICULO 14 %, EDUCACION 9 %, LIBRE_INVERSION 20 %.

**Cuota mensual:** se calcula con la fórmula de cuota fija (sistema francés).

**Flujo de estados:** un crédito nace en `SOLICITADO` y solo puede avanzar así:

`SOLICITADO` → `APROBADO` o `RECHAZADO` · `APROBADO` → `DESEMBOLSADO` · `DESEMBOLSADO` → `PAGADO`

Cualquier otro cambio devuelve un error, por ejemplo: *No se puede pasar de SOLICITADO a PAGADO*.

**Ejemplo:**

```graphql
mutation {
  grantCredit(customerId: 1, category: CONSUMO, amount: 5000000, termMonths: 24) {
    id creditCode category monthlyInstallment status
  }
}
```

El esquema completo del API está en `src/main/resources/graphql/schema.graphqls` de cada proyecto.

## Notas

- Las credenciales de la base de datos (`root` / `root`) son solo para desarrollo.
- Los datos de MySQL se guardan en un volumen de Docker, así que se conservan entre ejecuciones mientras no se use `docker compose down -v`.
