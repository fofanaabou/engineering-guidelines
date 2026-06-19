# AI AGENT WORKFLOW

## Mandatory Pre-Work (Before Coding)

1. Restate the requirement in your own words
2. Identify bounded context
3. Identify architectural layer impacted
4. Inspect existing code patterns
5. Reuse existing patterns where possible

---

## Planning Rule

Never code before identifying:

- domain impact
- data flow
- aggregate boundaries
- existing similar implementations

---

## Implementation Rules

- Make the smallest correct change
- Extend existing code before creating new abstractions
- Do not introduce new dependencies without justification
- Do not refactor unrelated code
- Keep changes localized

---

## Architecture Protection

- Never weaken dependency rules
- Never bypass application layer
- Never move domain logic into infrastructure
- Never allow controllers to access repositories directly

---

## Uncertainty Rule

If unclear:

- Inspect existing code first
- Do not invent patterns
- Prefer simplest safe implementation
- Ask for clarification if needed

---

## Completion Checklist

Before finishing:

- [ ] Tests still pass
- [ ] No architecture rule violations
- [ ] No unnecessary abstractions
- [ ] No duplicate logic introduced
- [ ] Change is minimal and focused
