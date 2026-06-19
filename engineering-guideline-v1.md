# Engineering Guidelines — Java/Spring Microservices, Clean Architecture + DDD

You are a coding agent working on a system of Java/Spring microservices, where **each service is one DDD bounded context implemented with Clean Architecture (hexagonal/ports & adapters)**. These rules govern every change you make. When a project's own conventions conflict with a rule here, follow the project's conventions and note the deviation — consistency within a codebase beats external dogma.

---

## 1. Decision Hierarchy

When design principles conflict, resolve them in this order:

1. **Correctness** — the code must work.
2. **Security** — no vulnerabilities or data leaks.
3. **Domain invariants** — business rules are inviolable.
4. **Simplicity (KISS)** — prefer the simplest solution that satisfies 1-3.
5. **Consistency with the existing codebase** — match conventions already established.
6. **Performance** — only after measuring; never for unmeasured gains.
7. **Reusability** — only when genuinely needed, not speculatively.
8. **Personal preference** — lowest priority.

Examples:
- Prefer duplication over a premature abstraction that violates domain invariants.
- Prefer consistency with the codebase over theoretical architectural purity.
- Prefer a simpler solution over a more extensible one unless extensibility is required *today*.

---

## 2. Core Principles (apply before anything else)

- **KISS** — prefer the simplest design that solves the actual problem. Simplicity is a feature.
- **YAGNI** — do not build for hypothetical future requirements. Solve today's problem; make tomorrow's easy, not pre-built.
- **DRY, but not prematurely** — duplication is cheaper than the wrong abstraction. Extract shared logic only after a real third repetition (rule of three), or when duplication already causes bugs.
- **Single Responsibility** — every function, class, and module should have one reason to change. If you can't summarize what something does in one sentence without "and," split it.
- **Explicit over implicit** — no hidden side effects, no magic. A reader should be able to predict behavior from the signature and the call site.
- **Boring is good** — prefer well-understood, battle-tested approaches over clever or novel ones unless there's a measured reason not to.
- **Optimize for the reader, not the writer** — code is read far more often than it's written. Favor clarity over brevity.

---

## 3. Clean Code

- **Naming**: intention-revealing, pronounceable, searchable. Booleans read as predicates (`isValid`, `hasPermission`). No abbreviations unless universally understood in the domain. Avoid noise words (`data`, `info`, `manager`, `helper`) unless they earn their place.
- **Functions**:
    - Small, ideally under ~20 lines; one level of abstraction per function.
    - Few parameters (≤3 ideally); group related parameters into an object/struct instead of adding a fourth.
    - No boolean flag parameters that change behavior (`process(x, true)`) — split into two named functions instead.
    - No side effects hidden behind innocuous names — a function called `getX()` must not mutate state.
- **Comments**: explain *why*, never *what*. If code needs a comment to explain what it does, rewrite the code to be self-explanatory instead. Delete commented-out code — version control remembers it.
- **Error handling**:
    - Never swallow exceptions silently. Never catch a generic exception type just to log and continue, unless that's a deliberate, documented boundary (e.g., top-level request handler).
    - Fail fast: validate inputs at the boundary, not three layers deep.
    - Use exceptions/Result types for exceptional or absent cases, not for routine control flow.
    - Error messages must contain enough context to debug without a debugger (what failed, with what input, in what operation).
- **Formatting**: consistent, automated (formatter/linter), never bikeshedded by hand. Match the project's existing style even if you'd choose differently.
- **No dead code, no speculative generality**: no unused parameters, flags, branches, or "just in case" abstractions/interfaces with a single implementation and no near-term second one.
- **Magic values**: no unexplained literals. Named constants for any number or string with meaning beyond its immediate, obvious context.

---

## 4. SOLID and Object Design

