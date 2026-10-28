> **Execution Protocol section file — §10 Section-by-Section Generation.** Routing + universal sections (§2, §10.5, §11) live in [`execution-protocol.md`](../execution-protocol.md). § numbering is preserved.

## 10. Section-by-Section Spec Generation

**Applicability.** Skills that produce a single multi-section spec file from large unstructured inputs (project briefs, transcripts, legacy docs, prior research artifacts). Examples: `researching-prd`, `planning-code-tasks`, `implementing-user-stories`, `generating-test-cases`, `defining-qe-strategy`, `researching-feature-impl`.

Skills that produce many small files (e.g. `implementing-code` writing source files) use the list-shape `_progress.json` variant (Section 2) — this section does not apply.

**Failure modes prevented.**
- **Silent SIGINT** from long thinking loops with no on-disk progress (skeleton-first fixes this).
- **Gateway 504** from generating all sections in a single huge completion (one-section-per-write fixes this).
- **Input-token explosion** from re-reading raw sources on every section (extraction pass fixes this).
- **Bulk-rewrite fallback** when targeted edits hit a snag (Edit-only discipline + ASCII stub text fix this — see Tool Discipline below).

---

### 10.1 Section-shape `_progress.json`

```json
{
  "skill": "{skill-name}",
  "session_id": "{SESSION_ID}",
  "status": "RUNNING",
  "started_at": "{ISO timestamp}",
  "completed_at": null,
  "skeleton_written": false,
  "sections": {
    "{section_name_1}": "pending",
    "{section_name_2}": "pending"
  }
}
```

Per-section status flips `pending` → `complete` after the section is populated and the `_progress.json` update has been written. `skeleton_written` flips to `true` after Phase A completes.

**Section value — string or object.** A section entry may be either the plain status string shown above, or an object carrying that status plus skill-specific fields:

```json
"sections": {
  "Findings": { "status": "pending", "iterations_used": 0 }
}
```

Use the object form only when the skill genuinely needs the extra fields (e.g. `reviewing-code` tracks `iterations_used` to enforce its iteration cap). Pick one form and use it for every section in the map — never mix string and object entries in the same tracker, or the resume parser has to branch per key. A reader that accepts the object form MUST read the status from `.status`, not from the value itself.

Legal section statuses: `pending`, `in_progress`, `complete`. These are item-level statuses and are lowercase — they are a separate vocabulary from the top-level `status` field, which is governed by the §2 closed set (`RUNNING`, `PARTIAL`, `COMPLETED`, …). Do not compare the top-level status against `"complete"`.

---

### 10.2 Source Extraction Pass

**Trigger.** Run extraction if ANY of:
- More than 2 source files, OR
- Any single source file > 200 lines, OR
- Cumulative source content > 500 lines.

If none of the above (small bounded input), SKIP — read sources directly in Phase B.

**Mechanism.**

```
mkdir -p {SPEC_FOLDER}/_extractions/

FOR EACH source in inputs:
  extraction_path = {SPEC_FOLDER}/_extractions/{basename(source)}.md

  IF MODE == REPAIR AND extraction_path exists AND source unchanged since prior run:
    SKIP — reuse existing extraction.
    LOG to SOURCE_LOG: "reused extraction: {extraction_path}"
    CONTINUE.

  Read source.
  Distill into extraction_path as a structured bullet list, using the
  extraction schema declared by the skill (each SKILL.md defines its own
  schema — fields differ by domain).
  Preserve source line numbers on every entry for traceability.

  Write extraction_path.
  LOG to SOURCE_LOG: { source: {raw_path}, extraction: {extraction_path} }
```

After the pass:
- Inputs reference extraction files, NOT raw source paths.
- Phase B reads extractions only. Raw sources are dropped from working memory.
- The audit must list extraction files alongside raw sources so provenance is intact: `raw_source → extraction → spec_item`.

**Zero Invention still wins.** Extraction is distillation, not invention. Facts not in the source do not appear in the extraction. Inferred industry-standard hints remain `[INFERRED]` in the extraction and become assumption entries (with appropriate IDs) in Phase B.

---

### 10.3 Phase A — Skeleton-First (within 5 tool calls of entering generation)

