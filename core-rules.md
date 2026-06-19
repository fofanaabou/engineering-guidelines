# Engineering Guidelines — Java/Spring Microservices, Clean Architecture + DDD (Enhanced Edition)

## Decision Hierarchy

When rules conflict, apply them in this order:

1. Correctness
2. Security
3. Domain invariants
4. Simplicity (KISS)
5. Consistency with the existing codebase
6. Performance
7. Reusability
8. Personal preference

Examples:
- Prefer duplication over a premature abstraction.
- Prefer consistency with the codebase over theoretical purity.
- Prefer a simpler solution over a more extensible one unless extensibility is required today.

---

## Core Principles

- KISS — prefer the simplest design that solves the actual problem.
- YAGNI — build for today's requirements, not hypothetical futures.
- DRY, but not prematurely (Rule of Three).
- Single Responsibility.
- Explicit over implicit.
- Boring is good.
- Optimize for the reader, not the writer.

---

## Change Discipline (Mandatory)

Before making any change:

- Understand the existing design before introducing a new one.
- Follow existing patterns within the bounded context.
- Prefer modifying existing code over creating new abstractions.
- Reuse existing conventions whenever possible.
- Do not perform drive-by refactors.
- Do not rename, move, or reorganize code unless required.
- Keep changes focused on one business concern.
- Keep PRs small and reviewable.

---

## Clean Code

### Naming
- Intention revealing.
- Domain language first.
- Booleans read as predicates.
- Avoid noise words.

### Functions
- One responsibility.
- Prefer ≤ 3 parameters.
- No boolean flags.
- No hidden side effects.
- Functions should fit comfortably on one screen.
- Prefer readability over arbitrary line limits.

### Comments
- Explain WHY, not WHAT.
- Delete commented-out code.

### Error Handling
- Fail fast.
- Never swallow exceptions.
- Include debugging context.

### General
- No dead code.
- No speculative abstractions.
- No magic values.

---

## Complexity Management

Before introducing:

- Inheritance
- Generics
- Reflection
- Custom annotations
- Design patterns
- Async workflows

Ask:

"Is this solving a current problem or a hypothetical future problem?"

If hypothetical, do not implement it.

---

## SOLID and Object Design

- SRP
- OCP (without speculative extension points)
- LSP
- ISP
- DIP

Additional rules:

- Composition over inheritance.
- Law of Demeter.
- Immutability by default.
- Tell, don't ask.

### Interfaces

Do not create interfaces solely for future extensibility.

Allowed exceptions:

- Clean Architecture ports
- Public contracts
- Framework boundaries
- Existing multiple implementations

---

## Clean Architecture + DDD

### Dependency Rule

domain → application → infrastructure

interfaces/web → application

Never invert dependencies.

### Architecture Fitness Functions

Build must fail when:

- Domain depends on Spring.
- Domain depends on JPA.
- Domain depends on Web.
- Controllers access repositories directly.
- Infrastructure leaks into web.
- Use cases are bypassed.

Enforce via ArchUnit.

### Domain

- Pure Java.
- No framework dependencies.
- Entities enforce invariants.
- Value Objects are immutable records.
- Domain events represent business facts.

### Aggregate Rules

- One aggregate root per consistency boundary.
- Aggregate roots enforce all invariants.
- Transactions must not span multiple aggregates.
- Repositories load/save aggregate roots only.
- Internal entities cannot be modified externally.

### Application Layer

Application services contain orchestration logic.

Allowed:

- Transactions
- Loading aggregates
- Persisting aggregates
- Coordinating aggregates
- Publishing events

Not allowed:

- Business invariants
- Business decisions
- Domain validation

### Infrastructure

Contains:

- Persistence adapters
- Messaging adapters
- REST/gRPC clients
- Configuration

### Web Layer

Contains:

- Controllers
- DTOs
- Mappers
- Exception handling

---

## Mapping Rules

- Map only at architectural boundaries.
- Avoid DTO → DTO → DTO chains.
- Prefer explicit mappings.
- Domain models are never API contracts.
- JPA entities never leave infrastructure.

---

## Persistence Rules

- Repositories speak domain language.
- Business logic never belongs in repositories.
- Read models may bypass aggregates for CQRS.
- Avoid exposing persistence models outside infrastructure.

---

## Event Rules

Events describe facts.

Good:
- OrderPlaced
- PaymentCaptured
- CustomerRegistered

Bad:
- CreateInvoice
- SendEmail
- ReserveInventory

Rules:

- Commands express intent.
- Events express facts.
- Events are immutable.
- Events are versioned.
- Events remain backward compatible.

---

## Microservices Rules

- One bounded context per service.
- One database per service.
- No direct DB access between services.
- API-first contracts.
- Eventual consistency.
- Saga for distributed workflows.
- Design for failure.
- Observability is mandatory.
- Prefer events over commands across contexts.
- No shared business-logic libraries.

---

## Testing

### Test Pyramid

- Many unit tests.
- Fewer integration tests.
- Very few E2E tests.

### Architecture Tests

ArchUnit failures are build failures.

### General

- AAA structure.
- Test behavior, not implementation.
- Edge cases are mandatory.
- Tests are production code.

---

## Performance

- Measure before optimizing.
- Algorithmic complexity first.
- Prevent N+1 queries.
- Minimize I/O.
- Prefer immutable data.
- Use virtual threads for I/O-bound workloads.
- Define performance budgets before optimization.

---

## Security

- Never trust input.
- Validation at boundaries and domain.
- No secrets in code.
- Least privilege.
- Parameterized queries.
- No sensitive data in logs.
- Keep dependencies patched.

---

## Modern Java

- Records for DTOs and Value Objects.
- Sealed hierarchies for closed models.
- Pattern matching for switch.
- Optional only as return values.
- Constructor injection only.
- No Lombok in domain.
- Streams when they improve clarity.

---

## AI Agent Operating Rules

Before coding:

1. Restate the requirement.
2. Identify the bounded context.
3. Identify the architectural layer.
4. Locate existing patterns.
5. Reuse existing patterns where appropriate.

When coding:

- Read surrounding code first.
- Match existing conventions.
- Prefer extending existing code.
- Do not introduce dependencies without justification.
- Do not introduce abstractions with a single implementation unless they represent an architectural boundary.
- Prefer the smallest correct change.
- Leave all tests passing.
- Never weaken architecture tests.
- When uncertain, choose the solution a future maintainer can understand fastest.
