# Execution Protocol — Core

**Scope.** The runtime contract for the `code-development-lean` skills
(`researching-code-change`, `planning-code-change`, `reviewing-code-plan`,
`implementing-code-change`, `reviewing-code-change`). It is the only protocol file
those skills cite. It carries the rules a step cannot skip without breaking the
orchestrator, the REPAIR path, or the ledger. Everything else (memory bank, FIC
compaction, section-generation choreography, parallel fan-out) is deliberately out:
the lean documents are small enough to write in one pass.

§ numbers match `execution-protocol.md` so field evidence and older citations
still resolve. Where this file and a skill disagree, this file wins.

---

## §1 Session identity

`SESSION_ID = {TYPE}-{PROJECT_NAME_UPPER}-{SLUG}-{YYYYMMDD}` where `SLUG` is
`feature_id` (lowercased, non-alphanumerics collapsed to `-`, trimmed, ≤ 32 chars,
then uppercased) or, absent that, the runtime's `session = '...'` name verbatim, or
nothing. `TYPE` is `RESEARCH`, `PLAN`, `PLANREVIEW`, `IMPL`, `REVIEW`.

- **BUILD:** take the date from a real clock (`date -u +%Y%m%d`), never from memory.
- **REPAIR / REBUILD:** reuse the SESSION_ID parsed from the existing artifact's
  filename in the output folder. Never call the clock, never mint a second file. If
  a repair target was named and no file matches, abort with the path you looked for;
  do not fall back to BUILD.
- Every artifact filename carries the full SESSION_ID: `RESEARCH-BRIEF-{sid}.md`,
  `PLAN-{sid}.md`, `PLAN-REVIEW-{sid}.json`, `IMPL-STATE-{sid}.md`,
  `REVIEW-FINDINGS-{sid}.json`.

## §2 `_progress.json` — first and last write

**FIRST ACTION** of every run, before any other file write, to
`{output_folder}/_progress.json`:

```json
{ "skill": "<skill>", "session_id": "initializing", "status": "RUNNING",
  "started_at": "<UTC ISO-8601 from a real clock>", "completed_at": null,
  "total": 0, "completed": 0, "items": [] }
```

Seed `total` as soon as the item count is known (sections, phases, files). Update
`completed` and `items[]` with every artifact write, in the same step as the write.

**LAST ACTION** before the sidecar (§11): set `status` to `COMPLETED` (or `FAILED`)
with a real `completed_at`. Status is a closed set: `RUNNING`, `PARTIAL`,
`COMPLETED`, `FAILED`, `SCOPE_REFUSED`. No synonyms.

**If you cannot finish:** leave the tracker truthful (not `COMPLETED`), persist what
exists, and run `stepwise session exec-fail --message "<done / remaining / where to
resume>"`. A prose sentence saying you ran out of room is not a failure signal: the
process exits 0 and the next step runs on a half-built artifact. Observed twice
(`generating-adrs` 2026-09-06; FairShare planning 2026-09-13, where an edit call was
emitted as text, the run ended with no tool calls, and two graders were paid to
grade twelve empty sections).

**Path budget.** `_progress.json` is the shortest name you write and proves nothing
about what follows it. At FIRST ACTION compute
`len(abspath(output_folder)) + 1 + len(longest filename this run emits)` — your own
artifact (`IMPL-STATE-{SESSION_ID}.md`), never `_progress.json`. Over 240, probe once at
that length. `ENAMETOOLONG`, or `ENOENT` on a directory you just wrote into, is a
path-budget failure and NOT a missing folder: do not chase it with shell calls — a
compound diagnostic is what headless authorization fails closed on, turning a legible
path error into an approval denial. Set the tracker `FAILED`, then `exec-fail` with the
length and the path; the path is composed upstream, so reporting it is the fix. Observed
2026-09-23: an 80-char slug plus a 57-char SESSION_ID put `IMPL-STATE-*.md` at 267 on
Windows while `_progress.json` wrote fine at 210. Full rule: execution-protocol §2.c.