- **S — Single Responsibility**: one reason to change per class.
- **O — Open/Closed**: extend behavior via new code (new classes/strategies), not by editing stable, tested code paths. Don't over-apply this — don't add extension points nobody needs yet (see YAGNI).
- **L — Liskov Substitution**: a subtype must be usable anywhere its base type is expected, without surprising the caller (no strengthened preconditions, no weakened postconditions, no throwing on inherited contracts).
- **I — Interface Segregation**: many small, focused interfaces over one fat interface. Clients shouldn't depend on methods they don't use.
- **D — Dependency Inversion**: depend on abstractions, not concretions. High-level policy should not import low-level implementation details directly — inject them.
- **Composition over inheritance**: prefer composing small objects/behaviors over deep inheritance hierarchies. Inheritance only for genuine "is-a" relationships with stable contracts; otherwise use interfaces/strategies/delegation.
- **Law of Demeter**: talk to your immediate collaborators only. Avoid chains like `a.getB().getC().doThing()`.
- **Immutability by default**: prefer immutable objects/values where the language allows it. Mutable shared state is a default source of bugs, especially under concurrency.
- **Tell, don't ask**: prefer objects that act on their own data over external code that pulls data out and decides for them.

---

## 5. Design Patterns — use as vocabulary, not as a checklist

Apply a pattern only when it removes real complexity, never to look sophisticated. Recognize the situations they fit:

| Pattern | Use when |
|---|---|
| Strategy | Multiple interchangeable algorithms/behaviors selected at runtime |
| Factory / Abstract Factory | Object creation logic is non-trivial or must be decoupled from use |
| Builder | Construction needs many optional parameters or staged validity |
| Adapter | Reconciling an external/legacy interface with your own |
| Decorator | Adding behavior to objects without subclass explosion |
| Observer / Pub-Sub | One-to-many state-change notification, decoupled |
| Repository | Abstracting persistence away from domain logic |
| Command | Encapsulating an action as an object (undo, queuing, audit) |
| Facade | Simplifying a complex subsystem behind one clean entry point |
| CQRS | Read and write models have genuinely different shapes/scale needs |

Anti-pattern awareness — actively avoid: God classes/objects, anemic domain models (all data, no behavior, logic leaked into services), singleton-as-global-state, premature abstraction/over-engineering, copy-paste programming, shotgun surgery (one logical change requires edits in many unrelated places — a cohesion smell).

---

## 6. Clean Architecture + DDD (this project's architecture — non-negotiable)

Each microservice is one bounded context. Inside it, four layers, dependencies pointing strictly inward:

```
domain          → entities, value objects, aggregates, domain events, domain services, repository INTERFACES
application     → use cases / application services, orchestrates the domain, defines port INTERFACES for I/O
infrastructure  → adapters: JPA repositories, REST clients, Kafka producers/consumers, Spring config
interfaces/web  → controllers, DTOs, request/response mapping, exception handlers
```

Suggested package layout per service:
```
com.company.<context>
│
├── domain
│   ├── model              // entities, value objects, aggregate roots
│   ├── event              // domain events
│   ├── service            // domain services (logic that doesn't belong to one entity)
│   └── repository         // repository INTERFACES only (ports)
│
├── application
│   ├── usecase            // one class per use case (or CQRS command/query handlers)
│   ├── port.in            // input ports (use case interfaces, optional if usecase IS the port)
│   └── port.out           // output ports (interfaces: persistence, messaging, external calls)
│
├── infrastructure
│   ├── persistence        // JPA entities, Spring Data repositories, mappers domain<->JPA
│   ├── messaging          // Kafka/RabbitMQ producers & consumers
│   ├── client             // REST/gRPC clients to other services
│   └── config             // Spring @Configuration, bean wiring
│
└── interfaces.web
    ├── controller
    ├── dto                // request/response DTOs, distinct from domain model
    ├── mapper             // DTO <-> domain mapping
    └── exception          // @ControllerAdvice, error mapping
```

### The Dependency Rule (enforced, not aspirational)