1. Write SPEC_FILE skeleton: header metadata + section stubs, each stub being the section heading + `status: pending - will be generated`.
   - **The ASCII rule applies to the stub string only.** Write it with an ASCII hyphen (U+002D) — never em dash, never en dash. Non-ASCII characters *in the stub* create downstream encoding traps for any agent that later reaches for shell tooling. The rule does NOT extend to titles, headings, or any other content the skill's output template specifies: those keep the template's characters exactly. Normalizing a template's em dash out of a title because of this rule is a conformance defect, not compliance with it.
   - **Where the skill declares an output template, the skeleton carries that template's structural shape, not only its headings** — full header field list, every heading and subheading, and every table's header + separator row, transcribed from the template file while it is open. Stubs go underneath the shape. Rationale: §10.4 step 5 and §10.4.1 rule 3 require dropping the template between sections and §10.4.1 rule 1's load allowlist excludes it, so any structure not pinned in Phase A is recalled from memory N sections later. Pinning it costs one read; not pinning it is how table columns, header fields, and title delimiters drift. The skill's own SKILL.md owns the specific field and column lists.
2. Update `_progress.json`: set `skeleton_written: true`. All `sections` remain `pending`.

If you find yourself in a 3rd `think` block before the skeleton write, STOP and write the skeleton now. Count ANY tool call (think, view, bash, edit).

**Tool:** `Write` for the initial skeleton (single file create).

---

### 10.4 Phase B — Populate, One Section At A Time

For each section, in the order defined by the skill's template:

1. **Gather only what this section needs:**
   - Read the relevant `_extractions/*.md` entries (or raw sources if extraction was skipped per 10.2 thresholds).
   - Read SPEC_FILE ONLY IF this section depends on cross-section state. Each SKILL.md declares its cross-section dependency list.