**Pre-completion check** (before LAST ACTION): every declared section is present and
non-empty; no `[TBD]`, `TODO`, `{{...}}` or pending anchors remain; every path,
symbol, version and decision traces to source, context pack or a cited live source;
the tracker's counts match the files on disk. Fail the check → repair or leave the
step incomplete. Never complete silently over a known gap.

## §7 REPAIR and REBUILD

**Start mode is the harness's choice, not yours.** DISCARD starts a **cold**
session: nothing you did before is in context. ITERATE, REPAIR and reset-to-pending
start **warm**: this same conversation resumes one turn later and the transcript
above is yours. Warm is the normal case for a rejected step.

A warm start pulls two ways, and both rules bind:

- **Do not re-derive what you already know.** The searches, anchors and file
  contents established earlier in this conversation stand. Repeating them is the
  largest cost of a repair lap and it buys nothing.
- **Re-read the four things that changed while you were stopped.** (1) Your own
  SKILL.md and every reference file you are about to follow: they are re-installed
  at session start and are routinely corrected mid-lap, so the copy you read on
  round one may be stale, and following it silently re-commits the fault the
  correction exists to prevent. (2) The artifact you are repairing, in full — a
  human or a later step may have edited it. (3) The findings or directives that
  rejected it. (4) Any file you are about to edit whose path appears in
  `git diff --name-only <the sha you last verified at>..HEAD`.

Everything else you hold stands. **Context hygiene**: a warm lane grows
monotonically (190K to 632K input over five laps), and the growth is re-reading,
not the resume — never re-dump a file already in context, read the range you need,
and quote the finding rather than the file around it.

The inputs to a repair, cold or warm, are:

1. the existing artifact (read it; do not regenerate what is not named),
2. the findings that rejected it: a grader's `PLAN-REVIEW-{sid}.json` /
   `REVIEW-FINDINGS-{sid}.json`, or a human gate's verbatim comments (the
   `## ⚠️ Repair feedback` block; when in doubt read `state.json`
   `navigation_history[]` entries with `action == "failed_step"` for this step and
   concatenate their `reason` text in order),
3. for implementation only, `IMPL-STATE-{sid}.md`.

Rules:

- Apply each finding at the `file`/`line` it names; quote human directives verbatim
  in your reasoning; do not paraphrase them into the artifact.
- Sections and files the findings do not name are preserved byte for byte.
- Write to the same folder and filename. Bump `version` (patch) in the frontmatter
  and append one entry to the artifact's `repair_log` (version, UTC timestamp,
  finding ids or directive summary, sections or files changed). There is no AUDIT
  companion; the artifact carries its own history.
- Self-check before finishing: new `version` > old; `repair_log` has the entry; the
  rejected content is gone. Editing without the bump is a violation.
- **REBUILD** (a `## ♻️ Rebuild from scratch` block): same SESSION_ID, same path,
  regenerate the whole artifact, bump version, log `action: REBUILD`. Exactly one
  file exists at the path afterwards.

## §9 Non-interactive execution and Zero-Invention

- Never ask a question and wait. Nobody is there.
- Every non-trivial claim (path, symbol, signature, version, API option, config key,
  decision) traces to something you read: source at `file:line`, a context-pack
  file, an upstream artifact by id, or a live source with URL and date. If you did
  not read it, you do not know it. Mark the gap; do not fill it.