- **`domain` depends on nothing.** No Spring annotations, no JPA, no Jackson, no `javax`/`jakarta` imports, no framework of any kind. Pure Java (+ a few stdlib types). This is the layer that should compile even if you ripped Spring out entirely.
- **`application` depends only on `domain`.** It defines *output ports* as plain interfaces (`LoadOrderPort`, `SaveOrderPort`, `NotifyCustomerPort`) and depends on nothing concrete. No `@Repository`, no `EntityManager`, no HTTP client types here.
- **`infrastructure` depends on `application` + `domain`**, implements the output ports (`@Repository class JpaOrderRepositoryAdapter implements SaveOrderPort`), and is where all Spring/JPA/Kafka/HTTP-client code lives.
- **`interfaces/web` depends on `application`** (calls use cases), never directly on `infrastructure` or on domain repository implementations.
- Never let a domain entity be returned directly from a controller or persisted directly via `@Entity` on the domain class itself — see "no leaking domain into JPA" below.

If you find yourself importing `org.springframework.*` or `jakarta.persistence.*` inside `domain/`, stop — that's an architecture violation, not a shortcut.

### DDD tactical patterns

- **Entities**: identity-based equality (`equals`/`hashCode` on ID only), mutable internal state but only mutated through methods that enforce invariants — never expose public setters that bypass business rules.
- **Value Objects**: immutable, equality by value, self-validating in the constructor (a `Money` or `EmailAddress` should be impossible to construct in an invalid state). Use Java `record` for these.
- **Aggregates**: one aggregate root per consistency boundary. External code only ever references/modifies the aggregate root, never an internal entity directly. Keep aggregates small — cross-aggregate consistency is eventual, not transactional.
- **Repositories**: interface lives in `domain` (or `application/port/out`), one repository per aggregate root, never a generic CRUD-everything repository. The interface speaks domain language (`findActiveOrdersForCustomer`), not persistence language (`findByStatusAndCustomerIdAndDeletedFalse`).
- **Domain Services**: only for logic that doesn't naturally belong to one entity/value object (e.g., a transfer between two accounts). If it belongs to one entity, put it on the entity.
- **Domain Events**: raise them from aggregates for anything other bounded contexts or other parts of this context may care about (`OrderPlaced`, `PaymentFailed`). Publish after the transaction commits (transactional outbox pattern), never as a side effect inside the transaction that can be rolled back inconsistently.
- **Application Services / Use Cases**: one class per use case (`PlaceOrderUseCase`), thin — fetch aggregate via port, call domain method(s), save via port, publish events. No business rules here; if you're writing an `if` that encodes a business rule in the use case, it belongs in the domain.
- **Anti-corruption layer**: when calling another bounded context (another microservice), translate its model into this context's domain language at the boundary (a dedicated adapter/translator) — never let another service's DTO shape leak into this service's domain model.
- **Ubiquitous language**: class, method, and variable names match the language the business actually uses for this bounded context. The same term can mean different things in different bounded contexts — that's expected, not a bug to "fix" by unifying names across services.

### Aggregate Rules (critical for consistency)

- **Transactions must not span multiple aggregates.** Each use case loads and modifies one aggregate (or creates a new one). If a business rule requires two aggregates to change together, it belongs in a Saga or a domain event listener, never in a transactional use case.
- **Aggregate roots enforce all invariants.** All internal consistency rules live on the root or in methods it delegates to. External code cannot bypass these rules.
- **Repositories load and save aggregate roots only.** Never load an internal aggregate entity directly; always go through the root. Never expose a collection of internal entities outside the aggregate.
- **External code must never modify internal aggregate entities directly.** Aggregates are black boxes; modification happens via method calls on the root that enforce invariants.

### Application Service (Use Case) Responsibilities

Application services contain *orchestration* logic only — they do not contain business logic.

**Allowed in application services:**
- Transaction boundaries (`@Transactional`)
- Aggregate loading via repositories
- Aggregate persistence via repositories
- Coordination between aggregates (via events or saga pattern)
- Event publication
- Logging and tracing

**Not allowed in application services:**
- Business invariants (these live in the domain)
- Validation that belongs to domain entities or value objects
- Decision-making that determines business outcomes

If you're writing `if` statements that encode business rules, stop — move that logic to the domain.

### Event Rules

**Events describe facts that happened; commands express intent.**

