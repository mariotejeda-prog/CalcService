> **Execution Protocol section file — §7 REPAIR / §7.3 REBUILD Mode.** Routing + universal sections (§2, §10.5, §11) live in [`execution-protocol.md`](../execution-protocol.md). § numbering is preserved.

## 7. REPAIR Mode

Triggered when `failure_feedback` parameter is provided.

### 7.1 FIRST ACTION — Mandatory Reviewer-Comment Absorption

Before any spec write in REPAIR mode, the agent MUST:

```
1. Read the latest state.json for the current capability/session.
2. Extract every navigation_history[] entry where action == "failed_step"
   AND step matches the current step being repaired.
3. Concatenate the `reason` text of those entries in timestamp order.
   This is the AUTHORITATIVE reviewer-comment payload — it overrides any
   abbreviated version that may have been passed via the failure_feedback
   parameter alone.
4. Scan that text for INLINE FILE REFERENCES of the form:
     - "Line {N} (near: …)"           — line-anchored content correction
     - "File: {path}"                  — file-anchored content correction
     - "comments file …: {path}"       — separate comment store
   If a separate comment store path is mentioned, load that file and treat
   its contents as additional ground-truth corrections.
5. Quote every reviewer instruction verbatim in the agent's internal
   reasoning before proposing any regeneration. Do NOT paraphrase reviewer
   text in the spec — copy the resolution faithfully.
```

Skipping any of steps 1-5 is a protocol violation. The reviewer-comment payload
loaded here feeds into Section 7.2 (the directive parser).

### 7.2 Directive Application

```
1. Load existing output spec — do NOT start from scratch.
2. If SESSION_ID needed: extract from existing spec filename.
3. Parse the reviewer-comment payload from §7.1 into directives:
   { section: "section_name" | "global", instruction, reason }
4. FOR EACH section in spec:
   - Has directive → load from existing spec, apply change, rewrite section
   - No directive → preserve existing content verbatim. Do NOT touch it.
5. "global" directive → re-execute full generation (Steps 3 onward)
6. Re-run all validation checks after fixes.
7. Produce repair_delta: { fixed: [], still_failing: [], regressions: [] }
8. Version bump + audit trail (MANDATORY — do NOT skip after editing content):
   a. Read the current `version` from the existing spec's front-matter.
   b. Write `version = increment_patch(current)` (e.g. 1.0.0 → 1.0.1) back into the
      SPEC front-matter. Reuse the same filename (§1 — never re-date the filename).
   c. In the AUDIT file (same SESSION_ID): set `mode: REPAIR`, set its `version` to the
      new value, and APPEND (do not overwrite) a `## Repair History` entry:
        - `version`, `timestamp`
        - `directives_applied`: the reviewer directives you actioned (verbatim)
        - `sections_changed`: section names you rewrote
        - `sections_preserved`: section names left untouched
        - `repair_delta`: { fixed, still_failing, regressions } from step 7
9. SELF-CHECK before `final_response` (MANDATORY): confirm (a) the spec `version` you
   wrote is strictly greater than the version you read in 8a, and (b) the AUDIT file
   contains the new `## Repair History` entry. If either is false, you edited content
   but skipped the bookkeeping — fix it now, before finishing. Editing the artifact in
   place WITHOUT bumping the version and appending the audit entry is a §7 violation.
```

REPAIR does NOT start over. Sections without directives are unchanged.

**Path Consistency Rule:** REPAIR output MUST be written to the same folder AND the same
filename as the original run. Read the original output path from the existing spec file,
IMPL-STATE, or `_progress.json` — do NOT re-derive it from parameters. If the original run
used a `feature_id`-scoped path (e.g., `{project}/i18n/code-impl-output/`), the REPAIR run
writes to that same path. Creating a new folder at a different level fragments the artifact
tree and breaks downstream step references.

**Discovery + reuse algorithm (mandatory):**
```
1. Determine the REPAIR search directory:
   - If failure_feedback names a rejected artifact path (e.g. "review rejected for: <path>"),
     use dirname(<path>) as the search dir and basename(<path>) as the target filename.
     Do NOT use the (possibly re-rendered) output-path parameter for discovery.
   - Otherwise use the recorded output path (existing spec / IMPL-STATE / _progress.json).