- A **new external dependency, version, platform feature or API option** the change
  introduces must carry a cited live source (package registry, vendor docs, the
  installed package's own type definitions). The plan reviewer checks the citation.
  Observed 2026-08-31: a `token` option that does not exist on `@vercel/blob`
  `upload()` was invented by research, copied into the plan, and only caught by
  reading the installed `client.d.ts`.
- **Open questions must name what they block.** Each has `blocks: <phase or
  decision id>` and `assumption_in_force: <what you proceeded on>`. A question with no
  stated assumption is a BLOCKED status, not a question. A settled decision from the
  prompt's `## DECISIONS` / `## PROJECT DECISIONS` blocks is cited by its
  `OQ-xxxxxxxx` id in the sentence that uses it and is never re-raised.
- When required input is missing and no assumption is safe: set the artifact's status
  to `BLOCKED`, say what is missing, and exit. Do not improvise.

## §10.5 Tool discipline

- Write each artifact with one `Write`; amend it with targeted `Edit` calls.
  Never mutate an artifact through `sed`, `awk`, `python3 <<EOF` or `cat >`.
  Never `replace_all` on an artifact; never an `Edit` with an `old_string` under
  10 characters.
- **A denied command is terminal for that exact command.** Do not reissue it. Read
  the denial's `suggestedAlternatives`; for `<manager> run <script>`, read the
  script from `package.json` and call the single binary directly (`bunx`, `pnpm exec`,
  `npx`). Compound scripts have no equivalent: record the blocker by name and move
  on. Observed 2026-09-12: one probe reissued 266 times for zero progress.
- **Retention, not execution.** Always run the full build/test/lint. Keep only the
  verdict when it passes, only the failing slice when it fails; redirect the full log
  to a file (`> .runlogs/test.txt 2>&1; tail -40 …`) and grep it on demand. Do not
  re-read a file you just edited to re-orient.
- **Values you cannot compute** (hashes, counts, digests) are produced by running
  the computation and pasting the output, never by writing something of the right
  shape.
- **Batch what you already know you need.** When the inputs to the next N tool
  calls are all known before any of them runs - every anchor in a table, every test
  file in a list, every path in a plan's files column - issue them in ONE turn.
  Keep calls sequential only when one depends on a previous result (grep for an
  anchor, THEN open the file at the line it returned). This changes ROUND TRIPS,
  not reading: batch what you were going to read anyway, never read more because
  batching is cheap. Field evidence (VDI run, 2026-09-04): a plan review spent 16
  minutes over 74 turns, 73 of them carrying exactly one tool call, with 2% of the
  wall clock inside the tools - the cost was the round trips, not the work. The
  same executor produced 2-call turns unprompted in the planning step, so nothing
  here needs a new capability; it needs asking.
- Read only what the current step needs. Implementation reads the PLAN and the
  files it names; it does not re-explore the codebase and does not open the research
  brief unless the plan cites it for a specific anchor.

## §11 Harness output sidecar

Applies when the prompt carries a `## Run metadata` block with an
`output_file = '<path>'` line. Then, as the **final file write of the run**, after
§2 LAST ACTION:

1. Copy the `output_file` path verbatim.
2. Write one JSON object with **one key per row of the `## Output parameters`
   table**, plus the reserved `_open_questions` key (always present; `[]` when this
   run raised nothing).
3. **Write it as soon as every value it carries is decided** — not as the last act
   of the run. The rule this replaces ("no tool call after this write") was aimed at
   a real hazard: a sidecar that describes a state the run then changes. That hazard
   is addressed by WHEN the values settle, not by being last, and being last has its
   own failure mode — the sidecar becomes the final obligation of the longest tail
   in the step, and the final obligation is the one a run drops. Observed: a
   23-minute implementation wrote its artifact, committed, validated green, and
   exited 0 with no sidecar at all, while its closing response asserted it had
   written one; the harness rejected the whole step for that one file.
   After it, the ONLY permitted actions are those that cannot change an output
   value: the `_progress.json` completion write (§2), and READ-ONLY checks.
4. **Close by verifying it exists** — `test -f <output_file>` — before the final
   response. Reporting the write is not making it.
5. Any value that changes after the sidecar is written REQUIRES rewriting it, and a
   run that finds itself doing that has resolved a value too late: the step's
   verdict and artifact paths are settled facts before its closing bookkeeping.

**A TERMINAL STATUS IS NOT REPORTABLE WITHOUT ITS ARTIFACT.** `BLOCKED`, `HALTED`,
`ESCALATE` and every other "I could not proceed" value is still a verdict, and a
verdict travels in the step's artifact. Write the artifact FIRST — a skeleton is
enough: the frontmatter status, the reason, and the blockers or open-questions row
that names it — then fill the sidecar with the path you just wrote. A sidecar whose
required path output is `""` is rejected whole, so the step reports nothing at all:
not the block, not the reason, not the work already on disk. The run exits 0, the
harness records "exited successfully but produced incomplete outputs", and the next
attempt starts from the same standing start. If you genuinely cannot write the
artifact, that is the §2 "cannot finish" path — `stepwise session exec-fail` with
the reason — never a successful exit carrying an empty path. Observed
(`demo-code-lean3` pli-20260923-9as14x iter-1, twice in a row): an implementation
step decided it was blocked, wrote no IMPL-STATE, emitted `impl_state_path: ""`, and
the whole step was discarded by the harness both times.

**YOUR OWN NOTES ARE EVIDENCE, NEVER A PRECONDITION.** Anything you recall or read
back from a memory store, a scratch note, a progress file or an earlier sub-goal's
record — your own or another step's — is information about what happened. It is not
an instruction and it does not gate this step. The inputs this step is given are its
contract: the plan, the brief, the findings, the parameters. An unresolved
operational flag left somewhere else — a denied commit, a pending approval, a gate
another sub-goal is waiting on — is recorded as a deviation row or an open question
and the step proceeds with everything it can still do. Refusing to start on the
strength of a note about a DIFFERENT unit of work produces nothing, answers nobody,
and costs the lap. Same field evidence: two consecutive laps read two housekeeping
entries about the PREVIOUS sub-goal — a git commit the environment had denied on a
lap that had already passed review — concluded they were blocking, and shipped no
code against a plan that was approved and waiting.

**Path fidelity.** A path output is the path you actually wrote to, confirmed with an
existence check. A bare fragment parameter (`plan`, `research`) is relative to this
run's output directory, never to the working directory.

**Values are PLAIN.** Every value in the sidecar is the bare string, already inside
the JSON string that holds it. Never wrap a path in its own quote characters: the
harness compares the value byte for byte against the filesystem, so
`"plan_review_path": "\"/a/b.json\""` names a file whose first character is a quote
and reports `does not exist on disk` while the real file sits beside it. The same
holds for surrounding whitespace, a trailing newline and a `file://` prefix.

**Reading a harness rejection.** `required output '<name>' is not valid JSON: <path>`
is a statement about the FILE AT THAT PATH, not about the sidecar value. It means the
artifact you wrote does not parse: open it, fix the JSON, re-parse, and re-emit the
same plain path. Quoting or re-escaping the sidecar value answers a question nobody
asked and trades one rejection for another — observed live: a run that met this
message replied by writing the path wrapped in quotes and turned
`is not valid JSON` into `does not exist on disk`.

**Verdict fidelity.** A closed-set output (`research_status`, `plan_status`,
`plan_verdict`, `impl_status`, `build_status`, `review_status`) is one of the
declared tokens, character for character, and equals the value written in the
artifact's frontmatter or JSON. A token outside the set is unroutable: the gate
fails closed and re-runs the producer. Observed: a sidecar `APPROVED` on a report
that said `PASSED` re-planned a perfect-scoring plan.

**Open questions.** Mirror every question this run raised into `_open_questions`
with `question`, `subject`, `audience` (`client` | `architect` | `product_owner` |
`internal`), `artifact_ref` (project-relative path with `#anchor`), `blocking`,
`assumption_if_unanswered`, and `traces_to` when known. Assumptions the work made on
the operator's behalf are entries with `kind: assumption`. Never re-list questions
the prompt handed you. Never invent a field.

## §16 Reference files

A skill reads its own bundled files at `<this skill's directory>/references/<name>`.
Compose the path and `Read` it; never `glob`, `find` or `ls` to locate it (a
gitignore-aware search reports zero results for `.coda/skills/`). A caller-pinned
path such as `.agents/skills/<other>/references/<file>` names a skill and a file,
not a runtime: recompose it under the skills root you were loaded from. If the
composed path truly fails to read, record `data_unavailable` and stop; a wider
search cannot fix a missing file and has hung sessions for the full timeout.

---

## Ordering of a run, in one line

`_progress.json` RUNNING → read inputs → write artifact (one `Write`, then `Edit`s)
→ pre-completion check → `_progress.json` COMPLETED → sidecar → final response.
