> **Execution Protocol section file — §8 Deviation Taxonomy.** Routing + universal sections (§2, §10.5, §11) live in [`execution-protocol.md`](../execution-protocol.md). § numbering is preserved.

## 8. Deviation Taxonomy

When implementation encounters something not in the plan, classify and act:

### Auto-Fix (proceed without stopping)
- **D-AUTO-1: Bug in generated code** — syntax error, runtime error, failing test caused by current task
- **D-AUTO-2: Missing critical safety** — null check, error handling, input validation, auth guard omitted
- **D-AUTO-3: Broken dependency** — missing import, unresolved reference, config error blocking build

Rules: Max 3 auto-fixes per file. Document each in IMPL-STATE deviations table.
If 4th auto-fix needed on the same file → escalate.

### Escalate (stop, document, request guidance)
- **D-ESC-1: Architectural change** — new database table, new service, schema migration, library substitution
- **D-ESC-2: Scope expansion** — feature not in plan, new API endpoint, new UI component
- **D-ESC-3: Spec contradiction** — plan says X, research says Y, code needs Z

Action: Write blocker to IMPL-STATE, set phase status = BLOCKED, include alternatives
with trade-offs. Do NOT proceed without guidance.

### Defer (note for future, continue current work)
- **D-DEF-1: Pre-existing issue** — bug/smell in untouched code discovered during implementation
- **D-DEF-2: Optimization opportunity** — performance improvement not in scope
- **D-DEF-3: Style/convention mismatch** — naming inconsistency in existing code

Action: Add to IMPL-STATE deviations with category = DEFERRED. Do NOT fix.

---