2. Glob that directory for the artifact (e.g. `<PREFIX>-*.md`). Reuse the matched file's
   dirname AND basename VERBATIM for the write — never recompute SESSION_ID/date (see §1).
3. ABORT, do NOT BUILD: if a rejected path was named but no matching file is found, ABORT
   with guidance — e.g. "REPAIR target not found: <path>. Refusing to BUILD a fresh spec,
   which would duplicate the artifact. Verify the output path or restore the rejected file."
   Silently falling back to a fresh BUILD on a divergent path is the documented duplicate defect.
```

**Violation Signal:** If, during REPAIR, the regenerated spec still contains content
that the reviewer directive explicitly rejected (e.g., still says "greenfield" after
the reviewer rejected greenfield wording), this is a §7 violation. Abort the write,
re-run §7.1, and try again.

### 7.3 REBUILD Mode (discard → regenerate)

Triggered by a `## ♻️ Rebuild from scratch` block (instead of `## ⚠️ Repair feedback`).
This is a **DISCARD**: the reviewer judged the artifact fundamentally wrong, so patching
it is not worthwhile — you REGENERATE it, but you must NOT duplicate it.

REBUILD is the mirror image of §7.2: same file, opposite content strategy.

```
1. FIRST, absorb the discard reason exactly as in §7.1 (read state.json
   navigation_history[] "failed_step"/"navigated_back" reasons for this step; the
   `## ♻️ Rebuild from scratch` block carries the same reason). Quote it verbatim in
   your reasoning — it tells you WHAT was fundamentally wrong so the rebuild avoids it.
2. Resolve the existing artifact's SESSION_ID + output path exactly as §1 / the §7.2
   Path Consistency Rule + Discovery algorithm: reuse the SAME SESSION_ID and write to
   the SAME folder + filename. NEVER mint a new date/SESSION_ID/folder — that creates a
   duplicate (the documented duplicate-on-REPAIR defect applies equally to REBUILD).
3. Run the skill's normal BUILD path (Steps 3 onward / the skill's generation phases)
   to produce the artifact AFRESH. Do NOT load-and-preserve prior sections the way §7.2
   does — the old content is superseded. The only thing you carry forward is the
   SESSION_ID, the output path, and (from step 1) the reason it was discarded.
4. Overwrite the existing file in place with the regenerated content.
5. Version bump + audit (MANDATORY, same as §7.2 step 8, with mode REBUILD):
   a. Read the current `version` from the existing spec front-matter.
   b. Write `version = increment_patch(current)` back into the SPEC front-matter
      (reuse the same filename — never re-date it).
   c. In the AUDIT file (same SESSION_ID): set `mode: REBUILD`, set its `version` to the
      new value, and APPEND a `## Repair History` entry recording `version`, `timestamp`,
      the discard `reason`, and `action: REBUILD (full regeneration, prior content superseded)`.
6. SELF-CHECK before `final_response` (MANDATORY): confirm (a) the spec `version` is
   strictly greater than the version read in 5a, (b) the AUDIT has the new `## Repair
   History` entry with `mode: REBUILD`, and (c) exactly ONE artifact file exists at the
   resolved path (you overwrote, you did not create a sibling).
```

**REBUILD vs REPAIR at a glance:** REPAIR preserves untouched sections and edits only what
the directives name (surgical). REBUILD discards all prior content and regenerates the whole
artifact. BOTH reuse the same SESSION_ID + path and BOTH bump version + append the audit —
the difference is content strategy, never the file identity.

---

