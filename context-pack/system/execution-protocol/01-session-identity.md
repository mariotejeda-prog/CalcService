> **Execution Protocol section file — §1 Session Identity.** Routing + universal sections (§2, §10.5, §11) live in [`execution-protocol.md`](../execution-protocol.md). § numbering is preserved.

## 1. Session Identity

**Format (priority-ordered):**

1. If parameter `feature_id` is set: `{TYPE}-{PROJECT_NAME_UPPER}-{FEATURE_SLUG}-{YYYYMMDD}`
2. Else if the Stepwise session_name is provided by the runtime (capability prompt's `session = '...'` line): `{TYPE}-{PROJECT_NAME_UPPER}-{SESSION_NAME}-{YYYYMMDD}`
3. Else (standalone invocation, no orchestrator, no feature_id): `{TYPE}-{PROJECT_NAME_UPPER}-{YYYYMMDD}`

| Skill | TYPE prefix | Example (feature_id="auth-v2") | Example (session_name="cd-CA-157343") | Example (standalone) |
|-------|------------|--------------------------------|---------------------------------------|----------------------|
| `researching-code-design` | `RESEARCH` | `RESEARCH-ECOMAPP-AUTH-V2-20260411` | `RESEARCH-ECOMAPP-CD-CA-157343-20260411` | `RESEARCH-ECOMAPP-20260411` |
| `planning-code-tasks` | `PLAN` | `PLAN-ECOMAPP-AUTH-V2-20260411` | `PLAN-ECOMAPP-CD-CA-157343-20260411` | `PLAN-ECOMAPP-20260411` |
| `implementing-code` | `IMPL` | `IMPL-ECOMAPP-AUTH-V2-20260411` | `IMPL-ECOMAPP-CD-CA-157343-20260411` | `IMPL-ECOMAPP-20260411` |
| `reviewing-code` | `REVIEW` | `REVIEW-ECOMAPP-AUTH-V2-20260411` | `REVIEW-ECOMAPP-CD-CA-157343-20260411` | `REVIEW-ECOMAPP-20260411` |
| `fixing-bugs` | `QUICKFIX` | `QUICKFIX-ECOMAPP-BUG-002-20260411` | `QUICKFIX-ECOMAPP-QUICK-FIX-20260411` | `QUICKFIX-ECOMAPP-20260411` |

**FEATURE_SLUG derivation (deterministic, no LLM judgement):**
- Lowercase the feature_id, then replace any run of non-alphanumerics with a single hyphen.
- Strip leading/trailing hyphens.
- Truncate to 32 chars.
- Then UPPERCASE the slug for the SESSION_ID (matches the `{PROJECT_NAME_UPPER}` style).
- Examples: `"FEAT LAC Android v2"` → `FEAT-LAC-ANDROID-V2`. `"i18n_pack"` → `I18N-PACK`. `"a/b/c"` → `A-B-C`.

**SESSION_NAME pass-through:**
- The runtime supplies session_name verbatim via the capability prompt; do not re-format. Preserve hyphens and case as given. (e.g., `cd-CA-157343` stays `cd-CA-157343` in the SESSION_ID — uppercased only in the final concatenation when the rest of the ID is uppercased.)

**Rules:**
- Set once at initialization. Never changes within a session.
- **BUILD mode — ground the `{YYYYMMDD}` segment and any `date:` front-matter field in the REAL current date, never in assumed/remembered knowledge.** Obtain it from a verifiable source before stamping SESSION_ID or any spec's `date:` field — e.g. run the shell `date -u +%Y-%m-%d` (or the executor's equivalent) and use that output verbatim. Do NOT infer "today" from training data or prior context: this has produced dates and session IDs that drift by months from the actual run date (e.g. a PLAN-SPEC stamped `20260127` when the run actually happened in July), and the drift can differ between artifacts written in the SAME run (filename date correct, `date:` field wrong, or vice versa) — a signature of ungrounded date guessing rather than an environment/clock fault.
- REPAIR mode reuses the existing SESSION_ID from the prior run — do NOT generate a new one. To recover the prior SESSION_ID, parse it from the existing spec filename in the input/output folder (preferred) or from the IMPL-STATE header.
  - **In REPAIR, NEVER call `date()`/`today()`/`time()` or recompute the SESSION_ID from parameters.** The date segment is fixed at BUILD time and is immutable across every iteration of the same artifact; minting a fresh date produces a SECOND file (e.g. `…-20260605.md` vs `…-20260610.md`) instead of overwriting the original — the documented duplicate-on-REPAIR defect.
  - **Recovery algorithm:** glob the recorded output folder for the artifact (`<PREFIX>-*.md`); from the matched filename extract the SESSION_ID as the substring between `<PREFIX>-` and `.md`; reuse it verbatim for every write. If multiple match, pick the most recently modified. If a repair target was named but no file matches → see §7 (ABORT, do not BUILD).
- The SESSION_ID propagates verbatim into every artifact filename listed in this protocol (`RESEARCH-SPEC-{SESSION_ID}.md`, `PLAN-SPEC-{SESSION_ID}.md`, `IMPL-STATE-{SESSION_ID}.md`, `REVIEW-SPEC-{SESSION_ID}.md`, etc.). Filenames are the primary collision-prevention surface — do NOT shorten or omit the SESSION_ID in any output filename.

**Why this format:** Prior runs that used `{TYPE}-{PROJECT}-{YYYYMMDD}` collided when two sessions ran on the same project on the same day, silently overwriting each other's specs. Including feature_id (or session_name as a fallback) keeps every concurrent or sequential run uniquely addressable in the same project folder.

---

