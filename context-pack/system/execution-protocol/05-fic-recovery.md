> **Execution Protocol section file — §5 FIC Context Monitoring + Recovery.** Routing + universal sections (§2, §10.5, §11) live in [`execution-protocol.md`](../execution-protocol.md). § numbering is preserved.

## 5. FIC Context Monitoring + Recovery Checkpoint

Monitor context utilization throughout execution.

**On hitting 60% context threshold:**
1. IMMEDIATELY write current output state to disk (even if incomplete)
2. Mark incomplete sections: `status: pending — context compaction triggered`
3. Update `_progress.json`: `{ "status": "COMPACTION_NEEDED" }`
4. WRITE Recovery Checkpoint (see format below)
5. LOG: `"FIC ALERT: Context at {N}% — partial state written for session recovery"`

**Recovery Checkpoint** — write to `{progress_folder_path}/RECOVERY-CHECKPOINT-{session_id}.md`:

```
(a) Current phase + current file being worked on
(b) Files COMPLETED in this session (paths only — do NOT re-read them)
(c) Key patterns/decisions from plan and research (compact summary, max 30 lines)
(d) Output file contract: path to _progress.json + expected output parameters
(e) Next action to take when resuming
```

Note: IMPL-STATE tracks status metadata. Recovery Checkpoint tracks WORKING CONTEXT —
what was in memory when compaction hit. They serve different purposes.

**After context continuation (compaction boundary), FIRST actions:**
1. READ Recovery Checkpoint
2. READ IMPL-STATE
3. Resume from the "next action" in the checkpoint
4. Do NOT re-read source files already marked COMPLETED in IMPL-STATE files_touched
5. Re-read ONLY the current file being worked on + plan/research sections for remaining work

---