Good event names (past tense, fact-based):
- `OrderPlaced`
- `PaymentCaptured`
- `InventoryReserved`
- `CustomerRegistered`

Bad event names (command-like, future-oriented):
- `CreateInvoice` (this is a command, not an event)
- `SendEmail` (command)
- `ReserveInventory` (command)

**Event discipline:**
- **Immutable**: events never change once published.
- **Versioned**: include a version field; when event structure changes, support multiple versions during transition.
- **Backward compatible**: new event versions must be interpretable by old subscribers.
- **Published after transaction commits**: use the transactional outbox pattern (or event sourcing) to ensure events aren't lost if the transaction rolls back.

### Persistence Rules

- **Repositories return aggregates or domain projections,** never raw entities or ORM objects.
- **Business logic never lives in repository implementations.** A repository's job is to load and save; nothing more. No complex queries with embedded business rules.
- **Read-only queries may bypass aggregates**, but only for explicit read-only use cases (projections, reports). The rest must load through the aggregate root.
- **CQRS read models may have their own persistence structure** independent of write (aggregate) models, optimized for reads.
- **Never expose JPA entities outside infrastructure.** The persistence model is an implementation detail; always return domain aggregates or DTOs to outer layers.

### Mapping Rules

- **Map at architectural boundaries only.** Boundaries are: domain ↔ infrastructure, domain ↔ web/API, service ↔ service.
- **Avoid DTO → DTO → DTO transformation chains.** Each boundary crossing gets one mapping; don't introduce intermediate representations.
- **Do not create mapper layers without a boundary crossing.** Mappers exist to translate between different models at specific architectural layers, not as a general pattern.
- **Prefer explicit mappings over reflection-based magic.** Hand-written mappers or MapStruct with explicit `@Mapping` are preferable to generic/convention-based magic; future maintainers will understand them better.
- **Domain models are never used as API contracts.** The API (REST, gRPC, events) uses its own DTOs; clients never depend on your domain model structure.

### Interface Rules

Do not create interfaces solely for future extensibility (YAGNI).

**Create interfaces only when:**
- They represent a Clean Architecture port (input or output).
- They're a public contract (API, event, message schema).
- They're a framework integration boundary (Spring `@Service`, `@Repository`, etc.).
- There are already multiple concrete implementations, or one is imminent.

### Spring-specific discipline

- `domain` and `application` use *constructor injection only*, and ideally don't even need `@Component`/`@Service` if you wire beans explicitly in `infrastructure/config` — at minimum, no field injection (`@Autowired` on a field) anywhere in the codebase.
- **No leaking domain into JPA**: don't annotate domain entities with `@Entity`. Keep a separate JPA entity in `infrastructure/persistence` and map explicitly (MapStruct or hand-written mapper) between JPA entity and domain aggregate. This is what keeps `domain` framework-free and lets the persistence model evolve independently of business rules.
- **DTOs at the edges only**: controllers and messaging consumers/producers use DTOs, mapped explicitly to/from domain objects. Never serialize a domain entity straight to JSON, and never bind a domain object directly to a JPA/Jackson annotation.
- Use cases are the unit that's `@Transactional` — not the controller, not the repository.
- Spring profiles/config (`application.yml`) belong to `infrastructure`; the domain must never read config/environment directly.

### Microservices / bounded-context boundaries

- **One service = one bounded context = one database.** No other service ever queries this service's database directly.
- **API-first contracts**: define and version the contract (OpenAPI for sync REST, schema for async events) before implementation. Treat contract changes as breaking unless proven additive/backward-compatible.
- **Decentralized data, eventual consistency across contexts**: avoid distributed transactions/2PC. Use the Saga pattern (choreography via domain/integration events, or orchestration via a coordinating use case) for cross-context workflows; design explicit compensating actions.
- **Design for failure, always**:
    - Timeouts on every network call (REST client, DB, broker), no exceptions.
    - Retries with exponential backoff and jitter, capped, only for idempotent operations.
    - Circuit breakers (Resilience4j) to stop cascading failure.
    - Bulkheads/isolation so one downstream's failure doesn't exhaust shared thread pools/connections.
    - Idempotency keys for any operation that might be retried (payments, writes triggered by event redelivery).
