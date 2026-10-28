> **Execution Protocol section file — §9 Non-Interactive Execution.** Routing + universal sections (§2, §10.5, §11) live in [`execution-protocol.md`](../execution-protocol.md). § numbering is preserved.

## 9. Non-Interactive Execution

When invoked by an automated capability step, this skill runs in non-interactive mode:
- Do NOT ask clarifying questions or pause for human input
- Make reasonable assumptions based on the provided parameters
- Document any assumptions in the output under `## Assumptions` or in `open_questions`
- If blocked by missing required information, write a Gap Report and EXIT — do not improvise

---

