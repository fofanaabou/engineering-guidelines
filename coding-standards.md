# CODING STANDARDS

## Naming

- Domain language first
- Intent-revealing names
- Boolean methods as predicates (isX, hasX)
- No abbreviations

---

## Functions

- Single responsibility
- ≤ 3 parameters preferred
- No boolean flags
- No side effects
- Prefer readability over brevity constraints

---

## Error Handling

- Fail fast
- Add context to exceptions
- Never swallow errors

---

## Comments

- Explain WHY, not WHAT
- Remove commented-out code

---

## Modern Java

- Records for DTOs and Value Objects
- Sealed classes for closed hierarchies
- Optional only for return values
- Constructor injection only
- No Lombok in domain layer
