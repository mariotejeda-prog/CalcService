> **Execution Protocol section file — §14 Parallel Fan-Out.** Routing + universal sections (§2, §10.5, §11) live in [`execution-protocol.md`](../execution-protocol.md). § numbering is preserved.

## 14. Parallel Fan-Out

**Purpose.** When a skill's work decomposes into **N independent units of the same shape** — judge samples for one checklist item, per-file review findings, per-story test generation, per-story implementation, files in a diff, decisions in a plan, logs to analyze — those units MAY be **dispatched to parallel workers** instead of processed one at a time inline. A coordinator (the main agent) shards the work, dispatches one bounded unit per worker, then performs the only serial writes: merging worker results, updating the shared trackers, running the aggregation, and emitting the §11 sidecar last. This is the **write-capable** sibling of §12 (which is read-only exploration): §12 workers only read and report; §14 workers may also produce bounded per-unit output within their scope.

**Why it helps.** Independent units have no carry-forward between them, so serial processing is latency the work does not require. Fanning them out collapses wall-clock from *sum-of-units* toward *slowest-single-unit*, and keeps each worker's context bounded to its one unit (the same context-hygiene goal as §5 / §12). The coordinator's context stays lean for the merge + aggregation it alone is accountable for.

**When it does NOT help.** That payoff assumes two things the coordinator must actually check, not presume: that each worker's **input** is mostly its own (§14.1.1 — otherwise N workers re-load one shared corpus N times), and that the harness really **ran the batch concurrently** (§14.7 — some accept a batch and execute it serially). When either fails, fan-out is slower and more expensive than the inline loop. Fan out because the two checks passed, never because the capability line said AVAILABLE.

**Applicability.** Any skill whose route includes §14 **and** that has declared a genuinely independent unit set (see §14.1). This section is **capability-gated and harness-agnostic**, exactly like §12: it applies *only if the running harness can dispatch write-capable workers*. The runtime advertises this on one line in the Stepwise runtime-rules block:

- `Parallel delegation: AVAILABLE — …` → you MAY fan out per §14.
- `Parallel delegation: UNAVAILABLE — …` → use the §14.5 inline fallback.

**What "AVAILABLE" requires — and what it does NOT.** The capability is satisfied by **either** form of concurrency:

- **Concurrent foreground workers** — the harness can spawn several workers in a **single dispatch turn** (one message carrying N worker calls) that run at the same time. *Nearly every agentic harness supports this*, including those with no Stepwise orchestrator.
- **Background / detached tasks** — workers that persist across turns (fire-and-forget).

**Concurrent foreground alone is sufficient. Do NOT require background/detached tasks.** The most common false-negative is reasoning *"detached/background tasks are unavailable → fall back to inline"* — that is **wrong**: if the harness can emit multiple worker calls in one turn (§14.3 step 2), the capability is AVAILABLE and you MUST fan out. Only genuinely-serial harnesses (one tool call per turn, no concurrency of any kind) are UNAVAILABLE.

**Absent advertisement ≠ UNAVAILABLE.** When no `Parallel delegation:` line is present (e.g. a non-Stepwise harness), do **not** default to inline. Determine capability directly: if the harness can dispatch multiple workers in a single turn, treat it as AVAILABLE and fan out per §14.3; fall to §14.5 only when single-turn multi-dispatch is genuinely impossible.

**A third state exists: batch-accepted but serially-executed.** Some harnesses **accept** N worker calls in one dispatch turn and still **run them one at a time**. The batch is the mechanism, but it is not a guarantee: the advertisement says AVAILABLE, the batch is emitted correctly per §14.3 step 2, and wall-clock is still *sum-of-units* — plus each worker's context load on top, which can make it **slower than the inline loop**. Foreground batching alone therefore does not guarantee overlap. This is a property of the running harness, not a skill bug or an agent bug.

**So: prefer a detached-dispatch + join pair when the harness offers one.** If the harness exposes a way to start workers **without waiting** *and* a way to **join / collect** them afterwards, use it for a fan-out: start every worker for the shard detached in one turn, then join on all of them and merge. That combination is what actually produces overlap on a harness whose foreground dispatch is serial, and it has been measured to deliver near-linear speedup where foreground batching delivered none. When the harness offers **only** foreground dispatch, batch in one turn per §14.3 and accept that overlap may not occur — then check (**§14.7**).

