> **Execution Protocol section file — §3 IMPL-STATE Schema.** Routing + universal sections (§2, §10.5, §11) live in [`execution-protocol.md`](../execution-protocol.md). § numbering is preserved.

## 3. IMPL-STATE Schema (implementing-code only)

Single consolidated tracking file. Written at Phase A as skeleton tables (header + separator
rows only — no example data). Rows appended after each file write using EOL append mode —
never rewrite or replace the whole file. Use Edit tool to insert at end of table.

```
IMPL_INDEX = {
  session_id: string,
  project_name: string,
  source_path: string,
  project_root: string,             // same as source_path
  mode: "STANDARD" | "REPAIR",
  input_format: "AGENT_NATIVE" | "LEGACY",
  active_phase: string,
  output_contract: {
    progress_folder_path: string,
    impl_state_file: string,
    source_path: string,
    expected_outputs: [string]
  },
  phases: [{ id, status, started_at, completed_at, files_count, tests_count }],
  files_touched: [{ phase, path, action, status, test_file }],
  repair_log: [{ iteration, phase, feedback, files_modified: [{ path, change_summary }] }],
  blockers: [{ phase, description, timestamp, resolution }],
  deviations: [{ phase, description, reason, impact }],
  build_commands_discovered: [],
  tech_stack_detected: {},
  tool_results: {},
  execution_log: [],
  metrics: {},
  validations: {},
  open_questions: []
}
```

**BLOCKING gate:** Do NOT generate the next file until the files_touched append has been
confirmed as a completed tool call. Batching updates = lost progress on interruption.

### 3.1 IMPL-STATE Write Budget (anti-heartbeat-churn)

IMPL-STATE writes are limited to:
  (a) skeleton write at Phase A start,
  (b) one append per file completed in Phase B (already required by the §3 BLOCKING gate),
  (c) one write at each phase boundary,
  (d) one final write at end.

Total writes per run MUST satisfy:

    writes_to_IMPL_STATE  <=  files_touched + phases + 2

All Phase B writes MUST be APPEND-MODE row inserts into `files_touched`.
Full-file rewrites of IMPL-STATE between sub-phases are FORBIDDEN.

Self-check before any IMPL-STATE write:

    heartbeat_ratio = existing_IMPL_STATE_writes / source_or_test_files_written_so_far
    IF heartbeat_ratio > 2.0 → skip this write. Resume only after the next
    source/test file is written.

Rationale: every IMPL-STATE write consumes one agent iteration. Excess
heartbeats are the leading cause of iteration-cap exhaustion before the
final sub-phase of medium-sized implementations.

---