- **Observability is not optional**: structured logging with correlation/trace IDs (propagated via headers/event metadata across service calls), metrics, distributed tracing (e.g., via Micrometer/OpenTelemetry). You cannot operate what you cannot see.
- **Events over commands between contexts** where workflows allow it, to reduce temporal coupling — publish `OrderPlaced`, don't call `ReserveInventory` synchronously unless the workflow genuinely requires an immediate answer.
- **No shared mutable libraries that encode business logic** across services — a shared "common" JAR with domain logic recreates the monolith with extra network hops. Shared code across services should be thin technical utilities only (logging config, tracing setup), never domain types.
- **Backward-compatible API/event evolution**: additive changes only on a given version; breaking changes get a new version with a documented migration/deprecation path and, for events, dual-publish during transition.
- **Each service independently deployable and independently testable**: contract tests against consumers (e.g., Pact) instead of relying on slow shared end-to-end environments.

---

## 7. Architecture Fitness Functions (automated enforcement, not guidelines)

These rules are enforced by the build via ArchUnit. A test failure here is a build failure, not a warning.

The build must fail when:

- Domain depends on Spring (`org.springframework.*`).
- Domain depends on JPA (`jakarta.persistence.*`).
- Domain depends on Web/HTTP (`jakarta.servlet.*`, `org.springframework.web.*`).
- Application depends on Infrastructure or Web implementations.
- Infrastructure is directly referenced by Interfaces/Web layer (only via application layer).
- Controllers access repositories directly (must go through use cases).
- Use cases are bypassed (business logic entry point must be a use case, not a controller calling a repository).
- Infrastructure modules reference each other (each is isolated; communication through application/domain only).

```java
// Example ArchUnit rule enforcement in a test:
ArchRuleDefinition.noClasses()
  .that().resideInAPackage("..domain..")
  .should().dependOnClassesThat().resideInAPackage("org.springframework..")
  .check(classes);
```

---

## 8. Complexity Management

Before introducing any of the following, ask yourself:

- Inheritance hierarchies (deep class trees)
- Generic types and type bounds
- Reflection or annotation-based configuration
- Design patterns or frameworks
- Asynchronous workflows
- Caching strategies
- Message queues or event buses

**Ask**: "Is this solving an *existing* problem in the codebase today, or a *hypothetical future* problem?"

**If the answer is hypothetical or "someday we might need this," do not implement it.** (This is YAGNI.)

Apply the Decision Hierarchy: simplicity (KISS) ranks higher than reusability or extensibility.

---

## 9. Testing

- **Test pyramid**: many fast unit tests, fewer integration tests, very few end-to-end tests. Don't invert it.
- **Unit tests on `domain` and `application`**: pure JUnit, no Spring context, no Mockito needed for domain (it has no dependencies to mock); mock only the output ports in `application` tests. These should run in milliseconds with zero Spring startup cost.
- **Integration tests on `infrastructure`**: `@DataJpaTest`/Testcontainers for persistence adapters, WireMock or Testcontainers for external HTTP/messaging adapters — verify the adapter actually satisfies the port contract against something real.
- **Slice/contract tests on `interfaces/web`**: `@WebMvcTest` for controllers (mock the use case), full contract tests (e.g. Pact) for the public API consumed by other services.
- **Architecture tests**: enforce the dependency rule with ArchUnit (e.g., "no classes in `..domain..` depend on `..springframework..`"). Treat an ArchUnit failure as a build failure, not a warning — this is what keeps the architecture honest as the team grows.
- **Arrange-Act-Assert** structure, one logical assertion focus per test, descriptive test names that state the scenario and expected outcome (`shouldRejectTransferWhenBalanceInsufficient`, not `test3`).
- **Test behavior, not implementation.** Tests should survive refactors that don't change observable behavior. If renaming a private method breaks a test, the test is too coupled.
- **TDD where it fits** (red-green-refactor) — especially valuable for tricky domain logic and bug fixes (write the failing test that reproduces the bug first).
- **Edge cases as a default checklist**: empty/null inputs, boundary values, concurrency races, failure of external dependencies, and malformed input — not just the happy path.
- **Tests are production code**: keep them readable, DRY where it helps clarity, reviewed with the same rigor.

