# ARCHITECTURE — CLEAN ARCH + DDD

## Dependency Rule

domain → application → infrastructure  
presentation or web → application  

Never reverse dependencies.

---

## Domain Layer

- Pure Java only
- No Spring, JPA, or framework imports
- Enforces all business invariants
- Entities and Value Objects are immutable where possible
- Domain events represent business facts

---

## Aggregates

- One aggregate root per consistency boundary
- Only aggregate root enforces invariants
- No cross-aggregate transactions
- Repositories only load/save aggregates

---

## Application Layer

Responsible for orchestration:

Allowed:
- Transactions
- Use case coordination
- Repository calls
- Event publishing

Forbidden:
- Business rules
- Domain logic decisions

---

## Infrastructure Layer

Contains:
- JPA implementations
- Messaging (Kafka, RabbitMQ)
- External API clients
- Configurations

Must NEVER leak into domain/application.

---

## Web Layer

Contains ONLY:
- Controllers
- DTOs
- Mappers
- Exception handlers

No business logic allowed.