2. Generate the section body using only the inputs gathered in step 1.
3. **Replace the section's stub with populated content using `Edit`.** One `Edit` call, one section.
4. Update `_progress.json`: set `sections.{section_name}: complete`.
5. **Drop the section body from working memory before moving to the next section.** Do not carry populated section content forward in your reasoning. If a later section needs it (per the skill's dependency list), re-read SPEC_FILE then.

---

### 10.4.1 Per-Section Context Budget (hard cap)

**Why this exists.** A REPAIR/CONTINUE run once hit ~2M prompt tokens with **zero sections written** before a 504 gateway timeout: it preloaded every `_extractions/*.md` file upfront and planned all pending sections in a single `think` block, so Phase B never started. The per-section budget below makes that failure mode unrepresentable.

**Hard rules — applied at the start of EACH section iteration in Phase B:**

1. **Load only what THIS section needs.** Before generating section `N`:
   - You may have loaded in this iteration: `_progress.json`, the SPEC_FILE (only if Section 10.4 step 1 requires it for cross-section state), and the `_extractions/*.md` files that contain entries for section `N`.
   - You may NOT have loaded: extraction files that feed only other sections, raw source files (those were distilled in Step 10.2 and must not be re-read in Phase B — already a §10.5 rule, restated here as a budget concern), prior populated section bodies still in your reasoning trace from earlier iterations.

2. **No multi-section planning.** Do NOT compose a `think` block that drafts content for more than one pending section. One section per inference call applies to **reasoning**, not just to writes. A `think` block that outlines sections `N`, `N+1`, `N+2` is a §10.4.1 violation — split it into one micro-plan immediately before section `N`'s `Edit` call.

3. **Drop between sections.** After section `N`'s `Edit` succeeds and `_progress.json` is updated:
   - Drop section `N`'s body, the extraction files you read for it, and the micro-plan from your reasoning trace.
   - The next iteration starts from disk truth (`_progress.json` + targeted re-reads), not from accumulated working memory.

4. **Soft prompt-size gate.** If you can observe your prompt size, treat **150K tokens** as the per-iteration soft cap. If you reach it before writing section `N`:
   - Stop loading. Write section `N` with what you have, or mark fields you cannot fill as `pending` + register them in `open_questions`.
   - Do NOT keep reading "one more extraction to be thorough." That is the pattern that produces the 2M-token failure.

5. **Symptom of violation.** If you find yourself in your 5th `view` call of Phase B without an intervening `Edit` to SPEC_FILE, you are violating §10.4.1. STOP, write the current section with the extractions already loaded, then resume.

**Why a budget and not a planning step:** A planning step ("first decide which extractions feed which section, then read them") is itself a load. The cheapest enforceable rule is "don't have loaded what you don't need RIGHT NOW" — checked at section-iteration boundaries, not via upfront planning.

---


> §10.5 (Tool Discipline) is UNIVERSAL — it lives inline in the root execution-protocol.md.

### 10.6 CONTINUE on Re-Entry

If a skill is invoked while its `_progress.json` shows `status: RUNNING` with `skeleton_written: true` and a partial `sections` map, this is a CONTINUE invocation.

**Resume protocol (in order — do not skip steps, do not preload):**

1. Read `_progress.json` ONLY. Do not read anything else first.
2. Identify the first `pending` section in template order — call it `N`.
3. Read the SPEC_FILE ONCE to confirm the skeleton + already-populated sections match `_progress.json`. This is the only full SPEC_FILE read of the resume phase.
4. Enter Phase B at section `N`. From this point §10.4 + §10.4.1 govern — load only the extractions section `N` needs, write it via `Edit`, update `_progress.json`, drop, advance.

**Forbidden on re-entry (the failure mode this clause exists to prevent):**

- Re-reading all `_extractions/*.md` files upfront "to refresh context." Each extraction is read lazily in Phase B, only for the section that consumes it. A run that opens every extraction before the first `Edit` is violating this clause.
- Re-running Step 10.2 (Source Extraction Pass). Extractions are on disk from the prior run; the §10.2 REPAIR-reuse rule (lines 619–622) applies.
- Composing an upfront `think` block that plans all remaining `pending` sections. See §10.4.1 rule 2 — one section's plan per inference call, drafted immediately before that section's `Edit`.
- Re-reading raw source files. Already a §10.5 hard rule; restated here because re-entry is the iteration where it is most commonly violated.

**Symptom of a misbehaving resume.** If after step 3 your next 3 tool calls are all `view` calls against `_extractions/*.md` files, you are in the failure mode. Stop the cascade: pick the first pending section, load only its extractions, write it.

This is distinct from REPAIR mode (Section 7). REPAIR corrects already-completed specs based on `failure_feedback`. CONTINUE finishes in-progress ones based on `_progress.json` state.

---

### 10.7 Composition with Section 2 (FIRST ACTION / LAST ACTION)

Section 10 extends Section 2; it does not replace it.

- **FIRST ACTION (Section 2):** write `_progress.json` with `status: RUNNING` before any other file write. Section-shape skills initialize the `sections` map with all entries set to `pending` and `skeleton_written: false`.
- **Skeleton write (Section 10 Phase A):** within 5 tool calls of FIRST ACTION, also write the SPEC_FILE skeleton and flip `skeleton_written: true`.
- **LAST ACTION (Section 2):** after all sections are `complete`, set `status: COMPLETED` and `completed_at`.

---

### 10.8 Skill-Level Enforcement

Like FIRST / LAST ACTION (Section 2), every skill applying this protocol MUST include inline callouts — agents do not reliably follow protocol-only references for the generation discipline. The SKILL.md provides skill-specific contract bits that Section 10 cannot:

| Contract bit | Owned by SKILL.md | Owned by Section 10 |
|---|---|---|
| Section list in template order | Yes | No |
| Cross-section dependency graph | Yes | No |
| Extraction schema (which fields) | Yes | No |
| Trigger thresholds (10.2 defaults) | Override if needed | Default values |
| Skeleton / Edit discipline | No | Yes |
| `_progress.json` shape | No | Yes |
| CONTINUE semantics | No | Yes |

Recommended SKILL.md structure for section-shape skills:

1. **Step N.5 — Source Extraction Pass:** "Apply Section 10.2 with the extraction schema below: …" then provide the skill's extraction schema.
2. **Step N+1 — Generate Spec:** "Apply Section 10 Phase A then Phase B. Section list in template order: … Cross-section dependency graph: …"

---

### 10.9 Independent-Section Parallelism (§14 whitelist)

Section generation is **sequential by default** — the cross-section dependency graph (§10.4) is real: later sections read earlier ones for declared state, so populating them out of order or in parallel corrupts the carry-forward. This is why §14 (Parallel Fan-Out) does **not** apply to section-shape generation in general.

**The one exception:** a skill MAY declare an `independent_sections` whitelist — sections that provably read *nothing* from any sibling section (their entries in the §10.4 dependency graph are "none"). Only those whitelisted sections may be generated by parallel §14 workers, and only under these rules:

- **Skeleton first, always.** Phase A (§10.3) still runs serially and writes the full skeleton (all stubs) before any parallelism. Workers never create the file or the skeleton.
- **One worker edits only its own pre-created stub.** Each §14 worker populates exactly one whitelisted section into the stub the coordinator already wrote — never a sibling's stub, never the header, never a dependent section.
- **Coordinator merges + owns everything else.** Dependent (non-whitelisted) sections are still populated serially by the coordinator, in dependency order, after the independent ones are merged. The coordinator does the single `_progress.json` update per §14.2.1 (workers write shards only) and the §11 sidecar last.
- **Fallback identical (§14.5).** With delegation UNAVAILABLE, the whitelisted sections are populated inline in template order like any other — the output is byte-identical.

If a skill does not declare `independent_sections`, treat **all** sections as dependent and generate them serially (the safe default). Declaring a section independent that actually reads a sibling is the failure this whitelist guards against — the declaration is the audited claim (engineering-skills validates it).

---