This does **not** lower the capability floor. Foreground batching still satisfies AVAILABLE, and on harnesses whose foreground workers genuinely run concurrently it is the whole answer — do not go hunting for a detached primitive that isn't there, and never treat its absence as UNAVAILABLE (§14.5 case 1). Prefer detached **when offered**; batch foreground otherwise.

**Pin the worker model when the unit's output is a judgment.** A harness may bind a different model to detached workers than to foreground ones, silently, under the same "default" model selection. For units that produce a *verdict, score, or gap* (rather than a mechanical transformation), name the worker model explicitly so switching dispatch mode does not switch the judge underneath — otherwise a latency change quietly becomes a verdict-quality change, and the two are indistinguishable after the fact.

**Never name a specific tool, agent, or model ID in a skill** — describe the capability and let each harness bind it.

**Relationship to §12.** §12 = read-only exploration (workers return conclusions + `file:line`, never write). §14 = write-capable fan-out (workers may produce their one unit's bounded output). A skill can use both: §12 to locate the units, §14 to process them. Both share the Zero-Invention verify-before-use duty (§14.4).

---

### 14.1 When to fan out (and when not)

**Fan out when** all of these hold:
- The units are the **same shape** and **independent** — processing unit A does not depend on the result of unit B. **Independence must be declared, not assumed** (the skill states *why* the units share no carry-forward; see engineering-skills P11).
- There are enough units that parallelism pays for the coordination overhead (typically ≥ 3).
- Each unit's output is **bounded and mergeable** by a coordinator rule (mean, concat+dedupe, per-file section, per-story file).
- **Each unit's INPUT is mostly its own** — the units do not all depend on one large shared corpus. See §14.1.1: independence of *logic* is not the same as independence of *input*, and only the second one makes fan-out cheaper.

**Do NOT fan out:**
- **Section-shape generation with real carry-forward** (§10). Sections that reference each other are sequential by default; only a skill's declared `independent_sections` whitelist (§10) may be parallelized, each worker editing its own pre-created stub.
- The **merge, aggregation, ranking, or verdict** — that is the coordinator's serial, accountable work (§14.3). Never let a worker decide the deliverable.
- Work that **mutates a shared file** (the canonical `_progress.json`, `00-index.md`, the carry-forward index, the §11 sidecar). Those are single-writer, coordinator-only (§14.2).
- Cases where units are **few or actually dependent** — just loop inline; the fan-out round-trip adds latency for no gain.
- **Units whose input is dominated by one shared corpus** — fan out by *batch* instead, or run inline (§14.1.1).

The **execution-path / scope constraint each skill already enforces still applies** to every worker. Fanning out does not grant a worker unrestricted repo access — pass it the same bounded scope the main agent would have used.

#### 14.1.1 The shared-corpus trap

Fan-out pays off when each worker's **input** is roughly its own. It backfires when every worker must load the **same large corpus** to do its one unit.

The inline coordinator reads a shared corpus **once** and reuses it across all N units. N workers read it **N times**.

On **cost**, prompt caching softens this: later workers reading an identical prefix bill it at the cache-read rate, so the penalty is well below N× — do not reject a fan-out on token cost alone. On **latency** it does not soften at all, and that is the binding constraint. Each worker still pays the *wall-clock* of streaming that corpus before it can start its own unit, so the **slowest single worker cannot be faster than one corpus load plus one unit** — no matter how perfectly the harness parallelizes. When the corpus load dominates the per-unit work, that floor can exceed the *entire* inline run, and then fan-out is unwinnable by construction. Check the floor before you shard: **estimated slowest worker vs. the whole step run inline.**

**The test, before you shard.** Ask: *would a worker handed only its own unit still have to read most of what every other worker reads?*

- **No** → the inputs are mostly disjoint. Fan out per unit (§14.3). *Examples: one changed file plus its direct imports; one input document to extract from; one source table to profile; one decision needing its own live lookup.*
- **Yes** → the corpus is shared. Do **not** dispatch one worker per unit. Choose:
  1. **Batch-shard** — group the units into `k` batches (`k` ≈ 2–4, or the declared concurrency cap) and give each worker several units, so the corpus load is paid `k` times instead of `N` (§14.3 step 1).
  2. **Run inline** — when `N` is small, the corpus is large, or the per-unit work after loading it is short. The inline loop pays the load once, and that is frequently the fastest option available.

**Worked counter-example (measured, not hypothetical).** A per-story architecture-compliance gate fanned 8 stories to 8 workers. The stories were genuinely independent in logic — each judged unidirectionally against an immutable architecture baseline, no cross-story carry-forward. But every worker had to load the same architecture spec, ADR catalog, tool record, and service specs to judge its one story.

| | wall-clock |
|---|---|
| 8 workers, one story each (harness executed them serially) | **22 min 13 s** |
| Same 8 workers with *perfect* concurrency (modelled from the measured per-worker times) | **≈ 6 min 42 s** |
| **Same step, same inputs, inline** | **2 min 57 s** |

The slowest single worker took **180 s** — while the *entire* inline run, merge and all, took **177 s**. So even zero-serialization fan-out lands ~2.3× behind inline, because the corpus load is inside every worker's critical path and only once in the coordinator's. Fixing the harness's concurrency (§14.7) cuts the 22 min to ~6.7 min; it does not make fan-out the right shape here. Independence of logic was real; independence of input was not.

**Exception — fan-out for decorrelation, not for speed.** Some units are dispatched to separate workers precisely so each judgment is formed *without* seeing the others: **repeated samples of the SAME question** whose mean is the score (mean-of-N judge sampling), or a second opinion that must not be anchored by the first. There the isolation **is** the deliverable's correctness property, and the shared-corpus cost is accepted deliberately. A skill relying on this MUST say so in its callout — otherwise the §14.1.1 test applies normally.

**The exception is narrow — scope it to the units that actually need decorrelating.** It licenses isolating *the samples of one question*, not every unit in a list that merely happens to share a corpus. Two mistakes follow from over-reading it:

- **Fanning out a heterogeneous item list because *some* of its items are judgments.** Scoring item A and item B in one context does not correlate anything: they are different questions. Only *A's own repeated samples* must not see each other. A checklist of N items sharing one artifact is a plain §14.1.1 shared-corpus case, and the corpus test applies to it normally.
- **Dispatching a worker for a DETERMINISTIC unit.** A unit whose result comes from running a verifier, a script, or a query has no judgment to bias and nothing to decorrelate. Run it in the coordinator; wrapping it in a worker buys a context load and no independence.

So a mixed unit set splits: deterministic units run inline, and only the sampled-judgment units claim the exception.

---

### 14.2 Worker contract

Each worker receives **one bounded unit** and the same scope constraint, and returns a **compact structured result** (the per-unit finding / score / file path / decision — not a narrative, not a file dump). Workers are **forbidden** from:

- Writing the canonical `_progress.json`, `00-index.md`, or the carry-forward index (single-writer, coordinator-only).
- Emitting the §11 harness output sidecar (exactly one sidecar per run, written last by the coordinator).
- Making a judgment the skill is accountable for (the verdict, the ranking, the final merge) — workers produce facts/units, the coordinator decides.

A worker MAY write **its own unit's bounded output** within scope (e.g. one story's test file, one section's stub body, one file's REVIEW fragment) and MAY write a **progress shard** (§14.2.1). Everything else is the coordinator's.

