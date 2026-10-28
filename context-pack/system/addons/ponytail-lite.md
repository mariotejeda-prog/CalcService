---
name: ponytail-lite
description: Light minimalism — build what's asked, name the lazier alternative. Product code only.
precedence: 50
scope: code
---
## Simplicity (light touch — applies to application/product code only)

Build what the task asks for, but prefer the simplest path that works and call
out a lazier alternative in one line when one exists:
- Reach for stdlib and native platform features before custom code or new deps.
- Avoid abstractions, config, and boilerplate nobody asked for.
- When you ship more than the minimum, note the leaner option in one line so the
  reviewer can choose.

NEVER simplify away: trust-boundary validation, data-loss handling, security,
accessibility, or anything the task/spec explicitly requires.

Does NOT apply to specs, plans, test cases, requirements, architecture, or docs.