---

## 10. Performance

- **Measure before optimizing.** Profile (JFR, async-profiler) to find the actual bottleneck; never guess. Premature optimization that sacrifices readability for unmeasured gains is a defect, not an improvement.
- **Algorithmic complexity first.** Fix an O(n²) hot path before micro-tuning O(n) code — Big-O dominates at scale far more than constant-factor tricks.
- **N+1 queries**: the most common Spring/JPA performance bug. Use fetch joins, `@EntityGraph`, or explicit batch fetching deliberately — never rely on lazy-loading by default in a hot path.
- **Know your I/O cost**: minimize network round-trips between services (batch where the contract allows it), cache deliberately with explicit invalidation strategy, prefer streaming/pagination over loading everything into memory for large datasets.
- **Concurrency**: prefer immutable data (records, value objects) and message-passing over shared mutable state with locks. Java 21+ virtual threads are the default choice for I/O-bound concurrency over manual thread-pool tuning. When locks are necessary, keep critical sections minimal and document the invariant being protected.
- **Database performance**: index what you query/filter/sort/join on, but don't over-index write-heavy tables blindly. Avoid `SELECT *` / fetching full entities when a projection suffices. Paginate large result sets.
- **Set a budget, not a vibe**: define target latency/throughput/memory before optimizing, stop once you hit it. Endless optimization is its own form of waste.

---

## 11. Maintainability & Evolution

- **Code for deletion as much as for reuse.** A feature that's easy to remove cleanly is healthier than one wedged into everything via shared state.
- **Backward compatibility and migrations are first-class work**, not an afterthought — plan deprecation windows, feature flags, and rollback paths for any change that affects external consumers or running data.
- **Feature flags / toggles** for risky or incremental changes, with an explicit plan to remove the flag once the change is fully rolled out — flags that live forever are technical debt.
- **Documentation that earns its keep**: README answers "how do I run this and why does it exist," ADRs (Architecture Decision Records) capture *why* a non-obvious decision was made, not just what. Inline docs only where the *why* isn't obvious from the code itself.
- **Static analysis, linting, and formatting as automated gates**, not manual review burden — a human reviewer should never be the one catching a missing semicolon or an unused import.
- **Refactor continuously, in small steps, under test coverage** — the Boy Scout Rule (leave code slightly better than you found it), not as a separate "someday" rewrite project. Large rewrites are high-risk; prefer the strangler fig pattern to incrementally replace legacy systems behind a stable interface.
- **Version control hygiene**: small, focused commits with meaningful messages; a PR does one logical thing and is small enough to actually review.
- **Track and pay down technical debt deliberately** — make it visible (tickets, comments with context), don't let it accumulate silently.

---

## 12. Security (baseline, always)

- **Never trust input.** Bean Validation (`@Valid`, `@NotNull`, etc.) at the DTO/controller boundary, plus domain-level self-validation in value object constructors — two layers, not one.
- **No secrets in code or version control.** Use Spring Cloud Config / Vault / environment injection, never hardcoded values or plain `application.yml` secrets committed to the repo.
- **Least privilege** for every datasource credential, service account, and API key — each service's DB user should only have rights to that service's schema.
- **Parameterized queries always** — Spring Data/JPQL parameter binding, never string-concatenated JPQL/native SQL.
- **Encode output for its context** (HTML, SQL, shell, URL) to prevent injection.
- **Keep dependencies patched**; treat a known CVE in a Spring Boot/transitive dependency as a bug, not background noise — check with `dependency-check`/`mvn versions:display-dependency-updates` periodically.
- **Log security-relevant events, never log secrets, tokens, or full PII** — watch for this especially in request/response logging filters and exception messages.

---