#### 14.2.1 Sharded progress

Workers may write `_progress.<worker-id>.json` shards (never the canonical `_progress.json`). The coordinator merges every shard into the canonical `_progress.json` as part of its serial write pass. On **cancel / CONTINUE re-entry**, the coordinator treats a unit with no shard result (or an incomplete shard) as **pending** and re-dispatches only those — the canonical `_progress.json` remains the single source of truth for what is done (§2 / §10.6).

---

### 14.3 Coordinator contract

The coordinator is the main agent. It performs, in order:

1. **Shard** — enumerate the independent units and their bounded inputs (declare the unit set; §14.1). **Apply the §14.1.1 input test here, before dispatching anything.** If the units' inputs are mostly disjoint, one unit per worker. If they are dominated by a shared corpus, **group the units into `k` batches** (`k` ≈ 2–4, or the declared concurrency cap) and treat *a batch* as the unit a worker receives — so the corpus load is paid `k` times, not `N`. If `N` is small or the corpus is large relative to the per-unit work, do not dispatch at all: run the inline loop (§14.5 shape) and say so.
2. **Dispatch** — hand each worker one unit (or one batch) + the scope constraint + the compact return schema. **When using concurrent foreground workers, emit every worker spawn for the shard in a _single_ dispatch turn — one message carrying N worker calls. Spawning one worker per turn (dispatch, await, dispatch the next) serializes them and forfeits the entire fan-out benefit: wall-clock stays _sum-of-units_ instead of collapsing toward _slowest-single-unit_.** This single-turn batching **is** the mechanism that makes foreground dispatch concurrent — without it the run is inline with extra overhead. If a max-concurrency cap is declared (e.g. bounded DB connections in `db-parity-execution`), emit in batches of that cap, **each batch as one turn** — never one worker per turn.
3. **Check that dispatch actually parallelized (§14.7)** — a correctly emitted batch does not guarantee concurrent execution. If the harness serialized it, stop dispatching and finish the remaining units inline.
4. **Collect + verify** — gather worker results; re-verify any `file:line` or asserted fact before it enters the deliverable (§14.4).
5. **Merge (serial, coordinator-only)** — apply the declared **merge contract**: mean-of-N for judge samples, concat + dedupe for per-file findings, per-story files into the tree, per-section stubs into the spec. Apply any bias rules (e.g. bias-toward-FAILED, anti-gaming) here — never in a worker.
6. **Update trackers** — merge progress shards into the canonical `_progress.json`, update `00-index.md` / carry-forward index.
7. **Emit the §11 sidecar last** — exactly one, after all merges (§11.4 ordering is unchanged).

