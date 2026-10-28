---
name: _aipods-implement
description: Always-on code-writing discipline for code steps — finish-the-loop, build/test gating, secure defaults (fail closed), completion evidence, fidelity, surgical repair, file/token safety. Covers implementing-code, fixing-bugs, implementing-quick-change. Full mechanics live in each skill.
precedence: 10
scope: code
---
## Code-writing discipline (apply to all code you write, fix, or change)

Builds on _aipods — do not restate it. These are the code-loop rules that, forgotten mid-step, ship broken or incomplete code. Cover implementing-code, fixing-bugs, implementing-quick-change.

**Finish the loop — never report "done"/verified on a partial:**
- Apply ONLY the approved plan/spec/diffs — exactly. No extra files. No silent drop as "known limitation" without a documented blocker.
- A defect you can see in the code you just wrote is NOT a known limitation. Writing a comment that names it (`// FIXME`, `// Defect B`, `// not secure`, `// mock`) is the silent drop wearing a disguise — the comment does not discharge it. Fix it, or if genuinely blocked, log it as a blocker/deviation to the proper channel (open_questions / deviation record / apply verdict feedback). Never ship a self-identified defect as a source comment and report COMPLETED.
- Multi-phase (implementing-code): run every PENDING phase B (generate) → C (verify) → D (finalize); never skip Verify/Finalize; emit its deliverables (IMPL-STATE status, README.md, AGENTS.md final phase, Memory Bank).
- Single-apply (fixing-bugs / implementing-quick-change): apply → verify → set the apply verdict. No phases, no README/AGENTS.

**Build/tests gate — a failing suite BLOCKS "done":**
- Self-correct build/test max 3 attempts; budget spent != license to pass.
- Genuine FAIL after 3 → never report COMPLETED / APPLIED_VERIFIED. Route to repair (`stepwise session exec-fail` / `NEEDS_REDIAGNOSE` / `NEEDS_REPLAN`), status FAILED / APPLY_FAILED.
- Environmental FAIL (sandbox / no network / service down) → BLOCKED or APPLIED_UNVERIFIED (verification skipped), never a clean pass. Do NOT re-run past the 3-attempt cap.

**Secure defaults — fail closed, never open (insecure defaults ship):**
- Any code that gates access on config (auth/authz, feature guard, admin route) MUST **deny** when that config is missing, empty, or unparseable — never grant. A `mock`/`local-dev`/`bypass` branch reachable because an env var is unset IS the production behavior the day someone deploys without it; guard bypasses on an explicit `env === "production"` deny, not on config absence, and prefer refusing to start over running open.
- Secrets (tokens, keys, credentials) you write to disk: owner-only permissions immediately (`chmod 0600` / `mode: 0o600`), never under `/tmp` or a shared/world-readable dir, one canonical sink — do not copy the same secret into multiple config files. Ensure any secret-bearing file the code emits is covered by `.gitignore`; never hardcode a secret literal in tracked source.
- Auth-bearing cookies get `HttpOnly` + `Secure` + a sane `SameSite`; never place an auth token where page script can read it. Privileged/admin/credential endpoints get authn, authz, and rate limiting — not "it's internal".
- Audit/security logs that the spec calls durable must reach a durable sink the enforcement path can write to — an in-memory array lost on restart does not satisfy a durability requirement.
- These are BLOCKING at code review (secure-defaults taxonomy SD-01..SD-10). "Dev only / temporary / the installer sets it / already noted in a comment" is NOT a reason to ship the insecure form.

**Prove completion — record a verdict; a missing one = self-fail before the human gate:**
- implementing-code: plan_adherence, plan_file_coverage, ac_coverage, artifact_fidelity (+ tdad when enabled). fixing-bugs / quick-change: the apply_verdict + its named verification. Any FAIL or missing verdict → self-fail (→ repair).

**Fidelity (sharpens _aipods):** cannot preserve an exact upstream literal (identifier, SQL predicate/order, config key, contract field, dependency pin, path) → STOP, log blocker/deviation. Never a quieter approximation.

**Repair / re-apply (failure_feedback present):** apply only the fixes the feedback names — surgical, not a rewrite. Re-run the full suite; never regress a passing test.

**File + token safety:**
- Read a file before you Edit/Write it. Never edit a path not read this session.
- Use plan/spec-pinned paths; consult `context-pack/codebase-map.md` before any repo-wide grep/glob.
- State files (e.g. IMPL-STATE): write skeleton once, then targeted anchored Edits only — never re-read in full, never replace_all.
- For large or binary-adjacent files, inspect the file type, size, and existing encoding before editing; preserve UTF-8/non-ASCII content exactly, use targeted `Edit` for text, and never pass binary content through a text rewrite.
- Run build/test in full but keep only verdict + failing slice; redirect verbose logs to a file. No full passing logs held in context.
- File wiped / truncated / unexpectedly replaced → git checkout first; never verify a wiped file's build; no bash scavenger hunts.

**TDAD (when enabled — incl. quick-fix red-green):** failing test first → green → refactor. Never xit / xdescribe / fit / fdescribe.

Full mechanics: each skill's SKILL.md + references. On conflict, the skill and the task/spec win over this summary.
