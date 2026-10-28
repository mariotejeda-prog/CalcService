> **Execution Protocol section file — §12 Delegated Exploration.** Routing + universal sections (§2, §10.5, §11) live in [`execution-protocol.md`](../execution-protocol.md). § numbering is preserved.

## 12. Delegated Exploration

**Purpose.** Broad, read-only exploration — locating which files implement a feature, mapping every call site of a symbol, diffing two branches, enumerating what a PR changed, surveying naming conventions across a module, or sweeping a large set of input artifacts (briefs, PRDs, epics, prior specs) — can be **delegated to a read-only exploration subagent** instead of being done inline by the main agent. The subagent reads broadly and returns a compact conclusion; the main agent keeps its context lean for the synthesis and writing it alone is responsible for. This is an **optimization of the input-loading / research phase**, not a new deliverable.

**Why it helps.** It directly serves the same goal as §5 (FIC context monitoring): a fan-out read that would otherwise fill the main context with file contents instead returns as a short summary, so the main agent stays in its healthy utilization band and can do more synthesis before any compaction. Running the sweep on a cheaper/faster model (see §12.3) also makes exploration materially cheaper than reading every file in the main session.

**Applicability.** Any skill whose route includes §12. This section is **capability-gated and harness-agnostic**: it applies *only if the running harness exposes a read-only exploration subagent* (the mechanism and its name differ across harnesses — a built-in "explore"/"search" agent, a task/agent spawn tool, etc.). If the harness exposes no such capability, follow §12.5 (inline fallback) and change nothing else. **Never name a specific tool, agent, or model ID in a skill** — describe the capability and let each harness bind it.

**When this section applies in the run order.** Exploration happens during input loading / research, *before* spec generation (§10) or review. In the routing rule §12 therefore sits early — after §2, before §10 / §10.5.

---

### 12.1 When to delegate (and when not)

**Delegate when** the work is a broad, read-only sweep where you need the *conclusion*, not the raw contents:
- "Which files implement / touch capability X?"
- "Find every call site / consumer of symbol Y."
- "What changed in branch A compared to branch B / on this PR?"
- "What are the naming and structural conventions across this module?"
- "Across these N input artifacts, which ones mention requirement Z, and where?"
- Several independent search angles that can run in parallel.

**Do NOT delegate:**
- The **analysis, root-cause reasoning, decision, or spec-writing** — those are the skill's deliverable and stay with the main agent.
- Work that **mutates files** — the exploration subagent is read-only. Edits, scaffolding, and writes are never delegated through §12.
- Cases where **scope is already a single known file or a tight, named path** — just read it inline; a subagent round-trip adds latency for no context savings.
- Anything that would have the subagent make a **judgment the skill is accountable for** (e.g. "decide the fix", "rank the options"). Ask it for facts and locations, not verdicts.

The **execution-path / scope constraint each skill already enforces still applies** — delegating does not grant the subagent unrestricted repo access. Pass it the same bounded scope the main agent would have used.

---

### 12.2 The return contract

Instruct the subagent to return **conclusions plus `file:line` pointers and a compact summary — never file dumps or pasted source**. A good delegated prompt:
- States one bounded, specific question (not "explore the codebase").
- Names the scope / path constraint to stay within.
- Asks explicitly for: the answer, the precise `file:line` references that support it, and a short summary — and asks it NOT to paste large file contents back.

The returned summary is consumed as **research input** to the main session — the same role inline exploration notes would have played.

---

### 12.2.1 Anti-patterns — hard stops, check before dispatch

A delegated call that violates any of these is a **misuse of §12**, not a strict reading of it. Each one converts a cheap local read into an expensive round-trip that lands the same bytes in your context anyway — you pay full subagent latency and save nothing.

**Never dispatch a subagent to:**

1. **Return content verbatim.** Any prompt containing *"return the complete content"*, *"do not summarize"*, *"return verbatim"*, *"read it in chunks and return all of it"*, or equivalent is a **file read wearing a subagent costume**. §12 exists so the dump never enters the main context; asking for the dump defeats the entire mechanism. If you want the bytes, read the file — that is what a read is for.
2. **Read a specific, already-known path.** Delegation answers *"where is X?"* / *"which files do Y?"*. Once you can name the file, the search is already over. Reading a named path inline is strictly faster and cheaper. (Restates §12.1's fourth bullet as a hard stop, because this is the most common violation.)
3. **Re-read an artifact another subagent already covered.** Track what each dispatched call was asked for. Two subagents reading the same document — even for "different sections" — is duplicated latency and duplicated tokens for one document you could have read once. Merge the questions into a single call, or read it inline.
4. **Sweep a corpus you only need a slice of.** When an upstream artifact carries traceability ids (requirement / decision / story references), resolve the ids first and read only what they name. "Extract ALL FR/NFR/KPI ids" from a large spec is a full ingest with a filter bolted on, not a targeted sweep.

**Pre-dispatch self-check** — before every delegated call, answer both:

> *Do I already know the path?* If yes → read it inline, do not delegate.
> *Am I asking for conclusions, or for content?* If content → read it inline, do not delegate.

If a run finds itself with more delegated calls than distinct unknown questions, delegation has become overhead. Stop and read inline.

**Symptom to watch for.** High input-token and cache-read volume with near-zero output tokens over a long wall-clock, before the deliverable has started, means the run is marshalling documents rather than reasoning about them. That is this anti-pattern, not a slow model.

---

### 12.3 Model selection

Exploration is search and retrieval, not synthesis — a **cheaper / faster model tier is sufficient and materially cheaper**. When the harness lets you pick the subagent's model, prefer its fast/economy tier (the harness's equivalent of a "fast" or "mini" model) for §12 sweeps. Reserve the strong model for the synthesis the main agent does. Describe this as a preference for the cheap tier — do not hardcode a model ID, since the available tiers differ per harness.

---

### 12.4 Verify before use

Subagent output is **input, not ground truth**. The **Zero-Invention Policy still governs the main agent's deliverable**: before citing a `file:line` in a spec, basing a fix or plan step on a delegated finding, or asserting a fact the subagent reported, the main agent MUST confirm the pointer actually exists (open it / re-grep it). A subagent hallucination that flows unverified into the deliverable is still an invention by the skill. Treat delegated findings exactly as you would treat any unverified source: confirm, then use.

---

### 12.5 Fallback — no subagent capability

If the harness exposes no read-only exploration subagent, the skill performs the same exploration **inline**, bounded by its execution-path / scope constraint, exactly as before this section existed. §12 is **never a requirement and never changes the deliverable**: a skill must produce identical-quality output whether or not delegation was available. Do not block, warn, or degrade when the capability is absent — simply explore inline.

---

### 12.6 Skill-Level Enforcement

Each SKILL.md routed to §12 includes a short callout at its input-loading / exploration step pointing here. Recommended callout shape:

> **During input loading / exploration — apply execution-protocol.md Section 12 (Delegated Exploration) if your harness supports it.**
>
> Broad read-only sweeps for this skill (e.g. <one or two skill-specific examples: "locating the files on the bug's execution path", "diffing the feature branch against the base", "surveying which input artifacts cover each requirement">) MAY be delegated to a read-only exploration subagent on a cheap/fast model, which returns conclusions + `file:line` pointers (not file dumps). Synthesis, decisions, and all writing stay with this agent, which verifies any delegated `file:line` before using it (Zero-Invention still applies). If no subagent capability exists, explore inline under the usual scope constraint — output quality is identical either way.

The callout is intentionally short. The when/when-not rules, return contract, model preference, verification duty, and fallback all live in Section 12 — the SKILL.md only names the skill-specific sweep examples.

---