## 13. Modern Java Practices

- **Records for value objects and DTOs.** Any immutable data carrier — value objects, request/response DTOs, internal data transfer — should be a `record`, not a hand-rolled class with getters/setters/equals/hashCode.
- **Sealed interfaces/classes** for closed, known hierarchies (e.g., a `PaymentResult` that's exactly `Success | Declined | Failed`) — lets the compiler enforce exhaustiveness via pattern matching.
- **Pattern matching for `switch`/`instanceof`** over chains of `if/else instanceof` casts, especially when working with sealed result types.
- **`Optional` for genuinely absent return values** at API boundaries — never as a field type, never as a method parameter, never just to avoid a null check you should be doing explicitly.
- **Virtual threads (Java 21+)** as the default for I/O-bound concurrent work (calling other services, blocking JDBC) instead of manually sized thread pools — but never use them for CPU-bound work, and never pair them with `synchronized` blocks that pin the carrier thread (use `ReentrantLock` instead).
- **Constructor injection only**, fields `private final`. No field injection, no setter injection.
- **No Lombok in `domain`.** If used at all, confine it to DTOs/infrastructure — domain value objects/entities should be plain, explicit Java (or records) so invariants in constructors are visible, not generated.
- **Streams for transformation pipelines, not for everything.** A `for` loop with early return/side effects is often clearer than a stream forced to do the same job — don't reach for `.stream()` reflexively.
- **Keep Spring Boot and core dependencies current**; don't pin to an old Boot/Java version without a documented reason — security patches and language features both depend on staying current.

---

- **Keep Spring Boot and core dependencies current**; don't pin to an old Boot/Java version without a documented reason — security patches and language features both depend on staying current.

---

## 14. Change Discipline

When implementing a change, follow this order:

1. **First understand the existing design** before introducing a new one. Read surrounding code and understand the patterns already present.
2. **Modify existing code before creating new abstractions.** Prefer extending existing components over building new frameworks.
3. **Reuse existing patterns** already present in the bounded context — don't invent new ones.
4. **Do not rename, move, or reorganize code** unless required by the task at hand.
5. **Do not perform drive-by refactors** while implementing a feature. Keep changes focused.
6. **Keep PRs and changes focused on one business change.** One logical feature = one PR. Mixing refactors with features makes review harder and introduces risk.

Additional discipline:
- Do not introduce new external dependencies without explicit justification — discuss with the team first.
- Do not add abstractions (interfaces, base classes, generics) with only one implementation unless that abstraction represents a Clean Architecture port or boundary.

---

## 15. Operating Rules for This Agent

**Before writing any code:**

1. Identify the requirement clearly in one sentence — if you can't, ask or investigate further before coding.
2. Identify which bounded context the change belongs to.
3. Identify which architectural layer the code belongs in (domain / application / infrastructure / interfaces/web).
4. Locate any existing patterns or similar code in that layer — this is your template.
5. Reuse existing patterns when appropriate; do not invent new patterns unless necessary.

**When making changes:**

- Read surrounding code first to understand conventions.
- Match existing conventions exactly — consistency with the codebase ranks higher than personal preference.
- Prefer extending existing code over creating new frameworks or abstractions.
- Do not introduce new dependencies without justification — discuss with the team if unsure.
- Do not add abstractions with only one implementation unless they represent a Clean Architecture port or boundary.
- **Before adding code, decide its layer first** and place it there — don't default to "wherever the existing mess is" or to infrastructure/controller because it's the path of least resistance.

**Conflict resolution:**

- If a rule in this document conflicts with an explicit user instruction, follow the user's instruction but note the tradeoff briefly.
- If a rule conflicts with the existing codebase's established convention, follow the codebase's convention and don't introduce inconsistency — but flag if the convention violates the dependency rule (e.g., domain importing Spring), since that's worth raising even if widespread.

**Final gate:**

- Always leave tests passing, including ArchUnit architecture tests. Never disable or weaken a test to make it pass artificially.
- When uncertain between two valid approaches, choose the one a future maintainer with no context could understand fastest.