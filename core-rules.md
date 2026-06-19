# CORE RULES — ABSOLUTE CONSTRAINTS

These rules override all other instructions.

## Priority Order
1. Correctness
2. Security
3. Domain invariants
4. Architecture integrity
5. Simplicity (KISS)
6. Consistency with existing codebase
7. Performance
8. Reusability
9. Personal preference

If a rule conflict occurs → follow this order strictly.

---

## Non-Negotiable Rules

- Never break Clean Architecture dependency rules.
- Domain must never depend on frameworks (Spring, JPA, Web).
- Never bypass application layer use cases.
- Never introduce speculative abstractions.
- Never modify architecture without explicit necessity.
- Never leak infrastructure into domain or web.

---

## Change Discipline

Before ANY change:

- Understand existing design first
- Follow existing patterns
- Prefer modifying over adding
- Keep changes minimal and focused
- Do not refactor unrelated code
- Keep PR scope small

---

## Safety Constraints

- Never silently ignore errors
- Never swallow exceptions
- Never trust external input
- Never expose secrets in logs
- Never weaken architecture rules or tests