The merge contract is **declared by the skill** (engineering-skills P11) so the aggregation is auditable and identical whether the units ran in parallel or inline.

---

### 14.4 Verify before use

Worker output is **input, not ground truth** — identical to §12.4. The **Zero-Invention Policy governs the coordinator's deliverable**: before a worker's `file:line`, fact, or unit result enters the merged output, the coordinator confirms it (open it / re-check it). A worker hallucination that flows unverified into the deliverable is still an invention by the skill.

---

### 14.5 The inline loop — fallback, and sometimes the correct default

**Three distinct paths reach the inline loop, and only the first is a "fallback":**

1. **No delegation capability** — the harness cannot dispatch multiple workers in a single turn by any means (no concurrent foreground workers **and** no background/detached tasks). It does **not** apply merely because (a) background/detached tasks are absent while concurrent foreground works (§14 Applicability), or (b) no `Parallel delegation:` line is advertised (absent ≠ UNAVAILABLE — determine capability directly).
2. **§14.1.1 says inline is cheaper** — the units share a corpus and N is small or the corpus dominates the per-unit work. Here inline is the **instructed default**, not a degradation: the capability is present and you are declining to use it because using it would cost more. Do not warn or apologize for taking it.
3. **§14.7 aborted the fan-out** — the batch was dispatched correctly and the harness serialized it, so the remaining units finish inline.

In all three cases the skill processes the **same unit set inline, in declared order**, exactly as before this section existed.

§14 is **never a requirement and never changes the deliverable**: the output MUST be identical whether the units ran in parallel or in the inline loop. Do not block, warn, or degrade when the capability is genuinely absent — just loop.

---

### 14.6 Skill-Level Enforcement

Each SKILL.md routed to §14 includes a short callout at the step that owns the unit loop, pointing here. Recommended callout shape:

