> **Execution Protocol section file — §13 Code-Location Discipline.** Routing + universal sections (§2, §10.5, §11) live in [`execution-protocol.md`](../execution-protocol.md). § numbering is preserved.

## 13. Code-Location Discipline

The context pack ships a `codebase-map.md` (directory layout, bounded-context ownership, key entry points, file responsibilities) **precisely so agents do not rediscover the repository structure with a full-tree scan on every run.** Observed failure: across a full code-development session — research, planning, implementation, code-review, and two excursions — `codebase-map.md` was **never opened**; it sat in the context-pack index as a one-line summary while agents located code by scanning the tree directly. This section makes the map a pre-scan step for code-searching skills.

### 13.1 The rule — consult before scan, not never scan

**Before running a repository-wide *discovery* scan to find where code lives** — a `grep`/`glob`/`find`/`ls -R` or directory walk whose purpose is "where is X?", "which files implement Y?", "what's the layout of Z?" — **first read `context-pack/codebase-map.md` (and `project-inventory.md` if present) and navigate from it.**

This is **"consult before scan," not "never scan."** The distinction is exact:

- **Forbidden first move:** an *exploratory* tree scan to discover layout/ownership/where-things-are, when the map already answers it. That is the scan the map exists to replace.
- **Always allowed:** **targeted reads** of the specific files the map points you to (open them, read them fully — that is the whole point of locating them). And a **fallback scan** when the map is **absent, or genuinely lacks the answer** (a file/area the map doesn't cover, or a question below the map's granularity).
- The map **replaces discovery scans, not targeted reads.** Finding the file via the map and then reading it is correct; grepping the whole tree to find the file the map already names is the waste.

### 13.2 When the map is insufficient — flag the gap

If `codebase-map.md` is missing the area you need, or is stale/contradicted by what you find, you **may** fall back to a scoped scan — and you **must record the gap** so the map can be corrected: name the path/area the map failed to cover in your output (the skill's open-questions / notes / findings, as appropriate). Silent fallback lets the map rot; a flagged gap is how it stays the cheap orientation surface. Never trust a stale map over ground truth — when the map and the code disagree, the code wins, and the disagreement is a flag.

### 13.3 Applies to (code-searching skills only)

This is **not** a global pre-flight read — it is targeted at skills that locate code. It does **not** apply to ideation, refinement, or other skills that never search a source tree.

Cited by: `researching-code-design`, `researching-feature-impl`, `researching-refactoring`, `researching-bug-fixing`, `planning-code-tasks`, `implementing-code`, `fixing-bugs`, `reviewing-code`, `conducting-excursion`, and the quick-lane skills `defining-quick-story` and `implementing-quick-change`. Each names §13 at its orientation / input-loading / localize step (the highest-value skills also make it an explicit Step-0 gate before any discovery scan). Skills not in this list do not read `codebase-map.md` on its account.

### 13.4 Composition with §12

§13 and §12 (Delegated Exploration) are complementary and ordered: **consult the map first (§13), then — only for what the map does not resolve — delegate the residual scoped sweep (§12).** The map shrinks what must be explored at all; delegation handles whatever exploration remains. A skill routed to both reads the map before it decides what (if anything) to fan out.

