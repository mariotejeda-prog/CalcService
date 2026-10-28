# Execution Protocol — Agent Runtime Standards

## Purpose

ROUTING: This file is self-routing. Read only the sections your skill needs,
in the order given by the routing rule below — do not preload the whole file.
Each skill discovers its sections from that rule, then reads each lazily as it
becomes relevant. Sections not selected for your skill are not required.

This file defines how agent sessions manage identity, state, context limits,
recovery, section-by-section generation, and the harness output sidecar
contract. All code-development and synthesize skills follow these patterns;
the routing rule below tells you which sections apply to yours.

---

## Skill → Required Sections (routing rule)

Skills self-route. Each SKILL.md declares its own **generation shape** inline
(`section-shape` = one multi-section spec; `list-shape` = many discrete files;
`hybrid` = section-shape manifest + list-shape per-item files) and references the
exact sections it applies at each step (`apply §10.2`, `§11`, …). Route by shape,
then layer on the role-based sections. Read in the listed order.

**Base route by shape:**
- **section-shape / hybrid** → §2 → §10 → §10.5 → §11
- **list-shape** → §2 → §10.5 → §11 — produces many files, so §10 (section generation) does not apply.

**Additive sections (layer onto the base route, in position):**
- **§1 (Session Identity)** — prepend for the code-development family, which mints/reuses a SESSION_ID in output filenames: `researching-code-design`, `planning-code-tasks`, `implementing-code`, `reviewing-code`, `reviewing-code-web`, `fixing-bugs`, `automation-code-generation`, `automation-test-execution`.
- **§12 (Delegated Exploration)** — insert right after §2 for skills that do broad read-only input sweeps: every `researching-*` skill, plus `reviewing-code`, `reviewing-code-web`, `planning-code-tasks`, `fixing-bugs`, `producing-abap-archaeology` (phases 1/2/4 sweep ABAP source exports), and `mapping-jde-fusion-data` (Phase B sweeps the JDE business-data KB and Oracle Fusion Documentation KB per in-scope table). Capability-gated and optional (see §12).
- **§13 (Code-Location Discipline)** — apply at the orientation / input-loading step for skills that locate code: every `researching-*` skill, plus `planning-code-tasks`, `implementing-code`, `reviewing-code`, `reviewing-code-web`, `fixing-bugs`, `conducting-excursion`, and the quick-lane `defining-quick-story` + `implementing-quick-change`. Read `codebase-map.md` before any repository-wide discovery scan (consult-before-scan, not never-scan; see §13). Composes with §12 — map first, then delegate the residual sweep.
- **§14 (Parallel Fan-Out)** — insert at the step that owns a same-shape independent-unit loop, for skills that declare an independent unit set. **Two axes decide the dispatch shape: unit independence (§14.1) and input shape (§14.1.1).** Routed skills, grouped by input shape:
  - **Disjoint inputs → one unit per worker.** `reviewing-code` (per changed file / diff-hunk group — own file + its direct imports/tests), `reviewing-code-web` (per changed frontend file / diff-hunk group — own file + its direct imports/tests, same shard rule as `reviewing-code`), `analyze-context-pack-inputs` (per input file — read-only extraction, coordinator synthesizes), `adversarial-platform-review` (per decision — each worker's own live lookup is the dominant cost), `db-source-profiling` (per source table — profile one table each; coordinator concats into the catalog), `db-parity-execution` (per table source↔target diff — the pass/fail verdict stays coordinator-serial; cap connection concurrency), `mapping-agent-tools` (per `TOOL-{NNN}` candidate), `planning-agent-backlog` (per agent candidate), `implementing-user-stories` + `generating-test-cases` + `generating-e2e-test-cases` (per epic — each suite/definition file derives only from that epic's own stories/journeys + ACs, so scope each worker's payload to its **epic slice**; handing a worker the whole PRD/architecture would convert this into a shared-corpus case), `researching-code-design` (per epic, Per-Epic Selective Loading Pattern ONLY — `implementation_sequencing` / `acceptance_criteria` sections, each worker reads one `epics/epic-{NN}.md` file (~60-130 lines) plus its own Tier-1 slice already resident in the coordinator's context; the OTHER 9 sections of that skill's spec are Tier-1-only rollups over a corpus every section shares and stay inline — routing those to §14 would reproduce the `validating-architecture-compliance` shared-corpus counter-example above), `secreview-static-scanning` (per (scanner, repo) pair — each pair writes its own finding files keyed by `finding_hash`; no pair reads another pair's output), `secreview-context-correlation` (per repo — each extraction reads only that repo's own source files and writes to its own `_extractions/repo-{name}.md`; no cross-repo reads occur during extraction), and the quick-lane `defining-quick-story` (per raw request in a batch intake — zero cross-request carry-forward; coordinator merges progress shards and emits the `stories[]` sidecar list serially).
  - **Shared corpus → batch-shard or run inline (§14.1.1).** `validating-architecture-compliance` (per story, but every story is judged against the same architecture + ADR + DTR corpus — the measured counter-example in §14.1.1: 8 workers 22 min vs inline 2 min 57 s; inline is now its default, batch-shard only at >= 12 stories), `generate-context-pack` (per normative context-pack file — all files derive from one upstream analysis digest; pass each worker its slice, batch-shard if no slice can be isolated).
  - **Mixed — the decorrelation exception covers only part of the unit set (§14.1.1).** `verifying-artifacts`: the whole checklist reads one shared `ARTIFACT_TEXT`, so **PROGRAM items (deterministic verifier runs) stay inline**, while **a JUDGE item's `judge_samples` draws DO fan out** — those samples must not see one another or `MEAN(samples)` stops being a variance-reduced score. Never one worker per checklist item: two different questions do not correlate, and bias-DOWN / anti-gaming are coordinator aggregation rules, not per-item isolation requirements. An all-PROGRAM checklist correctly fans out nothing.

  Capability-gated and optional — the runtime advertises `Parallel delegation: AVAILABLE/UNAVAILABLE`. Concurrent foreground workers (N spawns in one dispatch turn) satisfy the capability; background/detached tasks are NOT required and an absent advertisement does NOT mean inline (§14 Applicability). Dispatch all workers for a shard in one turn (§14.3 step 2) — then **check the workers actually overlapped in time; if the harness serialized the batch, stop dispatching and finish the remainder inline (§14.7)**. Run inline in declared order when the harness is genuinely single-dispatch, or when §14.1.1 says the shared corpus makes inline the cheaper path (see §14). Write-capable sibling of §12: compose them (map/locate units via §12, process via §14). **NOT `implementing-code`** — parallel code writes to a shared tree require per-worker git-worktree isolation (not available to subagents by default), so its plan-phase loop stays inline until that isolation is guaranteed.
- **§15 (Deliverable Upload Safety)** — append at the finalize step, immediately BEFORE §11, for skills whose declared outputs include a `.html`, `.svg`, or standalone `.js` file delivered through the platform: `rapid-prototyping`, `sap-fiori-rapid-prototyping`, `generating-flow-selection-report`, `build-design-prototype`, `glob-presentation`, `producer-visualizing-architecture`, `producer-visualizing-story-map`. Those files are uploaded through a byte-scanning WAF that rejects attack-payload-shaped text — in code, in mock data, *or* in prose — before the request reaches the application, so a file that renders perfectly can still be undeliverable. §15.4 is a mandatory pre-sidecar scan. Not routed for markdown-only skills; see §15.4's closing note when a report may quote payloads.

**Special routes (override the base route):**
- `implementing-code` → §1 → §2 → §3 (IMPL-STATE schema) → §5 (FIC monitoring) → §7 (if REPAIR) → §10.5 (+ §10.5.3 tool-output retention for build/test/lint runs) → §11
- `fixing-bugs` → §1 → §2 → §12 → §7 (if REPAIR) → §9 (non-interactive) → §10.5 → §11
- `refining-artifact-consistency` (sandbox) → §10.5 only — Stepwise owns `_progress.json` (§2) and apply-back (§11), so both are skipped; read §10.5 + §10.5.1 before any sandbox `Edit`.
- `conducting-excursion` (sandbox / Manual Bay) → §10.5 + §12 (capability-gated, narrowly — see below) only — Stepwise owns `_progress.json` (§2), the exit gate, and apply-back (§11), so both are skipped. Multi-turn: read the skill's working-set note (`EXCURSION-NOTES.md`) first on resume turns and do NOT re-run the §2/pre-flight read or re-scan source; read §10.5 + §10.5.1 before any sandbox `Edit`, and apply §10.5.3 (tool-output retention) to every build/test/deploy/probe run. §12 applies only to the rarer turn whose message is a genuinely broad sweep (e.g. tracing a symptom across the whole app, finding every place a config value is set) — the common turn that already names or implicates a single file stays inline per the skill's own minimum-evidence discipline (§12.1 already excludes single-known-file delegation).
- `authoring-deliverable` (Manual Bay / external-document authoring) → §10.5 (+ §10.5.1, + §10.5.3 for publish/render fetches) + §12 (capability-gated, narrowly) + §11 (declared outputs only) — Stepwise owns `_progress.json` (§2) and apply-back, so §2 and the lifecycle files are skipped; the deliverable is published to `document_target` through its own tool, and §11 reports the gate-consumed outputs (`authoring_status`, `document_url`, …) only. Multi-turn: read the working-set note (`AUTHORING-NOTES.md`) first on resume turns and do NOT re-run the §2/pre-flight read or re-fetch the published document to hunt for unrequested work; publish once and verify the render once (see the skill's `publish-and-render.md`). §12 applies only to the rarer turn whose message is a genuinely broad read-only sweep over many `source_artifacts` — a turn that names a single implicated file stays inline.
- `adversarial-platform-review` (discovery / conformance review) → §2 → §10.5 → §11 (+ §14 for per-decision fan-out) — list-shape: reads one bounded artifact + its declared target platform, adversarially web-verifies each load-bearing decision, and emits a single cited findings JSON + verdict (no multi-section spec, so §10 does not apply; no broad repo sweep, so §12 does not apply — the web grounding is this agent's own load-bearing work). Each load-bearing decision is an independent §14 unit (the coordinator merges + dedupes findings and decides the single verdict serially).
- `verifying-artifacts` (verification / checklist + program-verifier) → §2 → §10 → §10.5 → §11 (+ §7 if REPAIR) (+ §14 for judge-sample fan-out) — section-shape: produces one multi-section VERIFICATION-REPORT (checklist → program_verifiers → judge_scoring → aggregation → verdict). Scores SPEC items + consumed `adversarial-platform-review` findings + a universal anti-gaming item; generates/runs stdlib-only verifier programs for exactly-checkable items (RLCF Fig 6) and mean-of-N judge-samples the rest. The per-item JUDGE samples (and independent PROGRAM runs) are the §14 fan-out unit — the coordinator does the mean/bias/anti-gaming aggregation serially. No broad repo sweep, so §12 does not apply; no platform re-discovery (that is the discovery layer's job).
- `triaging-lane` (loop-entry lane routing) → §2 → §10.5 → §11 — list-shape: reads ONE decomposition-manifest entry plus the relevant sections of `codebase-map.md` and the conformance ledger, answers a fixed seven-field evidence-cited questionnaire, and EXECUTES a versioned rule-table script to emit `FULL|QUICK`. No multi-section spec, so §10 does not apply. **§12 and §13 deliberately do NOT apply** — this step never scans or searches the repository, and delegating exploration would cost more than the lap it is routing; a field it cannot answer from those three inputs stays uncited and the item routes FULL. No §14 (a single work item is not a fan-out unit).
- `analyzing-pr-integration-order` (PR-batch merge-order analysis) → §2 → §10.5 → §11 — list-shape: discovers a batch of open PRs, detects explicit dependency mentions and file-overlap between them, and emits one decomposition JSON + one rationale report (no multi-section spec, so §10 does not apply). Read-only against the tracker/forge — no §12 (the reads are N bounded per-PR API calls, not a repo-wide sweep) and no §14 (a typical batch is small enough that per-PR metadata fetch does not warrant fan-out; revisit if batch sizes grow materially).
- `integrating-pull-requests` (per-PR conflict resolution + merge) → §2 → §10.5 → §11 — list-shape: invoked once per PR by `pr-integration`'s `integrate-prs` step, resolves a real merge conflict when one exists (NIMBLE auto-resolve / DEEP escalate) and executes the merge — no multi-section spec, so §10 does not apply. Given an explicit route rather than left to the catch-all: it writes real git state (test-merges, merge commits) under real time pressure from §16/AP-004-class mistakes documented in its own references, so its §11 sidecar and §10.5 tool discipline are load-bearing, not incidental. No §12 (git/API calls are bounded per-PR, not a repo-wide sweep) and no §14 (it IS the per-unit worker for the calling step's own loop, not a coordinator with units of its own to fan out).
- `generating-flow-selection-report` (prototype flow report) → §2 → §10.5 → §15 → §11 — list-shape: reads the agentic-mode `flow-documentation.md` + `component-inventory.md` of an already-built Fiori prototype and emits ONE self-contained `flow-report.html` INTO the prototype folder (beside `public/`), each screen deep-linking via iframe into the running local prototype. No multi-section spec, so §10 does not apply; no repo sweep, so §12 does not apply. §15 is load-bearing — the `.html` is delivered through the byte-scanning WAF, and it may transcribe screen titles/routes that must survive the pre-sidecar scan.
- `android-tc-agentic-execution` (Android TC → Maestro flow, device-driven) → §2 → §10 → §10.5 → §11 (+ §4 opt-in) (+ §7 if REPAIR) — **hybrid**: the EXECUTION-JOURNAL is section-shape (ten sections, skeleton-first, one `Edit` per section), while the per-TC-step exploration records plus the flow YAML and `summary.md` are list-shape items with a seeded denominator. Apply §10.5.3 (tool-output retention) to every `maestro test`, `adb`, and emulator run, and to `report.html`. **§12 and §14 deliberately do NOT apply** — "exploration" here is stateful device interaction (driving a live emulator, mutating app state, capturing the current screen), not a read-only file sweep, so §12's `file:line` return contract cannot express it and a delegated session would race the one this agent depends on; and the TC steps are strictly ordered on a single shared device (step N+1's screen exists only because step N's action produced it), which is the strongest possible failure of §14's independence precondition. The one file-based sweep — skimming a prior `tests/*.yaml` for style — is a single known path and is read inline per §12.2.1.
- `planning-epics` (PRD → epic backlog, section-shape) → §2 → §12 → §10 → §10.5 → §11 (+ §7 if REPAIR) — section-shape: produces one multi-section EPICS-SPEC (skeleton-first, one `Edit` per section). §12 applies because the skill must load the full PRD-SPEC (potentially multi-file with batch files) before any FR-to-epic traceability claim can be made — a broad read-only sweep over unbounded input. No §14 (sections share the same PRD corpus and depend on prior-section output such as themes → epics ordering, so per-section fan-out would reproduce the shared-corpus counter-example).
- `plan-epics-definition` (requirements → epic backlog, standalone — no formal PRD required; section-shape) → §2 → §12 → §10 → §10.5 → §11 (+ §7 if REPAIR) — section-shape: produces one multi-section EPICS-SPEC (12 sections; 13 when a PRD-SPEC is auto-discovered) plus a metadata-only EPICS-AUDIT, skeleton-first with one `Edit` per section. §12 applies to the input-loading sweep, which has two parts, both read-only: (1) the PRD-SPEC auto-discovery in Step 1, **bounded to exactly two path patterns** — `PRD-SPEC-*.md` under `requirements_input_path` itself, then `./artifacts/outputs/*/prd/` — first search that returns any file wins; one match is read and feeds `prd_scope_validation`, more than one match is NOT merged (it becomes a single non-blocking `open_questions` row), none means standalone mode; (2) the Step 2 requirements ingestion over every file under `requirements_input_path`, which must be read in full before any FR-to-epic claim (never synthesize epics from a partial read). No §14 (sections share the same requirements/PRD corpus and later sections depend on earlier output — themes → epics → sequencing — so per-section fan-out would reproduce the shared-corpus counter-example).
- `analyzing-jde-fusion-scope` (JDE/Fusion scope extraction — Task 1.1 / Process Step 1 of `jde-fusion-integration-discovery`) → §2 → §10.5 → §11 — list-shape: reads the client-provided `input_document_path` once and extracts JDE Tools Release + implemented JDE modules + target Fusion modules/components into one `jde-fusion-scope-summary.md` (no multi-section spec, so §10 does not apply). Single bounded input read directly, not a broad sweep, so §12 does not apply. Single deliverable, no independent-unit loop, so §14 does not apply.
- `researching-fusion-integration-mechanisms` (JDE/Fusion mechanism research — Task 1.2 / Process Step 2 of `jde-fusion-integration-discovery`) → §2 → §12 → §10 → §10.5 → §11 — section-shape: produces one multi-section `integration-mechanism-research-findings.md` (skeleton-first, one `Edit` per section) evaluating REST/SOAP/FBDI per target Fusion module, RAG-grounded in the Oracle Fusion Documentation KB, JDE Orchestrator Guides KB, and the ErpIntegrationService reference documentation. §12 applies because the skill must broadly sweep three separate knowledge bases per module before any mechanism verdict can be cited — read-only exploration over an unbounded KB corpus. No §14 (the per-module evaluation loop shares the same 3-KB corpus across all modules — a shared-corpus shape per §14.1.1 — and module counts per scope are small, so fan-out was evaluated and rejected in favor of the inline loop).
- `specifying-integration-mechanism` (JDE-Fusion mechanism specification, section-shape) → §2 → §10 → §10.5 → §11 — section-shape: produces one multi-section `integration-mechanism-specification.md` (mechanism_decision → rest_soap_specification → fbdi_specification → other_mechanism_specification → assumptions_to_validate → open_questions) from the bounded `researching-fusion-integration-mechanisms` output plus cited Oracle FBDI/`ErpIntegrationService` documentation. No broad repo/input sweep, so §12 does not apply; single mechanism decision drives generation, so no independent-unit loop and §14 does not apply.
- `mapping-jde-fusion-data` (JDE-to-Fusion field-level data mapping matrix) → §2 → §12 → §10 → §10.5 → §11 — section-shape: produces one multi-section Data Mapping Matrix (scope/mechanism context → methodology → per-table field mappings → unmapped fields and gaps → coverage summary → open questions), skeleton-first with one section per `Edit` call. §12 applies because Phase B must sweep the JDE business-data KB and Oracle Fusion Documentation KB per in-scope JDE table before any field mapping can be proposed — a broad read-only sweep over unbounded knowledge-base content when no client mapping file is supplied. No §14 (per-table generation shares the same mechanism_shape context across tables, so parallelizing them would reproduce the shared-corpus counter-example — tables stay coordinator-serial inside Phase C).
- `generating-jde-orchestrator-transformations` (JDE Orchestrator Groovy transformation code generation, Task 2.1) → §2 → §10.5 → §11 — list-shape: reads the `integration_mechanism_specification` and `data_mapping_matrix` produced by the discovery stream, classifies each mapping row as pass-through or custom-transform, and emits ONE terminal `groovy-transformation-scripts.md` report (no multi-section spec, so §10 does not apply; no broad repo/artifact sweep, so §12 does not apply; the per-row Groovy generation loop is bounded, structured pattern-matching, so no §14 fan-out).
- `generating-jde-orchestrator-configuration-guide` (JDE Orchestrator Configuration Guide generation) → §2 → §10 → §10.5 → §11 — section-shape: produces one multi-section `orchestrator-configuration-guide.md` (Connectors, Service Requests, Orchestrations, and cross-references), skeleton-first with one section per `Edit` call. Grounded in the Integration Mechanism Specification, Data Mapping Matrix, and Groovy transformation scripts from the discovery and build streams; for SOAP/FBDI mechanisms, also grounded in the Oracle ERP Integration whitepaper. No broad repo sweep, so §12 does not apply; single guide output, so §14 does not apply.
- `validating-erp-integration-reference` (ERP Integration Service reference pre-flight validation) → §2 → §11 — preflight shape: reads the Integration Mechanism Specification to detect the selected mechanism, checks erp_integration_service_reference_path readability for SOAP/FBDI mechanisms, emits PASS or BLOCKING. Writes one preflight sidecar; no deliverable artifact, no §10/§12/§14 sweep.
- **Catch-all** — any skill not otherwise classified → §2 → §10.5 → §11.

Per-skill specifics that are NOT routing (e.g. MCP/Deterministic-API tooling, manifest-vs-per-file split, approval-token gates) live in each skill's own SKILL.md, not here.

### Universal rules (apply to every skill, regardless of routing)

A skill that discovers its sections by scanning for `## `/`### ` headers and
jumping straight to the ones its own route names by number will land on §2,
§10.5, §11 etc. directly — those have their own headers. It will NOT land on
this paragraph unless it is scanning headers, because until this heading was
added, this list had none: a header-jumping skill never learned these rules
existed at all, headers or not, since nothing pointed here. Observed
2026-09-04: an `opening-pull-request` run's own header-only section scan
(`grep -n "^## §2\|^## §10.5\|^## §11\|..."`) skipped straight past this
paragraph and never discovered §16 — the very rule that would have stopped it
searching the filesystem for its own reference file. Read this list
unconditionally, not only when your route's header-scan happens to surface it.

- §2 `_progress.json` FIRST ACTION / LAST ACTION write — prevents orchestrator SIGINT. **Exception:** sandbox-mode skills (e.g., `refining-artifact-consistency`, `conducting-excursion`) run inside a Stepwise-managed sandbox where Stepwise owns `_progress.json`; those skills skip §2 and use §10.5 only.
- §2.c path budget — at FIRST ACTION, measure the LONGEST path this run will write (your artifact filename, never `_progress.json`) and probe it once when it exceeds 240 characters. A successful `_progress.json` write is not evidence the output folder is usable: on Windows an over-long path surfaces as `ENOENT` and reads as a missing directory, and diagnosing that with a shell call is how a path error becomes a headless approval denial. Same exception as §2 (sandbox-mode skills).
- §10.5 tool discipline — applies whenever a skill writes or edits files, NOT only to section-shape skills. The destructive `edit` UTF-8 bug observed 2026-05-19 (zeroed out a target file on encoding failure with non-ASCII `new_string` content) is RESOLVED as of 2026-09-09 — see §10.5.1 for verification evidence. Targeted `Edit` is the path for every language now; no fallback needed.
- §11 harness output sidecar — last write before `final_response`. Without it the orchestrator records `outputs: {}` and downstream steps cannot pick up declared output parameters. **Exception:** sandbox-mode skills skip §11 because Stepwise owns the apply-back step; see the `refining-artifact-consistency` and `conducting-excursion` special routes above (`§10.5 only`).
- §11.6 open questions in the sidecar — `_open_questions` is a reserved, required row of the `## Output parameters` table: write it in the same sidecar as your declared outputs, listing every open question THIS run raised, `[]` when it raised nothing. Only questions THIS run raised: never re-list one the prompt handed you in `## OPEN QUESTIONS`, `## DECISIONS` or `## PROJECT DECISIONS`, in either place. Zero-Invention already tells you to raise an open question rather than guess (§9); this is what makes the raised question trackable instead of prose the orchestrator has to guess at. Same exception as §11.
- §4 Memory Bank session-end writes (`active-context.md`, `progress.md`) — only when the skill's SKILL.md explicitly opts in; otherwise skip.
- §7 REPAIR bookkeeping — when running in REPAIR mode (a `## ⚠️ Repair feedback` block is present), editing the artifact in place is NOT enough: you MUST also **bump the spec `version`** (semver patch, e.g. `1.0.0` → `1.0.1`) AND **record what changed in the AUDIT file** (set `mode: REPAIR` and append a `## Repair History` entry). Skipping this leaves the version and audit trail stale even though the content was fixed — the documented "iteration applied but version/audit not updated" defect. See §7.2 steps 7–8 for the exact fields and the mandatory self-check.
- §16 skill reference file resolution — whenever your own SKILL.md tells you to read a `references/*.md` file, resolve it relative to your own skill directory. Never `glob`/`find` the repo or filesystem to locate it. See §16 for why this hangs and how to recover.
- §12.6 / §13 both already forbid unscoped search **of source code under analysis**; §16 is the narrower, distinct rule for a skill's own bundled files and applies even to skills whose route omits §12/§13 entirely (e.g. `defining-quick-story`, list-shape/catch-all skills).

**REPAIR mode override:** if the prompt contains a `## ⚠️ Repair feedback` block, read §7 (REPAIR Mode) before any other section — even when §7 is not listed in your route. §7 carries the version-bump + audit-delta bookkeeping that is mandatory for every REPAIR (see the universal rule above); the routing rule above intentionally omits it to stay short, so this override is the authoritative trigger.

**REBUILD mode override (discard):** if the prompt contains a `## ♻️ Rebuild from scratch` block instead of `## ⚠️ Repair feedback`, read §7.3 (REBUILD Mode) before any other section. This is a DISCARD, not an iterate: the reviewer judged the artifact fundamentally wrong, so you must REGENERATE it from scratch (full BUILD) rather than surgically repair it — while still overwriting the SAME file/SESSION_ID (never duplicate) and doing the version-bump + audit bookkeeping (`mode: REBUILD`). REBUILD and REPAIR are mutually exclusive: only one of the two blocks is ever present.

## Section Index

- §1  — Session Identity (SESSION_ID format, REPAIR reuse rules)
- §2  — `_progress.json` early-signal write (FIRST ACTION / LAST ACTION)
  - §2.a Seed `total` once the item count is known (never leave `total: 0`)
  - §2.b Co-locate the tracker update with the checkpoint write in every production loop
  - §2.1 Pre-completion self-validation gate (universal — completeness · placeholders · grounding · format)
- §3  — IMPL-STATE schema (implementing-code only)
- §4  — Memory Bank cross-session continuity (`active-context.md`, `progress.md`)
- §5  — FIC context monitoring and recovery checkpoint
- §6  — Dual-format input detection
- §7  — REPAIR mode (when `failure_feedback` is provided); §7.3 — REBUILD mode (discard → regenerate)
- §8  — Deviation taxonomy (Auto-Fix / Escalate / Defer)
- §9  — Non-interactive execution rules
- §10 — Section-by-section spec generation (skeleton-first + one-section-per-write)
  - §10.1 Section-shape `_progress.json` schema
  - §10.2 Source extraction pass
  - §10.3 Phase A — Skeleton-first
  - §10.4 Phase B — Populate, one section at a time
  - §10.5 Tool discipline (hard rules; §10.5.1 non-ASCII fallback RETIRED — see below)
    - §10.5.3 Tool-output retention (context-budget: run full, retain only the verdict / failing slice / distilled evidence)
    - §10.5.4 Values you cannot compute — a checksum is RUN, never written from memory
  - §10.6 CONTINUE on re-entry
- §11 — Harness output sidecar (required when running under Stepwise)
  - §11.1.1 Path correctness (report the path you actually wrote to)
  - §11.1.2 Verdict fidelity (closed-set tokens verbatim; synonyms fail closed)
  - §11.6 Open Questions in the sidecar (the reserved `_open_questions` row — only what THIS run raised, `[]` when nothing)
- §12 — Delegated exploration (read-only subagent fan-out, harness-agnostic, optional)
  - §12.1 When to delegate (and when not)
  - §12.2 The return contract (conclusions + `file:line`, never file dumps)
    - §12.2.1 Anti-patterns — hard stops (no verbatim dumps, no known paths, no duplicate reads)
  - §12.3 Model selection (prefer the cheap/fast tier)
  - §12.4 Verify before use (subagent output is input, not ground truth)
- §13 — Code-location discipline (consult `codebase-map.md` before any discovery scan; code-searching skills only)
  - §13.1 The rule — consult before scan, not never scan
  - §13.2 When the map is insufficient — flag the gap
  - §13.3 Applies to (code-searching skills only)
  - §13.4 Composition with §12
  - §12.5 Fallback (no subagent capability → inline exploration)
  - §12.6 Skill-level enforcement
- §14 — Parallel fan-out (write-capable subagent fan-out over independent units; harness-agnostic, optional)
  - §14.1 When to fan out (and when not)
  - §14.2 Worker contract (incl. §14.2.1 sharded progress)
  - §14.3 Coordinator contract (shard → dispatch → merge → sidecar last)
  - §14.4 Verify before use
  - §14.5 Fallback (no delegation capability → inline unit loop)
  - §14.6 Skill-level enforcement
- §15 — Deliverable upload safety (web-shaped deliverables cross a byte-scanning WAF; HTML/SVG/JS-emitting skills only)
  - §15.1 Recognising the failure (403 + `text/html` = the request never reached the app)
  - §15.2 Forbidden byte patterns (chained bare `alert(`, SQLi signatures — in code, mock data, or prose)
  - §15.3 Authoring rules (no native browser dialogs; describe payloads, don't reproduce them)
  - §15.4 Pre-delivery scan — mandatory gate before the §11 sidecar
  - §15.5 Optional live-firewall verification
- §16 — Skill reference file resolution (universal — every skill with a `references/` folder)
  - §16.1 The rule — resolve relative to the skill's own directory, never search for it
  - §16.2 Why unscoped search fails here (`.coda/skills/` is gitignored; a gitignore-respecting `glob` returns zero hits even though the file exists)
  - §16.3 If a bundled reference truly appears missing

---


## Section files

Non-universal sections now live as individual files under `execution-protocol/`.
Read only those your route selects (see the routing rule above); each is readable
standalone with its original § numbering intact. The **universal** sections (§2,
§10.5, §11 — required by every skill) remain inlined below.

| § | Location |
|---|----------|
| §1  Session Identity | `execution-protocol/01-session-identity.md` |
| §2  `_progress.json` | inline below (universal) |
| §3  IMPL-STATE | `execution-protocol/03-impl-state.md` |
| §4  Memory Bank | `execution-protocol/04-memory-bank.md` |
| §5  FIC recovery | `execution-protocol/05-fic-recovery.md` |
| §6  Dual-format | `execution-protocol/06-dual-format.md` |
| §7  REPAIR / §7.3 REBUILD | `execution-protocol/07-repair-mode.md` |
| §8  Deviations | `execution-protocol/08-deviations.md` |
| §9  Non-interactive | `execution-protocol/09-non-interactive.md` |
| §10 Section generation | `execution-protocol/10-section-generation.md` |
| §10.5 Tool discipline | inline below (universal) |
| §11 Output sidecar | inline below (universal) |
| §12 Delegated exploration | `execution-protocol/12-delegated-exploration.md` |
| §13 Code-location | `execution-protocol/13-code-location.md` |
| §14 Parallel fan-out | `execution-protocol/14-parallel-fanout.md` |
| §15 Deliverable upload safety | `execution-protocol/15-deliverable-upload-safety.md` |
| §16 Skill reference file resolution | inline below (universal) |

A skill's route (e.g. `§2 → §10 → §10.5 → §11`) resolves each §N via this table:
universal sections are already here; the rest are one file-read away. Existing
"see execution-protocol.md §N" citations in the 123 skills keep working through
this one redirect hop — no skill edits required.

---

## 2. _progress.json — Early Signal Write

**Purpose:** Prevents the orchestrator coordinator from sending SIGINT to a running skill.

Write this as the **FIRST file action** in any skill execution (before creating any other outputs):

```json
{
  "skill": "{skill-name}",
  "session_id": "{SESSION_ID}",
  "status": "RUNNING",
  "started_at": "{ISO-8601 UTC, e.g. 2026-08-15T13:21:37Z}",
  "completed_at": null,
  "total": 0,
  "completed": 0,
  "items": []
}
```

Write to: `{output_folder}/_progress.json`

**Before that write, check the path budget — §2.c.** A successful `_progress.json`
write is not evidence the output folder can hold your actual artifact.

**Update cadence:**
- On FIC trigger (Section 5): `"status": "COMPACTION_NEEDED"`
- On completion: `"status": "COMPLETED"`, `"completed_at": "{ISO-8601 UTC}"`
- On failure: `"status": "FAILED"`, `"completed_at": "{ISO-8601 UTC}"`

**UTC, not local time with a `Z` on it.** Read the real clock and convert. A local timestamp
labelled `Z` reads as a lie about when the work ran — Stepwise stamps its own state in true UTC,
so the two disagree by your offset and every reviewer sees fabricated evidence where there was
only a wrong clock. If you cannot get UTC, omit the field rather than guess.

**Status vocabulary — closed set.** Every value carries distinct semantics; the set is
closed so a monitor can classify any tracker without knowing which skill wrote it.

| status | meaning | live? |
|---|---|---|
| `RUNNING` | step is executing | yes |
| `PARTIAL` | chunked step finished its chunk with work remaining; a later invocation resumes | yes |
| `COMPACTION_NEEDED` | FIC trigger (§5); awaiting compaction | yes |
| `COMPLETED` | all declared work done | no — terminal |
| `FAILED` | step could not complete | no — terminal |
| `SCOPE_REFUSED` | input exceeded the skill's scope envelope; step declined and recommended decomposition rather than producing a degraded artifact | no — terminal |

**Never invent a synonym for a value already in the table** — `IN_PROGRESS`, `STARTED`,
`OK`, `DONE` and similar are prohibited precisely because they mean `RUNNING` or
`COMPLETED` while reading as unrecognized to a monitor keying on the set above.

Adding a genuinely new state (distinct semantics, not a synonym) means adding a row to
this table in the same change. Do not introduce one ad hoc in a single skill: an
undocumented status is indistinguishable from a typo at read time.

Item-level statuses inside `items[]` / `sections{}` are a **separate** vocabulary and are
not governed by this table — `"status": "complete"` on an entry is fine.

### 2.a Seed the denominator (list shape — MANDATORY)

The skeleton in the template above ships `"total": 0` because the count is not yet known
at FIRST ACTION. It is a placeholder, **not** the value you leave behind.

**As soon as the item count is known — and before the first item is produced — write
`total` with the real number.** For most skills this is the end of the planning/discovery
step, once the catalog of services / contexts / ADRs / units / diagrams exists:

```
total = len(ITEMS_TO_PRODUCE)      # + any fixed tail artifacts (manifest, audit)
UPDATE _progress.json → { "total": total }
```

A tracker still reporting `total: 0` after production has begun is a defect, not a
cosmetic gap: `0/0` is indistinguishable from "hung before doing anything", which is
exactly the ambiguity this file exists to remove. Observed 2026-08-09
(`establishing-architecture-foundation`): a 27-minute run showed `0/0` end to end while
12 service files and a 36 KB manifest were being written.

### 2.b Co-locate the tracker update with the checkpoint write (MANDATORY)

Skills with a Write-Flush-Forget production loop (§5) typically maintain **two** state
files: `_checkpoint.json` (rich resume state, internal) and `_progress.json` (thin
liveness signal, read by the orchestrator and by humans). The observed failure is that
per-item bookkeeping lands in the checkpoint and never in the tracker — the loop is
invisible from outside for its entire duration.

**Rule: the `_progress.json` update is part of the same loop step as the checkpoint
write. Never one without the other.** Both go in the same numbered sub-step so a skipped
write is visible when reading the skill:

```
## WRITE CHECKPOINT + UPDATE PROGRESS (single step — never split)
   1. Serialize INDEX to CHECKPOINT_FILE (_checkpoint.json)
   2. UPDATE _progress.json → completed += 1, items += [{id, status: "complete", file}]
```

This applies to every discrete unit the loop emits: per-service, per-context, per-ADR,
per-unit, per-diagram, per-tab.

**Timestamps come from a real clock read.** `started_at` and `completed_at` MUST be
actual timestamps at the moment of writing — never round numbers back-filled at the end
to describe what the run "would have" done. A `completed_at` that postdates the file's
own mtime is a fabricated tracker and fails §2.1 check 5.

**Schema variants.** A skill picks ONE of the three shapes below based on its work model
and uses it consistently for the whole run. The common envelope (`skill`, `session_id`,
`status`, `started_at`, `completed_at`) is identical across variants.

- **List shape (default — shown above)** — uses `total`, `completed`, `items[]`. Suits
  skills that produce many discrete artifacts (e.g. `implementing-code` writing source
  files; `planning-code-tasks` tracking task entries).
- **Section shape** — uses `skeleton_written` (bool) + `sections{}` map. Suits skills
  that produce a single multi-section spec file from large unstructured inputs. See
  **Section 10** for the full protocol (skeleton-first + one-section-per-write +
  source extraction + CONTINUE re-entry).
- **Hybrid shape** — carries BOTH: `skeleton_written` + `sections{}` for the manifest,
  AND `total` / `completed` / `items[]` for the per-item files produced alongside it.
  Suits skills that emit a sectioned manifest *and* a fan-out of per-entry artifacts
  (e.g. `secreview-static-scanning`: `sections{}` tracks the SCAN-MANIFEST while
  `items[]` grows one entry per finding). See §10.1.

**A hybrid tracker MUST say so.** State the variant in the FIRST ACTION callout —
"write `_progress.json` using the **hybrid** variant (per §10.1)" — so a reader can tell
a deliberate hybrid from a skill that drifted between two shapes. Carrying both field
groups *without* that declaration is the drift failure §2.1 check 5 catches:
`establishing-architecture-foundation` (fixed 2026-08-09) incremented list-shape counters
in one phase and wrote `sections{}` in the next, leaving no coherent denominator.

Both halves of a hybrid follow their own rules: §2.a seeds `total` for the `items[]` half,
§10 governs the `sections{}` half. `secreview-memory-synthesis` and the `pentest-*` family
are the reference implementations.

### Skill-Level Enforcement (FIRST ACTION / LAST ACTION)

Every skill MUST include these two inline callouts — agents do not reliably follow
protocol-only references for session bookend writes:

**At session start** (after folder creation, before any other file write):

> **FIRST ACTION — MANDATORY:** Write `_progress.json` to the output folder before any other file write.
> This prevents the orchestrator from sending SIGINT.

Include the minimal JSON schema inline in the skill so the agent can write immediately
without reading this file first. Use `"session_id": "initializing"` as placeholder.

**At session end** (after all spec files written, after Memory Bank writes):

> **LAST ACTION — MANDATORY:** Update `_progress.json` status to `COMPLETED` with `completed_at` timestamp.
> If the session failed, set status to `FAILED` instead.

These callouts are NOT infrastructure duplication — they are the minimum viable bridge
between protocol and agent behavior. The full lifecycle lives here; the callouts trigger it.

**For `implementing-code` only** — also include `output_contract` (survives compaction):

```json
{
  "skill": "implementing-code",
  "session_id": "{SESSION_ID}",
  "status": "RUNNING",
  "output_contract": {
    "progress_folder_path": "{progress_folder_path}",
    "impl_state_file": "IMPL-STATE-{session_id}.md",
    "source_path": "{source_path}",
    "expected_outputs": ["source_path", "impl_state_path"]
  }
}
```

Pin the same values in the IMPL-STATE header under `output_contract`. After context
compaction, read IMPL-STATE header (not filesystem) to recover the output path and format.

### 2.c Path budget — measure the LONGEST path, not the first one (MANDATORY)

`_progress.json` is the shortest filename a run ever writes. Writing it successfully
proves nothing about the artifact that follows it, and on Windows the gap between the
two is the entire failure.

**At FIRST ACTION, alongside the `_progress.json` write, compute the budget from the
longest path THIS run will produce:**

```
BUDGET = len(abspath(output_folder)) + 1 + len(longest filename this run emits)
         # your own artifact name — IMPL-STATE-{SESSION_ID}.md,
         # PLAN-REVIEW-{SESSION_ID}.json, <section>-{SESSION_ID}.md —
         # NEVER _progress.json, which is 14 characters
BUDGET > 240  ->  write one probe file at exactly that length, then delete it
```

240, not 260: `.runlogs/<name>-<attempt>.txt`, sidecars and per-item files live under
the same root and outrun the artifact that set the budget.

**A probe failure is a PATH BUDGET failure, not a missing directory.** The signature is
counter-intuitive, and misreading it is the reason this rule exists:

| symptom | what it actually means |
|---|---|
| `ENAMETOOLONG` on write | over the limit — unambiguous |
| `ENOENT` on a path whose parent directory you just wrote into | over the limit. Windows returns `ERROR_PATH_NOT_FOUND` for an over-long path on some APIs and `ERROR_FILENAME_EXCED_RANGE` on others; the first surfaces as `ENOENT` and reads as "the folder is missing" |

**On a probe failure, do NOT investigate with shell calls.** The folder is fine, `mkdir`
reports success, and a compound diagnostic command is exactly what a headless
authorization policy fails closed on — converting a legible path error into an
illegible approval denial. Instead, in this order:

1. `_progress.json` → `status: FAILED` with a real `completed_at`. It is writable; it is
   short. It is the only artifact that survives this.
2. `stepwise session exec-fail --message "path budget exceeded: <N> chars at <path>;
   shorten the project root or the output slug"`.
3. Never end the turn in prose. A prose explanation exits 0 and the orchestrator records
   the step as successful (§2.1).

You cannot repair this from inside the run: the path is composed upstream, from the
project root and the parameters handed to you. Reporting it precisely IS the fix.

Two contributors routinely blow the budget and neither is capped by §1. The directory
segment comes from `feature_id` and §1 governs only the SESSION_ID *inside* a filename,
not the folder path around it. And §1's own `≤ 32 chars` cap applies to a MINTED
SESSION_ID — the fallback branch, "the runtime's `session = '...'` name verbatim", has
no cap at all. Name both in the `exec-fail` message so the operator fixes the right one.

Observed 2026-09-23 (`code-development-lean`, `pli-20260923-9as14x` iter-2): an
80-character sub-goal slug plus a 57-character SESSION_ID in the filename put
`IMPL-STATE-*.md` at 267 characters. `_progress.json` wrote fine at 210. The step read
the resulting `ENOENT` as a missing output folder, issued a compound PowerShell
diagnostic to inspect it, had that denied by headless authorization, and ended in prose
with no IMPL-STATE, no sidecar, a tracker still reading `RUNNING`, and a harness that
recorded the step as a success. The plan-review step had hit the identical 267-character
limit nine minutes earlier.

---

### 2.1 Pre-Completion Self-Validation Gate (universal — generative skills)

**Purpose:** Prevent predictable, repeatable gaps from reaching the human quality gate.
A generative step MUST self-validate against the checklist below **immediately before**
the LAST ACTION completion write (`_progress.json.status: COMPLETED`). This runs on the
agent's own output, before any human sees it.

Run all five checks. Each is tech-agnostic — no framework, language, or project name.

1. **Input-contract completeness.** Every required input, section, and output-contract
   item declared by the skill/step is addressed in the artifact. No declared section is
   empty, stubbed, or silently omitted.
2. **No unresolved placeholders.** The artifact contains no lingering `[TBD]`,
   `[to be identified]`, `[PLACEHOLDER]`, `TODO`, or unanswered `OQ-###` / open-question
   left as if resolved. Each is either resolved from available evidence OR surfaced
   explicitly as a pending item / assumption / blocker — never left silent. An
   `OQ-########` id that appears in the prompt's `## DECISIONS` / `## PROJECT DECISIONS`
   block is settled — the artifact must apply the decision and cite the id, never
   list it as open.
3. **Context-pack / source grounding.** Every non-trivial claim, path, identifier,
   technology, decision, estimate, and status traces to context-pack, upstream artifacts,
   or the source under analysis. Nothing is invented. If evidence is missing, mark the
   uncertainty visible rather than fabricating a resolution.
4. **Format-contract conformance.** Output matches the skill's declared structure:
   headings, stable IDs, tables, schemas, traceability links, filename, and status
   conventions.
5. **Tracker integrity.** `_progress.json` is a truthful record of the run, not a summary
   written at the end. Check the half/halves your declared §2 variant actually uses:
   - **list half** (list or hybrid): `total > 0` — a `COMPLETED` step still showing
     `total: 0` never seeded its denominator per §2.a — and `completed == total`.
   - **section half** (section or hybrid): `skeleton_written == true` and every declared
     section has a terminal status.
   - **all variants**: the tracker matches the variant the skill declared at FIRST ACTION
     and did not drift into another mid-run; `status` is from the §2 closed set; and
     `started_at` / `completed_at` are real clock reads, with `completed_at` not
     postdating the write itself.

   Carrying both field groups is NOT a failure when the skill declared the hybrid variant
   — that is the sanctioned third shape. It IS a failure when the skill declared list or
   section and grew the other half mid-run.

**On failure of any check:**
- If self-repairable from available evidence → repair, then re-run the checklist.
- If it requires information the agent does not have → do **not** mark `COMPLETED`.
  Record the gap as a visible pending item / open question / assumption in the artifact
  and (when a blocker) escalate per §8 Deviation taxonomy. Marking a step COMPLETED with
  a known, silent gap is a protocol violation.

**If you cannot finish the work at all — the run out of room / out of budget path.**
This is distinct from a gap in the inputs: the plan is sound, some items are done, and you
have concluded you cannot produce the rest in this execution. Ending your turn with a prose
sentence saying so is NOT a failure signal. The process exits 0, the orchestrator reads a
clean exit as success, the step is marked `completed`, and the next step runs against
whatever half-built output exists. Three things are mandatory instead:

1. **Leave the tracker truthful.** Do not set `COMPLETED`. `completed` and `total` must
   describe what is actually on disk, and every finished item must be listed. A later
   REPAIR/CONTINUE run resumes from exactly this, so an accurate tracker is what makes the
   work recoverable rather than repeated.
2. **Persist the durable checkpoint** — whatever your skill uses to resume (`_checkpoint.json`,
   the carry-forward index). Write it before you stop, not after you decide to stop.
3. **Signal failure explicitly.** Call `stepwise session exec-fail --message "<what is done,
   what remains, where to resume>"`. That message is what a human reads and what a retry
   acts on. Never let an unfinished run return normally.

Observed 2026-09-06 (`generating-adrs`, `pli-20260906-1vz9ubr`): the run produced 2 of 6
ADRs and none of the manifest, left `_progress.json` at `status: RUNNING, completed: 2,
total: 7`, said "I'm unable to complete the remaining ADR generation within this execution
window" and returned. `EXECUTION_SUCCESS, exitCode=0`. The step was marked completed, the
downstream review step reviewed the two ADRs that existed and passed them, and the
verification step then failed on a manifest that was never written — three steps and a
wasted lap to surface a failure the producing step already knew about.

**Budget awareness.** Check before starting each item, not after: if finishing the next
item would exhaust your room, stop at the item boundary and take the three steps above.
Stopping cleanly between items with an accurate tracker is a good outcome. Stopping
mid-item, or stopping silently, is not.

**Skill-Level Enforcement.** Generative skills SHOULD include a one-line inline callout
at their finalization step so the agent triggers this without reading the protocol first:

> **PRE-COMPLETION CHECK — MANDATORY:** Before the LAST ACTION completion write, self-validate
> the artifact against §2.1 (input-contract completeness · no unresolved placeholders ·
> context-pack grounding · format conformance · tracker integrity). Repair or surface gaps;
> never complete silently. A `COMPLETED` tracker still showing `total: 0` fails check 5.

This gate is the counterpart to the FIRST/LAST ACTION callouts in §2: FIRST ACTION opens
the step, §2.1 qualifies the output, LAST ACTION closes it.

---


---

### 10.5 Tool Discipline (Hard Rules)

These rules exist because each one has been observed as a failure mode. Violations defeat the protocol even when the prose discipline appears to be followed.

- **Skeleton write: use `Write`.** Single tool call, single file creation. Do NOT compose the skeleton via `Bash` heredoc.
- **Section population: use `Edit`.** Targeted find-and-replace, one section per call. The `Edit` tool handles unicode natively and validates the prior content matches what you expect.
- **NEVER use `Bash` + `sed` / `awk` / `python3 <<EOF` / `cat >` to mutate SPEC_FILE.** These are bulk-rewrite paths disguised as targeted operations. They read the whole file into memory, transform it, and write it back — the exact pattern Phase B is designed to prevent. They also bypass `Edit`'s prior-content validation, so a bug in the script silently corrupts the spec.
- **If `Edit` fails on a section, fix the `Edit` call.** Provide more surrounding context to make `old_string` unique. Do NOT switch to `Bash` / `python3` as a workaround — that path has been observed to cascade: agent hits one snag, then rewrites all remaining sections in a single script, violating one-section-per-write.
- **NEVER compose multiple sections in memory before writing.**
- **NEVER emit more than one section per Write/Edit call.**
- **NEVER re-read raw sources during Phase B.** Read `_extractions/*.md` files, or SPEC_FILE for declared cross-section state.
- **RETIRED (2026-09-09):** this bullet previously told agents to swap decorative non-ASCII glyphs (em dashes, smart quotes, `…`, `•`, `×`, NBSP) for ASCII equivalents, purely to stay on the targeted-`Edit` path and avoid the §10.5.1 fallback. That fallback is retired (see §10.5.1) — targeted `Edit` now handles non-ASCII content of any kind, decorative or required, natively. No transliteration is needed for tool-safety reasons. Accented and localized characters in non-English output (`á é í ó ú ñ ü ç ã …`) remain content to preserve exactly, as always.

If a section's content genuinely will not fit a single `Edit` call (extreme size), split that one section into sub-edits — but every sub-edit is still an `Edit` call against SPEC_FILE, never a Bash mutation.

---

#### 10.5.1 Non-ASCII Content Fallback — RESOLVED, retired 2026-09-09

**Status:** RESOLVED. The runtime bug this subsection worked around is fixed. Targeted `Edit` is the path for every section, in every language — there is no fallback to apply. This subsection is kept (not deleted) only so the many existing `§10.5.1` citations across skill files keep resolving to something; do not add new detection logic that routes around targeted `Edit`.

**History (for context, no longer actionable).** Some runtimes' targeted-edit operations were observed (first noted 2026-05-19) to fail with `'ascii' codec can't encode character ...` errors when `new_string` contained non-ASCII codepoints (accented vowels in Spanish / French / Portuguese / German, em dashes, smart quotes, etc.), and the failure was destructive — the tool truncated the file before catching the encode exception, zeroing prior content. The workaround was to detect non-English/non-ASCII content and force full-file replace (`Write`) instead of targeted `Edit` for the whole run, at ~5× the token cost.

**Verification (2026-09-09).** Re-tested directly against the current runtime: targeted `Edit` calls with `new_string` containing Spanish accents/punctuation (á é í ó ú ñ ü ¿ ¡), em dash, smart quotes, CJK, and emoji — single and back-to-back — all succeeded with correct UTF-8 output, no truncation, sibling content untouched. The underlying tool fix described below appears to be in place.

**If this ever regresses:** the two-line runtime fix this subsection used to wait on was (1) UTF-8 encoding in the edit tool's write path, (2) atomic-write semantics so a failed edit leaves the prior file intact. §10.5.2 (Safe-Write Protocol) stays in force regardless as a general defensive backup — it is not specific to this bug and was never retired.

---

#### 10.5.2 Safe-Write Protocol (mandatory defensive backup for SPEC files)

**Why this exists.** Even with the discipline in §10.5, a single bad targeted-edit call has been observed to truncate a SPEC file to 0 bytes mid-run (authored work wiped) — the non-ASCII-specific case is resolved (§10.5.1), but any other cause of a bad `Edit` call is still possible. The Safe-Write Protocol adds a `.bak` snapshot and a destructive-edit refusal gate so a single bad call cannot destroy authored work.

**Applies to:** any skill writing to `RESEARCH-SPEC-*.md`, `PLAN-SPEC-*.md`, `IMPL-STATE-*.md`, `REVIEW-SPEC-*.md`, or any other SPEC-shaped artifact the skill declares.

**Rule 1 — Defensive backup before any large edit:**

Whenever you must edit a SPEC file larger than 20 KB AND the edit changes more than 30% of the file's lines:

1. Compute: `lines_to_change` (new content vs current).
2. If `lines_to_change > 0.30 * total_lines AND file_size_kb > 20`:
   - `cp {target_file} {target_file}.bak.{ISO_TIMESTAMP}` via a single Bash call.
   - Proceed with the edit.
   - On success (post-edit file is non-empty AND > 50% of pre-edit size): delete the `.bak` file.
   - On failure or suspected truncation (post-edit file is < 50% of pre-edit size): restore from `.bak`, then emit a structured event:
     ```json
     {"event":"safe_write_restore","file":"<path>","reason":"post_edit_too_small","pre_size":<N>,"post_size":<M>}
     ```

**Rule 2 — Never `replace_all=true` on SPEC files.** Reinforces §10.5 "Section population: use `Edit`." Use targeted, anchored Edits with non-empty `old_string`. If a full rewrite is genuinely needed, write to a new file first, then `mv` atomically.

**Rule 3 — Refuse destructive `replace` calls (empty / too-short `old_string`):**

Before any `replace` / `Edit` whose `old_string` is empty (`""`) or shorter than 10 characters: REFUSE the operation. Emit:

```json
{"event":"safe_write_refuse","reason":"old_string_too_short_or_empty","file":"<path>","old_string_length":<N>}
```

This is the direct guard for whole-file overwrites caused by under-specified anchors.

**Rule 4 — Stray `.bak` cleanup at FIRST ACTION:**

In each section-shape skill's FIRST ACTION step (the `_progress.json` write described in §2), also glob `{output_path}/*.bak.*` and delete any files older than 1 hour. This bounds disk overhead from crashes that leave a `.bak` on disk.

**Relationship to §10.5 and §10.5.1.**
- §10.5 prevents bulk-rewrite cascades from `Bash` / `sed` / `python3`.
- §10.5.1 documents the (now-resolved) non-ASCII-truncation failure mode; no active mitigation needed from it anymore.
- §10.5.2 (this section) is the last line of defense — even if the agent uses the correct tool with the correct intent, the `.bak` snapshot survives a runtime malfunction.

---

#### 10.5.3 Tool-Output Retention (context-budget hard rule)

**Why this exists.** Running a command is a one-time cost; the expense is that its raw output then sits in the conversation and is **re-sent on every subsequent model round-trip** in the turn (and across turns until compaction). In an observed code excursion, a single tool-heavy turn (build + `npm test` + 14 edits) reached **6.9M input tokens** — not from re-reading across turns, but because verbose build/test logs and re-read source files rode along on every one of ~70 round-trips inside that one turn. The fix is not to run or log less. It is to **always execute fully, then retain only what the next round-trip needs.**

**The rule is about retention, never about execution. Always run the full command.** What you keep resident is decided by the output's *role*:

| Regime | Examples | Keep resident | Rationale |
|--------|----------|---------------|-----------|
| **Success** | build OK, tests green, lint clean, deploy healthy, migration applied | the **verdict only** — counts, exit code, the one summary line | A passing log carries no actionable detail; the body is pure noise. Full output is never needed. |
| **Failure** | failing test, compile error, probe 5xx, healthcheck down | the **failing slice** — the error, the stack, the failing names | The signal is the failure (usually <5% of the log). The passing lines before it are noise. Trim to the failure, not the whole run. |
| **Output *is* the evidence** | stack trace under root-cause, grep map being built, scanner findings, coverage report, diff under review | full fidelity **for this turn only**, then distill → drop | Sampling here can lose correctness, so capture full once, write the conclusion to the note/artifact, then drop the raw bytes — do not let them ride into the next round-trip. |

**The mechanism that satisfies "I sometimes need the full log" without paying to re-send it — redirect to a file, summarize into context, read on demand:**

```
npm test > .runlogs/test.txt 2>&1; tail -40 .runlogs/test.txt      # context gets the tail + counts
grep -A30 "FAIL src/work-orders" .runlogs/test.txt                  # full detail ONLY when a failure needs it
```

You never lose the full log — you stop re-billing it. Prefer the tool's own quiet/summary mode at the source where one exists (`jest` summary reporter, `--silent`, `pytest -q`, `gradle --quiet`, `npm --silent`), and on a run you expect to pass do **not** enable the runner's verbose / per-case-expanding mode (`--verbose`, `-vv`, `--reporter=spec`, etc.) — that prints a line per assertion and is the exact opposite of retention. Reach for verbose only after a failure, scoped to the failing target.

**Hard rules:**

- Never paste a large raw log (build/test/scan/probe) into the transcript when its outcome is success — emit the verdict line instead.
- On failure, emit only the failing portion; redirect the full log to a file and `grep`/`tail` it on demand.
- After any verification run, **distill the result (pass/fail counts, the exit verdict) into the working-set note or the artifact, then drop the raw output from working memory** — the same discipline §10.4.1 / §10.6 apply to re-read source.
- Do not re-read a file you just edited in the same turn to "re-orient" — reason from the edit result. (Observed: one service file read 4× in a single turn.)

**Enumeration guardrail (do NOT trim these away).** When the output is an enumeration-complete deliverable — a security scan's findings, a coverage-gap list, a toolchain inventory — every item matters and must not be sampled. The rule still applies, but "full fidelity" lands in the **artifact/output file**, not the chat transcript: write all findings to the file, keep only the count + the file path resident. Full is retained where it is the deliverable; it is simply not re-sent on every round-trip.

**Applies to:** any skill that runs build/test/lint/deploy/probe/scan commands inside a multi-round-trip turn. Cited by `conducting-excursion`, `implementing-code`, `fixing-bugs`, and the quick-lane `implementing-quick-change` (the quick-fix playlist runs `fixing-bugs`, so both quick lanes are covered); other tool-running skills (`bootstrapping-runtime-environment`, `launching-app`, `launching-subscription`, `tearing-down-services`, `managing-releases`, `researching-bug-fixing`, `reviewing-code`, `reviewing-code-web`, `api-smoke-probe`, `browser-smoke-probe`, and the security/enumeration skills with the guardrail above) adopt the §10.5.3 citation as they are next touched.

---


---

#### 10.5.4 Values you cannot compute — checksums and digests (mandatory)

**You cannot compute a SHA-256 in your head. Do not write one as if you had.**

Several artifact templates carry a `content_hash` — a sha256 over the artifact's
own content with the `content_hash:` line itself excluded — and a verifier
recomputes it. Stating that contract in prose and leaving the agent to satisfy
it does not produce a hash; it produces a plausible-looking 64-character hex
string that fails every check.

Field evidence (2026-09-06): across 117 hashed artifacts on one machine, two
project trees and three months, **35 verified — roughly one in three**. For
PRD-SPEC specifically it is 22 of 96.

The intermittency is the point, and it is worse than a clean failure. Some runs
get it right, presumably by shelling out; most do not, and nothing in the
artifact says which happened. A gate that fails three times in four for no
stated reason reads as flaky rather than as a contract nobody can satisfy by
hand.

On the run that surfaced it, the mismatch failed the automated gate, burned a
full repair lap re-running the producing step, failed identically, and escalated
to a human — while that repair lap had genuinely fixed two other findings with no
regressions. The value was recomputed on the second lap and was wrong again,
differently: it was never a hash of anything. No canonicalisation matched, and no
prefix of the document matched either.

**The rule.** Any field whose value is defined by a computation — a checksum, a
digest, a byte count, a line count — is produced by RUNNING the computation and
pasting its output. Never by estimating, and never by writing something of the
right shape.

**For `content_hash`, do exactly this, as the LAST write before the §11 sidecar:**

1. Write the artifact with the line `content_hash: ` followed by 64 zeros. The
   placeholder must be 64 hex characters or the verifier's own regex will not
   recognise the line, and it will report the hash as missing rather than wrong.
2. Run this, substituting your artifact's path:

   ```bash
   python3 - "$SPEC_FILE" <<'EOF'
   import hashlib, re, sys
   path = sys.argv[1]
   text = open(path, encoding='utf-8').read()
   m = re.search(r'^content_hash:\s*([0-9a-f]{64})\s*\n?', text, re.M)
   if not m:
       sys.exit('content_hash line missing or not 64 hex characters')
   canonical = text[:m.start()] + text[m.end():]
   digest = hashlib.sha256(canonical.encode('utf-8')).hexdigest()
   open(path, 'w', encoding='utf-8').write(
       text[:m.start()] + f'content_hash: {digest}\n' + text[m.end():]
   )
   print(digest)
   EOF
   ```

3. The digest it prints is the value now in the file. Report that value if your
   sidecar or audit asks for it — do not retype it from memory.

Removing the whole `content_hash:` line yields the same canonical text whatever
value sits on it, which is what makes this converge in one pass: the verifier
strips the line exactly as this does.

**This is the one sanctioned exception to the `python3`-mutation ban above.**
That ban exists to stop bulk section rewrites that bypass `Edit`'s prior-content
validation. This script rewrites a single header line whose correct value cannot
be known before the file is final, it runs once after every section is written,
and it is the only way the field can be right.

---

#### 10.5.5 Tool-Denial Adaptation (mandatory) — a denial is terminal for that exact command

**A tool/command-policy denial (`decision: deny`, an approval-required refusal, or
equivalent) is a terminal signal for the exact call that was denied — never a
transient failure to retry as-is.** The environment is telling you, deterministically,
that this literal command will not be allowed to run; issuing it again produces the
identical denial, not a different outcome.

Field evidence (2026-09-12, FairShare `local-runtime-validation`, `smoke-probing`
step): an HTTP health probe (`curl -s http://localhost:3000/health/ready`) was denied
by the project's command policy, which returned `suggestedAlternatives:
["Bash(pnpm:*)","Bash(bun:*)","Bash(bun run test:*)"]` in the same response. The agent
reissued the byte-identical denied command **266 times** until it hit `max_steps` —
**31.8M tokens** for zero forward progress, and the step ended `exec_failed` with no
outputs at all. The alternatives were returned and never read, let alone acted on.

The identical shape then recurred twice more in the same project, in two different
skills, on two different commands: `bootstrapping-runtime-environment` issued
`bun run prisma:generate` (124+ identical denials) and, separately,
`launching-app` issued `bun run build` (201 identical denials, manually
interrupted) — both denied for the same reason ("approval is required, but no
operator is present"), both returning the same `suggestedAlternatives`, neither
ever read. This is not three unrelated incidents; it is one recurring shape —
**a package.json script invoked via `<manager> run <script>`** — that any skill
shelling out in an npm-ecosystem project can hit, not a defect local to one skill.

**The rule:**

1. On any denial, do not reissue the same command (same binary, same arguments) a
   second time. One denial is information; a second identical one is not — it is
   pure waste, since the policy has not changed between your two calls.
2. Read the denial payload for `suggestedAlternatives` (or an equivalent hint) and
   for the project's command policy generally. If an allowed alternative can perform
   the SAME operation — a different program, a different invocation shape, a
   language runtime already on the allowlist — rewrite the call using it and
   continue the SAME plan; only the transport/mechanism changes, not the goal.
3. If no allowed alternative can perform the operation at all, that is a hard
   blocker, not a retry target: stop after that one adaptation attempt, record the
   blocker explicitly (open_questions / audit / tracker — never a silent
   zero-finding PASS), and signal failure (`stepwise session exec-fail` or the
   skill's equivalent) naming the missing capability. Do not spend the remaining
   step budget on further attempts of the same denied operation.

**The concrete recipe for the recurring `<manager> run <script>` shape** (this is
the single most common case rule 2 will need in practice): `<manager> run
<script-name>` asks the sandbox to resolve and execute an ARBITRARY named script
from `package.json` — an approval policy that pre-authorizes a known binary prefix
(`bun`, `bunx`, `pnpm`, `npx`) cannot pre-authorize an arbitrary script name, and
denies it on sight, every time, identically. Before running any package.json
script, or immediately on its first denial:

1. Read the script's definition from `package.json`'s `scripts` block.
2. If it is a SINGLE binary invocation (e.g. `"build": "tsc -p tsconfig.json"`,
   `"prisma:generate": "prisma generate"`) — call that binary directly through the
   allowlisted runner for the detected package manager: `bunx <binary> <args>` /
   `pnpm exec <binary> <args>` / `npx <binary> <args>` / `yarn dlx <binary> <args>`.
   This is not a guess or a workaround; it produces the identical program run,
   through a form the policy can actually authorize.
3. If it is COMPOUND (`&&`, `;`, `|`, multiple programs) there is no single-binary
   equivalent — do not invent one. This is a hard blocker per rule 3 above: record
   that the project's command policy has no path to run this specific script, and
   name the exact script. (A project can pre-approve a specific compound script,
   e.g. `Bash(bun run test:*)` in the evidence above — that is a policy decision
   for the project to make, not something a skill can route around.)

**Applies to:** every skill that can shell out or call a tool subject to approval or
command-policy gating — this is universal, not specific to one tool or one skill.
`api-smoke-probe`, `bootstrapping-runtime-environment`, and `launching-app` each
carry the incident's own Known Failure Modes entry as concrete instances that
produced or reinforced this rule.

---

## 11. Harness Output Sidecar

**Purpose.** The Stepwise harness reads each step's declared outputs from a sidecar JSON file written at a path the harness controls. If this file is missing, the orchestrator records `outputs: {}` in `state.json`, the UI's artifact panel renders empty, and downstream steps cannot pick up output values via propagation. **Artifacts on disk are not enough** — the sidecar is the framework contract between the skill and the orchestrator.

**Applicability.** Any skill that may be invoked by Stepwise. Standalone invocations (no orchestrator) do not need the sidecar — see the trigger gate below.

**When this section applies.** The input prompt contains a `## Run metadata` block with an `output_file = '<absolute path>'` line. If that block is absent (e.g. standalone invocation outside Stepwise), skip the sidecar write entirely.

---

### 11.1 Mechanism

```
1. Re-read the prompt's `## Run metadata` block. Copy the path on the
   `output_file = '...'` line VERBATIM. Do NOT reconstruct it from runId,
   session name, project name, or any other source — the path the harness
   wrote there is the path the harness expects to read from.

2. Re-read the prompt's `## Output parameters` table. For EVERY row in that
   table, include a key in the JSON object:
     - key   = the parameter name from the table
     - value = the resolved value the skill produced for that output
               (typically the path to the artifact folder or file)

3. Write that JSON object to the `output_file` path. This is the FINAL file
   write of the run — strict ordering (see Section 11.4):
     - AFTER any skill-defined output verification (e.g. file-existence checks).
     - AFTER Memory Bank session-end writes (Section 4).
     - AFTER `_progress.json` flipped to `status: COMPLETED` (Section 2 LAST ACTION).

4. Do NOT emit any tool call after the sidecar write. The next event in the
   transcript should be the `final_response` summary.
```

---

### 11.1.1 Path Correctness Rule (mandatory)

**First, resolving the path you were given — before any write.** A path parameter can arrive
as a bare fragment (`prd`, `test-cases`, `domain-boundaries`) rather than a rooted path,
because the capability's `value_template` passed the operator's value through without
composing it. A fragment is **relative to this run's output directory**, never to your
working directory:

```
GIVEN a path-typed input parameter P:
  IF P starts with "/" or "./" or "../"  -> use P as-is (already rooted)
  ELSE                                   -> P = <this run's output directory> + "/" + P
```

Your run's output directory is normally the prompt's "All the generated artifacts should be
placed in the output directory: ..." line, or the `output_folder` parameter. `mkdir`-ing a
bare fragment creates a stray folder at the repository root, outside `artifacts/outputs/`,
where no downstream step or gate will look for it.

**If the prompt gives you no rooted anchor, do not guess.** A capability whose instruction
renders that line from the same uncomposed parameter — `output directory: prd` — and which
omits `output_folder` from its parameter table has left you nothing to resolve against.
Silently falling back to the working directory is what produces the stray folder. Treat it as
a capability defect: proceed with the value you were given, and raise it as a blocking open
question naming the parameter and what you assumed, so the wiring gets fixed rather than the
symptom recurring every run.

**Symptom.** Your outputs land next to the project's top-level folders instead of under
`artifacts/outputs/<capability>/`. Observed 2026-09-06 (`pli-20260906-qxlefw`): a playlist set
`prd_output_path: "prd"` — consistent with every other capability in that playlist, which
compose fragments in their `value_template` — the producing capability passed it through
unchanged, and the step created `<project>/prd/` while the prompt's stated output directory
was `<project>/artifacts/outputs/product-definition`. The verification gate then failed
against the folder nobody wrote to.

Report the composed path per the rule below; the parameter you were handed is the input, not
the answer.

---

The value reported for any output **path** parameter MUST be the **actual on-disk path the skill wrote
to** — never a re-derived input parameter, never a fresh `value_template` substitution, and never a
scope-/sibling-adjusted path that was NOT the one actually used for the write.

```
BEFORE writing the sidecar, for every path-typed output parameter:
  1. Confirm the artifact exists at the path you are about to report
     (e.g. `ls <PATH>/<EXPECTED-FILE>*` or a stat/existence check).
  2. If the path the skill actually wrote to differs from the input parameter
     for ANY reason — sibling-folder rule, scope adjustment, feature_id stripping,
     SESSION_ID derivation — report the ACTUAL written path, NOT the parameter.
  3. Reporting a parameter you did not write to is the documented cause of the
     duplicate-artifact-on-REPAIR defect: a later REPAIR run is handed the wrong
     folder, finds no prior artifact, and silently rebuilds a duplicate. (See §7.)
```

**Worked example (sibling rule).** A planning step receives `planning_output_path =
/base/code-development/code-task-planning` but, because `feature_id` was empty and the upstream research
was scoped, Step 1 actually wrote to `/base/code-development/TICKET-001/code-task-planning/`. The sidecar
MUST report the latter (the path that holds the files), never the flat input parameter.

> **Note for skill authors:** "report the parameter VERBATIM" guidance (used by some skills to avoid
> pre-resolution double-prefixing) means *do not re-prepend project/scope segments yourself* — it does
> NOT license reporting a path you did not write to. When your own logic diverges from the parameter,
> the actual written path always wins.

---

### 11.1.2 Verdict Fidelity Rule (mandatory)

§11.1.1 governs path-typed outputs. This rule governs **closed-set outputs** — any output parameter
that declares an enumerated `values:` array, which in practice means every gate / verdict-gated-transition
(VGT) verdict the orchestrator routes on.

The value reported for such a parameter MUST be **one of the declared tokens, copied character-for-character**.

```
BEFORE writing the sidecar, for every output parameter with a declared `values:` set:
  1. Re-read the allowed tokens from the prompt's `## Output parameters` table
     (or, when the table omits them, from this skill's own verdict definition).
  2. Emit one of those tokens EXACTLY: same spelling, same case, same underscores.
     No synonym, no title-casing, no pluralisation, no added qualifier, no
     translation, no "PASSED (with notes)".
  3. Confirm the emitted token is a member of the declared set — a literal string
     comparison, not "close enough". If your computed verdict does not map onto a
     declared token, that is a bug in your mapping: pick the correct declared token,
     do not invent one.
  4. Confirm the token AGREES with the artifact you just wrote. When the report /
     spec carries the verdict in its own front-matter or summary, the two MUST be
     identical. A report that says PASSED and a sidecar that says something else is
     a self-contradiction, and the sidecar is what the orchestrator acts on.
```

**Why this is not cosmetic.** The orchestrator matches the reported token against the step's
`gate_policy` accept/reject lists. A token in neither list is **unroutable**, so the engine
**fails closed and treats it as a reject** — the only safe default, since silently passing an
unrecognised verdict through a correctness gate would be worse. The consequence is a spurious
retry loop: the upstream producer step is re-run, every step between it and the gate is reset and
re-run with it, and the retry budget is consumed. When the budget is exhausted the run escalates
to a human gate carrying a rejection that never happened.

**Worked example (observed).** A plan-verification gate declared
`values: [PASSED, PASSED_WITH_FINDINGS, FAILED]`. The skill wrote a report whose front-matter read
`verification_verdict: PASSED`, `weighted_total: 100`, `failed_blocker_count: 0` — a clean pass — but
emitted `"verification_verdict": "APPROVED"` in the sidecar. `APPROVED` is not in the set, so the gate
failed closed, rejected, and re-ran planning plus the two steps between. A perfect-scoring plan was
re-planned because the sidecar used a synonym.

**Robustness note.** This write is the last action of a run, often after a long generation, and may be
performed under a degraded executor — a fallback model, a smaller tier after a quota or rate-limit
failure, or a resumed session. Those are exactly the conditions under which a token gets paraphrased.
Treat the closed-set comparison in step 3 as a real check to perform at write time, not a stylistic
preference — it is cheap, and it is the only thing standing between a correct verdict and a wasted lap.

---

### 11.2 Example

For a skill that declares a single output parameter named `foo_output_path`:

```json
{ "foo_output_path": "/abs/path/to/artifacts/outputs/<capability>/<step>" }
```

For a skill with multiple output parameters, include every row from the `## Output parameters` table — one key per row. The skill-specific parameter names live in each SKILL.md (Section 11.5); this section does not enumerate them.

---

### 11.3 Violation

Ending a Stepwise run with a `final_response` summary that lists the generated artifacts but no sidecar write breaks output propagation. The harness MAY have a `value_template` fallback for specific outputs, but that fallback is not guaranteed for every skill / every output. Treat the sidecar write as **non-optional** whenever the `## Run metadata` block is present in the prompt — do not rely on the fallback.

Skills whose outputs lack a `value_template` fallback fail **silently** when the sidecar is missing: downstream steps see `outputs: {}` and propagate empty values; no error surfaces in the run. This is the failure mode this section prevents.

---

### 11.4 Composition with Section 2 LAST ACTION

The sidecar write extends Section 2's LAST ACTION; it does NOT replace it. Strict ordering for any Stepwise-invoked skill:

1. All skill-specific outputs written and verified.
2. Memory Bank writes (Section 4 — `active-context.md`, `progress.md`).
3. `_progress.json` flipped to `status: COMPLETED` (Section 2 LAST ACTION).
4. **Sidecar write** (this section) — final file write of the run.
5. `final_response` summary — no further tool calls.

If the skill is invoked standalone (no `## Run metadata` block), step 4 is skipped. Steps 1–3 and 5 still apply.

---

### 11.5 Skill-Level Enforcement

Section 11 cannot enumerate output parameter names — those are declared per-skill in the capability YAML's `outputs` block and surface in each prompt's `## Output parameters` table. Each SKILL.md MUST include a short inline callout naming its output parameters, so the agent knows which keys to put in the sidecar JSON without parsing the YAML.

Recommended SKILL.md callout shape:

> **Step N: Emit harness outputs sidecar — apply execution-protocol.md Section 11.**
>
> **Output parameters this skill produces** (one key per row of the prompt's `## Output parameters` table):
>
> - `<param_name_1>`: <one-line description of the resolved value, e.g. "the resolved output folder path written in earlier steps">
> - `<param_name_2>`: ...
>
> Example sidecar contents (illustrative):
>
> ```json
> { "<param_name_1>": "/abs/path/...", "<param_name_2>": "..." }
> ```
>
> **The sidecar ALSO carries the reserved `_open_questions` key — see §11.6.** It is not one
> of this skill's declared outputs — Stepwise itself renders it as a required row of the
> `## Output parameters` table — so it is not listed above; that is why the example shows
> only declared keys. Write it in every sidecar: every open question THIS run raised, `[]`
> when it raised nothing. Name the registries this skill writes (e.g. "the standard
> `pending_inputs` / `traceability_gaps` / `assumptions_to_validate` tables") and stop
> there — §11.6 owns the field set and the mapping.

The callout is intentionally short. Trigger gate, mechanism, ordering, and violation rule all live in Section 11 — the SKILL.md only names the keys.

**A worked example outranks a rule the reader has to recall.** A callout whose example shows
only declared parameters is read as the complete sidecar contract for that skill, and it will
beat §11.6 at the moment of writing — observed 2026-09-05, where a skill with a
fourteen-row open-questions registry emitted a one-key sidecar and the orchestrator had to
scrape the artifact instead. That is why the paragraph above belongs in the callout rather
than being left implicit.

---


### 11.6 Open Questions in the sidecar (universal)

**Trigger.** The prompt has a `## Run metadata` block (§11 trigger gate). That is the only condition: the key is written on EVERY sidecar, whether or not this run raised a question.

**Rule.** Write the reserved `_open_questions` key in the same sidecar JSON as your declared outputs — ALWAYS. If this run wrote an `## Open Questions` / `## open_questions` section (any heading level, any spelling) into an artifact, mirror that section into it; if this run raised nothing, write `[]`. This is not extra analysis: you already decided what the open questions are and wrote them down. This emits the same list in a form the orchestrator can act on, and the empty array is how you tell it, unambiguously, that there was nothing to track.

**Only what THIS run raised — never what the prompt handed you, and never what the project has already settled.** If a question appears in your prompt's `## OPEN QUESTIONS`, `## DECISIONS` or `## PROJECT DECISIONS` block, it is already tracked. The same holds for a decision you meet anywhere else you read — a value tagged `[OQ-xxxxxxxx]` in an upstream artifact, a row of `context-pack/decisions.md`, or the count in the `## PROJECT DECISIONS` pointer sentence ("N further settled decisions are not repeated here"): the block carries only what nothing else in your inputs holds yet, so most settled decisions reach you through the documents, not the block. A settled decision is not a question. Do not put it in your artifact's open-questions section, and do not declare it in `_open_questions` — not under your own numbering, not re-worded in your own voice, not "for completeness", and not as an assumption that reads "…, per settled decision OQ-xxxxxxxx" (observed 2026-09-09, `backlog-story-generation`: four such rows, each re-asking a decision it had already applied, each answered with the very value it cited). Where your work depends on one, cite its id in the sentence that proceeds on it:

```
- EPIC-01 asume el claim `smart.legajo` como fuente de identidad del analista (OQ-bd39279d).
```

`OQ-xxxxxxxx` (eight hex characters) is Stepwise's ledger id, and citing it is how an
artifact shows which decision or open question a sentence rests on. It is a cross-reference,
not one of the artifact's own ids: a verifier that checks "every id uses an allowed prefix"
must treat `OQ-[0-9a-f]{8}` as always allowed, or it fails the exact behaviour this
section requires (observed 2026-09-08: CHK-008 on a PRD flagged `OQ-3551076d` as a
disallowed id and rejected a correctly cited document).

**Why this is a hard rule and not a preference.** Identity is a hash of the question text, so a re-worded copy does not match the original — it opens a SECOND row that no automatic merge can close, and the operator has to recognise and fold it by hand. Measured on a live run: a step was handed 33 inherited questions and 17 decisions in its prompt, re-listed them in its own artifact under its own numbering, mirrored all of it into `_open_questions`, and turned 74 questions into 124. Thirty-seven were the same question asked twice.

This applies to the SIDECAR as much as to the artifact section. The earlier form of this rule addressed only the prose, on the assumption that the sidecar mirrors the prose — so a skill that writes the inherited questions into its section complies with the mirror rule and breaks the ledger anyway. Both halves are the same rule: **the open-questions registry of an artifact, and its sidecar, describe what this step discovered.**

`_open_questions` is a RESERVED row. Stepwise itself renders it into the `## Output parameters` table of every step prompt, marked required; it is never one of the skill's declared outputs, and Stepwise strips it before output resolution, so writing it never breaks propagation. An absent key is a contract violation that costs one repair turn — write `[]` rather than leaving it out. Stepwise validates entries at the boundary: a `question` without `assumption_if_unanswered` is accepted but flagged "needs a basis" and can never be bulk-accepted; an `answer_shape: enum` without `options` is downgraded to free text.

```json
{
  "prd_output_path": "/abs/path/to/artifacts/outputs/product-definition/prd",
  "_open_questions": [
    {
      "id": "PI-10",
      "question": "Which inventory slice does the pilot run on?",
      "subject": "scope",
      "why_it_matters": "Every downstream sizing estimate assumes one slice",
      "blocking": true,
      "assumption_if_unanswered": "one DTC channel with real sell-out visibility",
      "kind": "question",
      "answer_shape": "enum",
      "options": [
        { "id": "dtc", "label": "One DTC channel with real sell-out visibility" },
        { "id": "wholesale", "label": "The wholesale slice" },
        { "id": "all", "label": "Every channel" }
      ],
      "audience": "client",
      "artifact_ref": "artifacts/outputs/product-definition/PRD-SPEC-....md#open-questions",
      "traces_to": ["EPIC-02", "FR-14"]
    }
  ]
}
```

When this run raised nothing:

```json
{ "prd_output_path": "/abs/path/to/artifacts/outputs/product-definition/prd", "_open_questions": [] }
```

**Fields.** `question`, `audience`, `artifact_ref`, `subject` and — on a `question` — `assumption_if_unanswered` are required. Emit every other field you actually have; omit what you do not. Never invent one — Zero-Invention applies here exactly as it does to the artifact.

`artifact_ref` is required because the trigger already told you the answer: you are mirroring a section you just wrote INTO an artifact, so you know which artifact it is. There is nothing to look up and nothing to invent.

- `id` — your own local id (`PI-10`, `OQ-03`) if the section has one.
- `kind` — `question` (default) or `assumption`. See the note below; getting this right decides which verbs the operator is offered. **Every row of an `assumptions_to_validate` table is an `assumption` entry**, whether or not it carries a fallback and whether or not it is blocking: an assumption with a fallback is still a choice the work made on the operator's behalf, and one that stays only in the artifact is a decision nobody was offered. Observed 2026-09-08 (`researching-code-design`): the backend runtime was chosen in an `ASM-01` row, the sidecar said `[]`, and nothing reached the ledger — implementation would have proceeded on Node.js because one table was read as "not questions".
- `subject` — **REQUIRED.** What the question is ABOUT (`scope`, `compliance`, `personas`, `observability`). It turns a 60-row inbox into a dozen themes — and it is also how a settled decision finds its way into the context pack: Stepwise routes a decision to the topic file whose `when_to_read` its subject overlaps. A decision with no subject can never be routed, so it is re-offered on every reconcile forever and never folded into the pack. Measured on one project: 30 of 126 decisions carried no subject and were permanently unharvestable, from four steps that emitted everything else correctly. Use the noun the registry row is already about; do not invent a taxonomy.
- `why_it_matters` — the consequence of leaving it open.
- `blocking` — `true` ONLY if downstream work genuinely cannot proceed. See the warning below.
- `assumption_if_unanswered` — **REQUIRED on a `question`.** What you actually proceeded on. See **Say what you proceeded on** below. (Omit it on an `assumption`: the claim IS what the work proceeded on.)
- `audience` — **REQUIRED.** Who OWNS the answer: `client`, `architect`, `product_owner`, `internal`. See **Every question names an owner** below.
- `answer_shape` — `free_text` (default), `enum`, `path`, `number`, `boolean`. Prefer `enum` with 2–5 `options` whenever the answer is a choice: it gives the operator one-click buttons instead of a text box.
- `options` — required when `answer_shape` is `enum`: an array of `{ "id", "label" }` objects (or plain strings). Stepwise downgrades an `enum` without options to free text, so an enum with no options buys nothing.

  **Options carry the trade-off, not just the choice.** A label naming only the option ("Standard" / "High-Performance") makes the operator go read the artifact to find out what either one costs; a label naming the concrete consequence lets them decide from the button alone. Compare "Standard" against "Standard (< 2s response, lower infra cost)" — same choice, only one is answerable without leaving the inbox. Two further rules apply to every `enum` row: options MUST be mutually exclusive (an operator who could reasonably pick two is being offered one question disguised as two) and MUST cover the reasonable range of answers a downstream artifact would actually be built differently against — not padded with an option nobody would choose to reach the 2–5 count.
- `artifact_ref` — **REQUIRED. Project-relative** path (optionally `#anchor`) of the artifact that raised it, with `/` separators — `artifacts/outputs/product-definition/PRD-SPEC-20260812.md#open-questions`. NOT an absolute path: the ledger is committed to the project repo and opened on other people's machines, and an absolute path from yours is unusable on theirs. This field is what the operator's "Edit the artifact" button resolves, and on one measured project 997 references named a machine nobody else had (Stepwise issue #1047). Same rule the harness applies to every path it persists.

  **Omitting it is not a smaller sidecar, it is an unrepairable question.** The orchestrator computes which documents an answer invalidated from these refs; a question with none contributes no document, so when the operator answers it there is nothing to point a repair at and no "Edit the artifact" to offer. Measured on one run: `prd-creation` emitted 75 refs and every one of its questions was repairable, while `backlog-epic-generation` (58) and `discovering-bounded-contexts` (13) emitted none at all — 71 questions that could be asked and answered but never applied. Emit it on every row.
- `traces_to` — ids the question affects (`EPIC-02`, `FR-14`, `BC-09`).

**Enumerate the registries from YOUR artifact, never from a remembered list.** Agent-native
spec skills put several tables under `open_questions`, and the set differs per skill:
`pending_inputs`, `coverage_gaps`, `alignment_gaps`, `traceability_gaps`, `missing_inputs`,
`assumptions_to_validate`. Before
building the sidecar, **read the `###` headings under your own `open_questions` section and
mirror every row of every one of them** (except `summary`, which is counts, not questions).
A registry you did not mirror is a partial list, and §11.6's completeness rule treats a
partial list as "the rest were resolved".

Observed 2026-09-06 (`backlog-epic-generation`, `pli-20260905-1jcnazb`): a skill was told its
registries were `pending_inputs` / `traceability_gaps` / `assumptions_to_validate`, mirrored
exactly those, and silently dropped `AG-01` from the `alignment_gaps` table it also had. The
orchestrator recovered it by scraping the markdown, where it landed as a row that cannot hold
a step. Naming a list is a hint; the artifact on disk is the authority.

**Carried-forward registries are retired** (`prd_gaps_carried_forward`,
`upstream_gaps_carried_forward` and any `*_gaps_carried_forward` table). The prompt's
`## OPEN QUESTIONS` / `## PROJECT DECISIONS` blocks ARE the carry-forward: Stepwise delivers
every inherited question and every settled decision there, with its id. Never rebuild one in
the artifact and never mirror one into the sidecar; where your work relies on an inherited
item, cite its id in the body instead.

The columns below are the shape used by `pending_inputs` and its close relatives. **Do not
assume any other registry has them** — gap tables frequently do not, and
a missing column is never a licence to emit `null`. Map what your table actually has:

| sidecar field | registry column |
|---|---|
| `id` | `id` (`PI-XX`, `TG-XX`, `ASM-XX`) |
| `question` | `question` on PI/TG rows; the `assumption` statement on ASM rows |
| `kind` | `question` for `pending_inputs` and `traceability_gaps`; `assumption` for `assumptions_to_validate` |
| `subject` | `section` on PI rows; `traceability` for TG rows; for ASM rows the section the assumption belongs to |
| `why_it_matters` | the CONSEQUENCE, in a sentence — what breaks or degrades, named against the `field` / `item_id` it holds up; `impact_if_wrong` on ASM rows. **The bare impact level is not an answer:** `HIGH` is a rejected value, `HIGH — FR-05 cannot be enabled in production until the window is fixed` is the field. Restate the row; do not invent a new consequence |
| `blocking` | **re-decided, never copied** — see below. The registry column and this field mean different things |
| `assumption_if_unanswered` | `fallback_behavior`, subject to the BLOCKED rule below. Omit on `kind: assumption` |
| `traces_to` | `affected_ids` |
| `artifact_ref` | the artifact's project-relative path + `#open-questions` |
| `audience` | not a column — decide it per row, see **Every question names an owner** |
| `inherited_from` | on a repair/iterate lap, the ids from the prompt's `## OPEN QUESTIONS` / `## DECISIONS` / `## PROJECT DECISIONS` blocks this row restates |

**The `BLOCKED` collision.** These templates conventionally set
`fallback_behavior: BLOCKED` on every `blocking: yes` row, and the rule below forbids refusal
tokens in `assumption_if_unanswered`. So where `fallback_behavior` is `BLOCKED`, do NOT copy
it — and do NOT reach for a generic stand-in either. Name the narrowest thing THIS artifact
actually did about THIS item, in terms of its own ids:

> `FR-01 is emitted with status: pending and no acceptance criterion for legajo extraction`
> `the existing endpoint contract is preserved unchanged; no new field was added`
> `NFR-04 keeps the source's 19:00-06:00 value and the 23:00 variant is not modelled`

Then set `blocking: true` with the reason in `why_it_matters`. A run that shipped an artifact
proceeded on something; name it specifically.

**One sentence per row — reused text is a defect.** If two rows in the same sidecar share an
`assumption_if_unanswered` string, at least one of them is wrong: two different unknowns
cannot have been proceeded on in the same way. Before emitting, check that the values are
distinct and that each names its own ids. A row is finished when a reader who has not seen
the artifact could tell, from that field alone, what the document says about that item today.

Observed 2026-09-05 (`product-discovery__pli-20260905-1jcnazb__pd-001`): ten of eleven
blocking rows carried the identical sentence "the field is left pending and no value was
assumed" — an earlier revision of this section offered it as an example and it was copied
verbatim. That is not a cosmetic problem. When the operator bulk-accepts a row, the ledger
records `assumption_if_unanswered` AS THE ANSWER (`answered_by: operator`,
`note: "Bulk-accepted by the operator"`). Ten questions were therefore settled with an
answer of record that says nothing, and no future answer can be compared against them.

**Registries without those columns — derive, do not blank.** Gap tables
(`coverage_gaps`, `alignment_gaps` and similar) typically carry only something like `id | type | ref_id | description |
affected_section` or `id | origin | description | impact_on_boundaries`. Every sidecar field
is still derivable, and every one is still required:

| sidecar field | where it comes from when the standard column is absent |
|---|---|
| `id` | the table's OWN id column — `GAP-01`, `AG-01`, `ASM-01`. Never drop it: an entry with no id cannot be referenced, folded, or answered against |
| `question` | the `description` / `assumption` text. A statement of absence ("hosting platform is not identified") is a valid question — keep the wording, do not invent a new one |
| `kind` | `assumption` for any assumptions table, `question` for everything else |
| `subject` | `affected_section`, `section`, or the registry's own name |
| `why_it_matters` | `impact_on_boundaries`, `impact_if_wrong`, `rationale`, or the consequence implied by `type` (e.g. `deferred_by_v3.0_scope` → what is therefore unplanned) |
| `traces_to` | `ref_id`, `affected_ids`, or the ids named in the description |
| `blocking` | decided per the rule below; these rows are usually `false` |
| `assumption_if_unanswered` | **SYNTHESIZE it.** These tables have no `fallback_behavior` column, and the field is still required on every `question`. State what the artifact did in its absence — "no hosting provider is assigned to any service", "the dependency is recorded with owner unassigned". Omitting it is the one thing you may not do |
| `artifact_ref` | your own artifact's project-relative path + `#open-questions` |

**An assumption already confirmed is a decision, not an open question.** If an
`assumptions_to_validate` row carries `confidence: confirmed`, `validation_needed: none`, or
a rationale saying the point is settled, it does not belong in `_open_questions` — it will
sit in the operator's inbox asking them to decide something the artifact says is already
decided. Emit only assumptions that genuinely still need validating.

Observed 2026-09-05 (`discovering-bounded-contexts`, `pli-20260905-1jcnazb`): all five
registries were correctly mirrored, and the five rows from the four non-`pending_inputs`
tables arrived with `id`, `kind`, `subject`, `why_it_matters`, `traces_to` and `artifact_ref`
all null — `GAP-01`, `AG-01`, `UG-01`, `UG-02` lost their ids entirely, two rows carried no
`assumption_if_unanswered` at all, two more shared one string, and a `confidence: confirmed`
assumption was raised as an open question. Mirroring the table is half the job; mapping its
columns is the other half.

**A mostly-blocking sidecar means you copied the column.** In the registry table
`blocking: yes` means "this document is not complete"; in the sidecar it means "downstream
steps must not run," which stops a playlist and needs an operator to confirm. Those diverge
in practice: most rows whose `fallback_behavior` names a real behavior the artifact proceeded
on are `blocking: false` here, however the table marks them. So treat a high blocking rate as
a self-check, not a result — if most of your rows come out `true`, re-read the column
semantics rather than shipping it. Observed 2026-09-05: eleven of fourteen rows arrived
`blocking: true`, `blocking_asserted_by: sidecar`, and every one was waived by the operator
in a single bulk action.

**`audience` is decided per row, not defaulted per artifact.** The test is who is ENTITLED to
decide, and for a technical artifact that is frequently NOT the client. A tooling choice
("Flyway or Liquibase for the versioned DDL?"), a policy the delivery team owns ("does
architecture accept the absence of backend authorization?"), a contract shape your own
downstream architecture step will settle — those are `architect`. Scope, priority and
what "done" means are `product_owner`. `client` is for facts about their business, their
data, their policy or their commercial choice; it is the only value that sends a question out
of the building, and over-using it stalls questions your own next step could answer. A useful
check: if the question is one your own architecture or delivery capability will have to
answer anyway, it was never `client`.

A skill whose registries differ from this shape names its own mapping in its §11.5 callout.
A skill whose registries match it needs no instruction beyond naming them.
- `inherited_from` — the ids of questions this one restates, copied verbatim from the `## OPEN QUESTIONS` / `## DECISIONS` / `## PROJECT DECISIONS` blocks (`[OQ-1d0efb0c]` → `"OQ-1d0efb0c"`). This is what folds a restatement onto the row already tracking it instead of opening a second one. See **Your `## Open Questions` section lists what THIS run raised** below.

**`kind` separates a gap from a claim.** A `question` names something UNKNOWN — the operator resolves it by supplying the value, or by accepting the assumption you proceeded on. An `assumption` names something you ASSERTED and could not verify ("Theobald is compatible with the SAP S/4HANA landscape", "Fabric is licensable in Germany West Central") — the operator resolves it by confirming it, which changes nothing because the artifact already says it, or by correcting it, which invalidates everything built on it. Emit `assumption` for anything you would have written into an `assumptions_to_validate` table. A question phrased as a statement of absence ("No client-approved RPO exists") is still a `question`: it needs a value supplied, not a claim confirmed. For an `assumption` you may omit `assumption_if_unanswered` — the claim IS what the work proceeded on.

**`blocking` means the SCHEDULE, not the document.** Registry tables use `blocking: yes` to mean "this document cannot be considered complete." Here it means "downstream steps must not run until this is answered," and an operator has to confirm before it stops anything. If work can proceed on a stated assumption, it is `false` with an `assumption_if_unanswered` — that is the normal case.

**Completeness.** The sidecar is complete by contract: list EVERY open question this run is raising, or write `[]`. Do not emit a partial list — the orchestrator reads an unlisted question as one you resolved and stops tracking it — and do not omit the key: an absent key is not "nothing raised", it is a missing required output.

**Say what you proceeded on.** `assumption_if_unanswered` is required on every
`question`, and it is the field that decides what answering COSTS.

With it, an answer that agrees with what you assumed settles the question and
touches nothing. Without it, there is no basis to compare an answer against, so
EVERY answer must be treated as invalidating — and each one puts every artifact
built on the gap into the operator's repair queue, including the answers that
merely confirm what the document already said.

Field evidence (2026-08-27): eleven open rows on one ledger, none with an
assumption. Five of them re-asked something an earlier lap had already raised
WITH an assumption and the operator had already settled. One — a tie-break rule
for identical values — existed five times, had been answered twice by the
client, and came back a fifth time with the assumption dropped, so the
established answer could not be matched against it.

If you genuinely could not proceed, that is not a reason to omit the field. Say
what you did instead — "the row is emitted with the field absent and flagged" —
because that IS what you proceeded on. Do not write a refusal token (`BLOCKED`,
`TBD`, `N/A`, `unknown`) into it: that says you stopped, not what you assumed,
and it renders downstream as "proceed on: BLOCKED", which is worse than leaving
the question open. If work truly cannot continue, set `blocking: true` and put
the reason in `why_it_matters`.

**Deferring to a later phase still names an interim baseline.** Registering a
gap against a future phase ("Ideation", "Discovery", "Foundation") instead of
answering it now is a legitimate outcome when the information genuinely is not
available yet — but the deferral itself is not what you proceeded on; the
phase name alone tells a downstream agent nothing to build against today. The
preferred pattern is `Deferred to <phase>; interim baseline: <the concrete
state a downstream agent can build against right now>` — e.g. "Deferred to
Discovery; interim baseline: scope excluded from Phase 0" or "Deferred to
Foundation; interim baseline: proceed with the industry-standard 99.9% SLA
target." This is the same rule as above, phrased for the deferral case: the
prohibited value is not `BLOCKED` alone, it is any deferral with no baseline —
"Deferred to Discovery" and nothing else is a refusal token wearing a phase
name.

**Every question names an owner.** `audience` is required, and it is a decision
rather than a formality: it separates a technical call the delivery team makes
from a domain question only the client can settle. Standard delivery asks the
client early and waits — an engagement cannot do that if nobody said which
questions were theirs.

Choose by who is ENTITLED to decide, not who is likely to reply:

- `client` — a fact about their business, their data, their policy, their
  commercial choice. The only value that sends a question out of the building.
- `product_owner` — scope, priority, what "done" means for this engagement.
- `architect` — a technical decision with real trade-offs, ours to make.
- `internal` — anything the delivery team settles among itself.

**Never use `internal` as a default.** It is what a record gets when the field is
absent, so an omitted `audience` is indistinguishable from a considered "we will
handle this" — and it means "do not send it out". Field evidence (2026-08-27):
across two live ledgers, 318 of 353 rows and 85 of 85 were `internal`, and
almost every one was a skill that simply never declared it. On the second
project the operator's client-questions list was empty while the client owned a
dozen of them, so the questions that most needed asking were the ones nobody
could see.

When you genuinely cannot tell, `product_owner` is the right hedge: it puts the
row in front of a human who can route it instead of burying it.

**A template field is not a question.** An open question is a DECISION the project
needs that the inputs did not supply. It is not the absence of a value in a template
column. Agent-native templates carry descriptive fields on every item — `usage_frequency`
on a persona, `measurement_method` on a KPI, `owner` on a dependency, `likelihood` and
`impact` on a risk, `complexity` on an epic — and a brief rarely speaks to any of them.
Registering each empty one as a pending input fills the operator's inbox with rows nobody
will answer and buries the handful of real unknowns among them.

Field evidence (2026-09-08, `fs-training` / `product-definition`): 59 rows from one PRD,
51 of them shaped `<section>/<item>: what is the value of \`<field>\`? (missing)`;
seventeen shared five identical assumptions ("No usage frequency assumed" on six personas,
"No owner assumed" on seven dependencies, …); 33 were tagged `client`, asking the customer
for the value of a template column. The genuine unknowns — thresholds, award amounts, a
points formula — were eight rows out of fifty-nine, and the prompt budget dropped 41 rows
before the next step saw them.

The rules, in order:

1. **The decision test.** Raise a question only when a downstream artifact would be BUILT
   DIFFERENTLY depending on the answer AND the inputs do not contain it — acceptance
   criteria of an MVP requirement, a threshold an NFR commits to, an amount, a formula, a
   provider. A descriptive field the inputs never mention is written as `not stated in
   inputs` in the artifact and the item keeps its status; it is not a pending input, and
   it is not an assumption either.
2. **One question per decision, never one per field.** Several items missing the SAME
   descriptive field is at most ONE question ("Do the personas need a usage frequency, and
   who supplies it?") — and usually none. Two rows with the same `assumption_if_unanswered`
   are already a defect (above); a table where a dozen rows share one is this rule being
   broken.
3. **Phrase it for the owner.** "What is the value of `tier_thresholds`?" is a template
   speaking. "Which Relationship Value boundaries define the three tiers?" is a question a
   product owner can answer. If you cannot rephrase a row away from the column name, it
   was not a question.
4. **A missing template field is never `client`.** The client owns facts about their
   business; whether a template column is filled is a delivery-team matter — `product_owner`
   at most, and only when rule 1 passes.
5. **Self-check before writing the registry.** If more than a third of your items would get
   a pending-input row, or the rows would exceed ~15, you are registering template
   completeness, not project unknowns: re-apply rule 1 and collapse under rule 2. The
   artifact's `status` may still read `draft`; the inbox is not where completeness is
   tracked.

**A question that does not name its own consequence is not a question yet.**
Passing the decision test (rule 1) is necessary but not sufficient — a row can
be about a genuine decision and still fail to earn its place in the registry
if the wording does not show WHY it is decision-bearing. Before writing a row,
apply this screen:
- **Names the downstream artifact it changes.** `why_it_matters` states, in
  terms of the artifact's own ids, which FR/NFR/epic/story would be worded or
  valued differently on each answer — not the bare impact level ("HIGH" is not
  an answer; "HIGH — FR-05 cannot be enabled in production until the window is
  fixed" is). A row whose consequence you cannot state this specifically has
  not passed rule 1, it has only been asserted to.
- **Does not presuppose an answer.** A question phrased so that only one
  answer reads as correct is not gathering information, it is asking for
  confirmation of a decision already made — if you already know the answer,
  it is a `decisions_confirmed` entry or an `assumption`, not a `question`.
- **Does not bundle independent decisions into one row.** "What is the SLA
  target and who owns the escalation path?" is two decisions wearing one
  question mark; a reader can answer one half and leave the other silently
  unanswered. Split it, or — if only one half is actually decision-bearing —
  drop the other half per rule 1.
- **Is answerable by its named `audience` without reading the artifact.** If
  answering the question requires the recipient to first go read the section
  it came from to understand what is being asked, phrase it in the recipient's
  terms instead of the artifact's internal vocabulary (see rule 3 above).

Traceability gaps and assumptions-to-validate are unaffected: an orphan requirement and
an unverified claim are real, and each is one row.

**A status line is not a question.** `_open_questions` carries questions, not a
report about them. "None newly raised", "no new gaps identified", a count of
what the section holds, or a note that inherited questions are carried forward
unchanged — none of these is something an operator can answer, and each one
lands in the inbox as work. Say it in the artifact's prose if it is worth
saying; emit `[]` here if this run raised nothing.
Field evidence: "None newly raised. PLAN-SPEC's existing open questions are
inherited unchanged" reached a live inbox from a `code-review` sidecar.

**This includes reporting that you followed the rule above.** Having correctly
declined to republish the questions you inherited, do NOT then write a line in
the section — or a `note:`-prefixed entry in the sidecar — saying that you
declined. An empty section, or no section at all, already says it. Field
evidence (2026-09-05, the day the no-republishing rule shipped): a verification
step emitted `note: No new questions were raised by this verification run;
inherited product questions were not republished.` It had done exactly the right
thing and then filed a status line about it, which the operator had to read and
discard. The correct sidecar for that run was `"_open_questions": []`. If a reader genuinely needs to know the run examined the inherited
questions and found no new gap, that belongs in the report's prose, not in a
register of things somebody has to answer.

**A harness failure is not an open question.** A pinned path that does not
resolve, an `output_spec_source` absent on disk, a caller-supplied path that
arrives malformed, an alias that did not resolve against the skills root
(§16.4), a directory you could not write to — these are run bookkeeping, the
raising-side mirror of "Process metadata is never Blocking" in the reviewer's
verdict duty. They are not claims about the artifact's subject, and no operator
can answer them: "What is the valid absolute path to the ADR collection?" has no
answer in the project's domain, only in the wiring that produced the wrong path.

Emitting one anyway does lasting damage, because the ledger cannot tell it apart
from a real question. It gets answered — necessarily with a non-answer — and
that non-answer is promoted into `decisions.json`, where every downstream
capability inherits it as settled fact about the project. When it is also marked
blocking, it burns the verification loop's budget and escalates to a human gate
that cannot fix it either, because the retried producer reproduces the same
failure every lap.

Instead: fail the step, put the diagnosis in the report's prose with the paths
you actually tried, and route the defect the same way the reviewer does — a
`#calibration:` tag, or a `report_issue` draft naming WHERE it lives (a
`SKILL.md`, a numbered `execution-protocol.md` section, a capability YAML, or
the Stepwise component) and WHAT the change is. Fix the class at the shared
layer, not this one run.

Field evidence (2026-09-06): a capability whose `domain_boundaries` / `adrs`
parameters arrived as absolute paths — composed against `output_folder` into a
doubled path — produced four such rows across two verification steps and one
conformance step. All four reached `decisions.json`. One of them recorded "No
platform conformance claim is made; findings remain empty until the ADR
collection is provided" as a project decision; the ADR collection existed, at
the path the harness had mangled. Three were blocking, cost two escalations to
human gates, and none was answerable by the humans they escalated to.

**Your `## Open Questions` section lists what THIS run raised — nothing else.**

Your prompt carries these blocks above this one: `## PROJECT DECISIONS` (every
human decision on the project, `[OQ-xxxxxxxx] question → value`), `## DECISIONS`
(settled — apply them) and `## OPEN QUESTIONS` (still undecided in this run).
None of them is yours to re-publish.

1. **Do not copy the inherited questions into your artifact.** They are already
   tracked, with their own ids, their own provenance and their own place in the
   operator's queue. A copy in your document is read back out on completion as a
   fresh raise, in a new piece of work — which is how one question becomes five.
   Measured on 22 live projects: 304 of 776 multi-raise records had grown this
   way, and operators watched their counters climb (42 → 48 → 52) while nobody
   had asked anything new.

   Where your work depends on an inherited question, proceed on the assumption
   stated in the block, and say so in ONE sentence in the body citing the
   bracketed id (`… assuming the seven-year retention in [OQ-1d0efb0c] …`). That
   is a reference, not a register.

2. **If the subject is under `## DECISIONS` or `## PROJECT DECISIONS`, it is
   settled.** Apply it, cite the id, and move on. Re-raising it puts a decision the operator already made back in front of
   them.

3. **If a genuine residual gap remains, raise the GAP — not the question again.**
   An answer of "one desktop and one mobile viewport; 768px provisional" leaves a
   real question about tablet, and *that* is what to emit. Re-emitting "viewport
   breakpoints not stated" reads as though the answer never arrived.

4. **If you genuinely must restate one, carry its id.** Copy the bracketed id
   verbatim into `inherited_from` (and into `id` if your section has no local id
   of its own). Identity is derived from the question TEXT, so a restatement in
   different words becomes a second row unless the id says otherwise. Do not
   rebuild a carried-forward table for this — those registries are retired; the
   prompt blocks are the carry-forward.

Field evidence (2026-08-26): a step received eleven answered questions in its
`## DECISIONS` block — including a concrete "768px provisional
breakpoint; evergreen browsers; automated accessibility checks" — and raised
"viewport breakpoints/device matrix for responsive rendering not stated" an hour
later, citing nothing. On that project the same underlying question ended up as
SEVEN rows across three capabilities, and a human answered it repeatedly, each
time saying the previous answer had been fine. Every one of the fifteen
still-open rows had been raised after its answer already existed.

**Wording.** Copy `question` verbatim from the section you wrote. Identity is derived from the question text, so a reworded sidecar entry becomes a SECOND record instead of enriching the one already tracked.

**Ordering.** No change to §11.4 — `_open_questions` is one more key in the same JSON, written at the same point.

**Standalone runs.** No `## Run metadata` block means no sidecar at all (§11 trigger gate), and therefore no `_open_questions` either — this is the one case where the key is legitimately absent. The artifact still carries the section; nothing is lost.

---

## 16. Skill Reference File Resolution

Universal — applies to every skill that ships a `references/` folder (or any
other bundled file: `evals/`, `templates/`) and tells you, in its own SKILL.md,
to "Read `references/<name>.md` now."

### §16.1 The rule — resolve relative to your own skill directory, never search for it

The path in that instruction is already correct as written. It is relative to
**the directory that contains the SKILL.md you are currently executing** — the
same directory the harness resolved before invoking you. You already know that
directory; do not rediscover it.

- Compose the full path directly: `<this skill's own directory>/references/<name>.md`,
  then `Read` it. **Read it directly — do not `glob`/`ls`/`find` the composed path
  first "to check it exists."** §16.2's gitignore failure hits a narrowly-scoped
  existence check exactly as hard as a broad search: `.coda/skills/` is gitignored,
  so `glob(".coda/skills/<skill>/**/*")` reports zero results even when the
  directory is right there. Observed 2026-09-04: an `opening-pull-request` run
  correctly derived the composed path, citing §16 by number, then ran that exact
  glob "to check" before reading — got the gitignore false-negative — and only
  then fell back to the broader searches this section forbids. The pre-check was
  the mistake, not the fallback that followed it. Only `Read`'s own outcome is
  evidence of whether the file is there; a `Read` failure is when §16.3 applies.
- **Never** run a repository-wide or filesystem-wide `glob` or `find` to *locate*
  a bundled reference file. That file's location is not unknown — it is a fixed
  offset from a directory you are already inside. (`grep` is not listed because
  you do not locate a file by name with it — but the §16.2 scan ban covers an
  unscoped `grep -r /` just the same.)
- This applies even under REPAIR/REBUILD, multi-turn resumption, or delegated
  sub-agent execution: whoever is executing the skill's steps still knows the
  skill's own directory and must not search for what it already has.

### §16.2 Why unscoped search fails here

Two failures compound into a session-hanging defect, observed three times across
`defining-quick-story` and `implementing-quick-change`:

1. **A gitignore-respecting `glob` returns zero hits even though the file
   exists.** `.coda/skills/` is excluded in this repo's `.gitignore`. A `glob`
   tool that honors `.gitignore` will report 0 results for a skill installed
   under that path — that is a search-tool blind spot, not evidence the file is
   missing.
2. **Recovering from the zero-hit result by broadening the search, instead of
   narrowing it, escalates to a filesystem-root scan** (e.g. `find / -iname
   "scope-escalation.md"`). An unscoped `find` over the whole filesystem can run
   for the full step timeout (1800s+ observed) before self-recovering, and it
   fires this even *after* the file was already found via a narrower, correctly
   scoped search — pure wasted wall time on top of the original mistake.

The fix is not a better search — it is no search. §16.1 removes the need to
search at all.

**Scope note — the two bans here have different reach.** "Never search for a
bundled file, `glob` included" is §16-specific: it applies because the file's
location is already known, and because a gitignore-respecting `glob` is blind to
`.coda/skills/`. "Never scan the whole filesystem" is **universal** — it holds
for every search in every step, whatever you are looking for: no `find` without a
path argument, and none rooted at `/`, `~` or `$HOME`. A *scoped* `find` over a
directory you were given is fine and several skills instruct one; `glob`/`grep`
remain the right tools for repo-scoped search outside §16's narrow case.

### §16.3 If a bundled reference truly appears missing

If, after applying §16.1 (composing the path directly, not searching), a `Read`
of that composed path genuinely fails — the skill package itself is corrupted or
incomplete, not merely unreachable via a gitignore-filtered search tool — treat
this as a `data_unavailable` condition per §9 (non-interactive execution): record
it and escalate/stop. Do not retry by broadening a search; a broader search
cannot fix a missing file, and (§16.2) it can hang the session trying.


### §16.4 Cross-skill paths pinned by the caller (`output_spec_source` and friends)

§16.1 covers a reference bundled inside YOUR skill. A capability step may instead pin a path
into a DIFFERENT skill — most often `output_spec_source`, naming the producer's template or
SKILL.md so a review step can derive its criteria from the real output contract.

**The runtime prefix in that pinned path is an alias, not a literal.** Capability YAML is
written once and executed by several runtimes, each of which materialises skills into its own
directory: `.agents/skills/`, `.coda/skills/`, `.kilo/skills/`, `~/.codex/skills/` have all
been observed for the same capability. A path pinned as
`.agents/skills/<producer>/references/<file>.md` therefore resolves, at run time, to

> `<the skills root YOU were loaded from>/<producer>/references/<file>.md`

You already know that root: it is the parent of your own skill directory, the same one §16.1
tells you not to rediscover. Read the composed path directly. Treat the pinned prefix as
naming *which skill and which file*, never *which runtime*.

**A pinned path that "does not exist" is almost always this.** Before recording a missing
producer spec, recompose it against your own skills root and read that. Observed 2026-09-05:
two verification steps in one project reported their caller-pinned `.agents/skills/...` paths
absent and fell back — one to a bundled template, one to scoring the input as failed — while
the file was present the whole time under `.coda/skills/`. One of those runs returned FAILED,
weighted 0, on a producer artifact it never opened.

If the recomposed path also fails to `Read`, §16.3 applies: `data_unavailable`, record and
escalate — and say which roots you composed against, so the capability can be corrected.

---