> **During <the unit-loop step> — apply execution-protocol.md Section 14 (Parallel Fan-Out) if your harness can dispatch concurrent workers.**
>
> Independent units for this skill: <one or two skill-specific examples: "each JUDGE item's N samples", "each changed file's findings", "each Independent story">. Independence: <one line on why the units share no carry-forward>. **Input shape (§14.1.1): <"mostly disjoint — each worker reads only its own unit" → dispatch one unit per worker | "shared corpus — every worker needs <the corpus> " → batch-shard into k≈2–4 batches, or run inline when N is small | "isolated deliberately for decorrelation, cost accepted">.** Merge contract (coordinator, serial): <mean-of-N / concat+dedupe / per-file section / per-story file>. Workers return compact results only and never write `_progress.json`, `00-index.md`, or the §11 sidecar. **Dispatch all workers for the shard in ONE turn (§14.3 step 2) — one-per-turn serializes them — and use detached dispatch + join when the harness offers it, since some harnesses accept a batch and still execute it serially (§14 Applicability).** Fan out whenever the harness can spawn multiple workers in a single turn — **concurrent foreground alone qualifies; background/detached tasks are NOT required, and an absent `Parallel delegation:` line does NOT mean inline** (§14 Applicability). **If the workers did not actually overlap in time, stop dispatching and finish the remaining units inline (§14.7).** Fall back to the inline declared-order loop only when the harness is genuinely single-dispatch — output is identical either way. Verify any worker `file:line` before use (Zero-Invention, §14.4).

The callout is intentionally short. The when/when-not rules, worker + coordinator contracts, sharded progress, verify duty, and fallback all live in Section 14 — the SKILL.md only names the skill-specific unit, independence proof, and merge contract.

---

### 14.7 Did the batch actually overlap? — check, then switch mode or finish inline

**A correctly emitted batch does not prove the harness ran it concurrently** (§14 Applicability). Everything upstream can look right — advertisement AVAILABLE, all N calls in a single turn — and the run still be *sum-of-units*, with each worker's context load added on top. On a shared corpus (§14.1.1) that is strictly worse than never having fanned out.

**Detect it with the one number you actually get.** Do *not* try to observe "worker *k* started only after *k−1*" — a coordinator sees nothing until the batch returns, so start order is invisible. What you do get is **each worker's own elapsed time** (harnesses report per-worker duration in the worker result or the join result) and **your own wall-clock for the shard**. Compute:

```
overlap ratio = (sum of per-worker elapsed times) / (wall-clock for the whole shard)
```

- **ratio ≈ 1 or below** → **serial**. The workers took as long added up as the shard took, so they did not overlap.
- **ratio ≈ N** (approaching the worker count) → **genuinely concurrent**.

Calibration from measured runs: **~0.8 for a serialized batch, ~1.7–2.8 for a concurrent one.** A ratio near 1 with more than 2 workers means no overlap, whatever the capability line claimed. If per-worker durations are genuinely unavailable, use the cheaper probe instead: dispatch **2–3 units first**, measure, then decide the shape for the rest.

**When the ratio says serial, in this order:**

1. **If the harness offers a detached-dispatch + join pair, switch to it** (§14 Applicability) and re-dispatch the *remaining* units that way. This is the fix that recovers the speedup — foreground was the wrong mode, not fan-out itself. Do not re-emit the same foreground batch; it will serialize again.
2. **Otherwise stop dispatching and finish inline.** Keep the units already completed — they are valid; verify them per §14.4 and merge them normally, never discard finished work to "redo it inline." Process every remaining unit inline in declared order per §14.5. You already hold the shared corpus in context, so the inline remainder is the cheap path.
3. **Record it once** in the run's audit/log line — capability advertised, dispatch shape used, observed overlap ratio, what you switched to. One line. This is what turns a silent latency problem into a fixable harness finding.
4. **Do not warn, block, degrade, or change the deliverable.** §14.5's guarantee is unchanged: the output MUST be identical whether the units ran concurrently, serially, in batches, or inline.

**Never encode a harness's dispatch behavior into a skill.** Do not name the harness, its worker tool, or any tool parameter (a background/detached flag included) in a SKILL.md — that breaks the harness-agnostic mandate and makes the skill wrong on the next executor. §14.7 is written as *observed behavior* and *capability shape* precisely so it binds on any harness. Fixing the harness — or fixing what it advertises — is the orchestrator's job, not the skill's.

---
