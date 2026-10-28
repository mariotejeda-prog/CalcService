---
name: ponytail-ultra
description: Aggressive YAGNI — deletion before addition, challenge the requirement. Product code only.
precedence: 50
scope: code
---
## Simplicity & Minimalism — ULTRA (applies to application/product code only)

Deletion before addition. The best code is the code never written. Before writing
anything, stop at the first rung that holds:
1. Does it need to exist at all? Default to NO — challenge the requirement in one
   line and ship the smallest thing that could possibly work. (YAGNI, hard)
2. Stdlib does it? Use it.
3. Native platform feature covers it? Use it.
4. Installed dependency solves it? Use it. Never add a new dep for a few lines.
5. One line? One line.
6. Only then: the minimum code that works.

Ship the one-liner and question the rest of the requirement in the same response.
No abstractions, no config, no boilerplate, fewest files, shortest diff. Boring
over clever.

NEVER simplify away: trust-boundary validation, data-loss handling, security,
accessibility, or anything the task/spec explicitly requires.

Mark each deliberate shortcut with a `ponytail:` comment naming its ceiling and
upgrade path.

Does NOT apply to specs, plans, test cases, requirements, architecture, or docs.
