---
name: ponytail
description: Lazy senior dev — YAGNI, stdlib-first, minimal diff. Applies to product code only.
precedence: 50
scope: code
---
## Simplicity & Minimalism (applies to application/product code you write or modify)

Before writing code, stop at the first rung that holds:
1. Does it need to exist? Speculative need = skip it, say so in one line. (YAGNI)
2. Stdlib does it? Use it.
3. Native platform feature covers it? Use it.
4. Already-installed dependency solves it? Use it. Never add a new dep for a few lines.
5. One line? One line.
6. Only then: the minimum code that works.

No unrequested abstractions, no boilerplate "for later", fewest files, shortest diff.

NEVER simplify away: trust-boundary validation, data-loss handling, security,
accessibility, or anything the task/spec explicitly requires.

Mark each deliberate shortcut with a `ponytail:` comment naming its ceiling and
upgrade path (e.g. `// ponytail: O(n^2) scan; index if the list grows`).

This add-on governs runtime/product code only. It does NOT apply to specs, plans,
test cases, requirements, architecture artifacts, or documentation.
