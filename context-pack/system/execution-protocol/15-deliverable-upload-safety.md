> **Execution Protocol section file — §15 Deliverable Upload Safety.** Routing + universal sections (§2, §10.5, §11) live in [`execution-protocol.md`](../execution-protocol.md). § numbering is preserved.

## 15. Deliverable Upload Safety

**Purpose.** A skill that produces a web-shaped deliverable (`.html`, `.svg`, `.js`) does not
finish when the file is written — it finishes when the file **reaches the person it was made
for**. Those files are uploaded to a platform that sits behind a byte-scanning web application
firewall (WAF). The WAF inspects the raw bytes of the upload and rejects anything that *looks
like* an attack payload, before the request ever reaches the application. A file that renders
perfectly in a browser can still be undeliverable.

**The consequence for you as the author.** The WAF does not parse HTML, does not parse
JavaScript, and does not know what your file means. It matches text. That has three
implications people get wrong:

1. **It is not about code.** The scan covers the whole file — prose, comments, sample data,
   table cells, a paragraph of documentation. A design doc *describing* an attack pattern is
   blocked exactly like an executable one.
2. **Intent is irrelevant.** A confirmation dialog written by a designer and an injected XSS
   proof-of-concept can be byte-identical. The WAF is not wrong to block it; the file is wrong
   to contain it.
3. **The failure is silent and misattributed.** The upload returns `403`, nothing appears in
   the application logs, and the obvious suspects (file size, extension, permissions, the
   upload feature itself) are all innocent. Hours get spent in the wrong place.

**Applicability.** Every skill whose declared outputs include a `.html`, `.svg`, or standalone
`.js` file delivered through the platform. Currently: `rapid-prototyping`, `glob-presentation`,
`producer-visualizing-architecture`, `producer-visualizing-story-map`. Markdown-only skills are
out of scope — but see §15.4.

---

### 15.1 Recognising the failure

If an upload returns **`403` with `Content-Type: text/html`**, the request never reached the
backend. The application is a JSON API: every error it produces is `{"detail": "..."}` with
`Content-Type: application/json`. An HTML-bodied 403 is the firewall's own block page.

| Signal | Meaning |
|---|---|
| `Content-Type: text/html` on a 403 from a JSON API | The WAF answered, not the app |
| `X-CDN: Imperva` / `X-Iinfo: ...` response headers | WAF-generated |
| ~779-byte body containing an incident ID | Standard block page |
| Nothing in the application logs | Confirms the request was never proxied |

**Do not** open a platform bug, ask for the upload feature to be checked, or request that the
firewall rule be relaxed. The rule is doing its job; the fix is in the file.

---

### 15.2 Forbidden byte patterns

**The rule: never emit text that is byte-identical to a textbook attack payload, regardless of
intent, and regardless of whether it sits in code, in mock data, or in prose.**

| Pattern | Rule | Why |
|---|---|---|
| `;` followed by a bare `alert(` | **Never.** Confirmed to block uploads. | Identical to a chained-XSS proof of concept. The `;` is what makes it match — the same `alert(` without a preceding `;` passes. |
| `' OR 1=1`, `" OR "1"="1`, `UNION SELECT`, `DROP TABLE`, `--` following a quote | **Never** in literals, sample data, table cells, or narrative text. | Classic SQL-injection signatures. |
| `<script>` written inside a *string* or a prose example | Avoid. | Reflected-XSS signature; harmless as real markup, suspicious as data. |
| Several bare statements chained after `;` on one line generally | Avoid. Prefer one statement per line. | Chaining is the shape the signatures key on; it also reads worse. |

**Two clarifications that matter.** The list is a floor, not a ceiling — firewall rules change,
and the next block will have a signature nobody has written down yet. The durable rule is the
bolded one above, not the table. And the `;` in the first row is load-bearing: authors who
"fixed" the problem by keeping the payload and deleting a space have not fixed anything.

---

### 15.3 Authoring rules

**Do not use native browser dialogs — `alert()`, `confirm()`, `prompt()` — in a delivered
artifact.** This is a deliverable-quality rule first and an upload rule second. A native dialog
is unstyled, blocks the page, ignores the design system, and displays the page origin to the
viewer. In a prototype or deck shown to a client it reads as unfinished work. Use the
artifact's own in-page primitive instead: a modal, a toast, or an inline message. Doing so
sidesteps §15.2 entirely, so a skill that ships a dialog helper in its shell template rarely has
to think about this section at all.

**If a native dialog is genuinely unavoidable**, any one of these is sufficient to clear the
pattern — but the rule above still stands, so record why in the audit:

| Approach | Example |
|---|---|
| Qualify the call | `hideDialog('m'); window.alert('Saved')` |
| Put it on its own line | `hideDialog('m')` ⏎ `alert('Saved')` |
| Put it first in the sequence | `alert('Saved'); hideDialog('m')` |
| Move it into a named function | `onclick="confirmRetry()"`, body in `<script>` — provided that body does not itself chain the two statements on one line |

**For sample and mock data**, use values that are obviously synthetic and structurally harmless.
When the artifact's subject genuinely *is* injection — a security dashboard, a pentest finding,
a validation-rules screen — describe the payload rather than reproducing it (`a tautology-based
SQL injection in the search parameter`), or break it so it cannot match (`' OR 1` + `=1`
rendered as two adjacent elements). Never paste a working payload to make a mock look realistic.

---

### 15.4 Pre-delivery scan — mandatory gate

Run this before the §11 sidecar write, on **every** web-shaped file the run produced:

```bash
grep -nE ';[[:space:]]*alert[[:space:]]*\(' <file>
```

| Result | Action |
|---|---|
| No output | Passes. Proceed to the sidecar. |
| Any hit | Fix each hit per §15.3, then re-run the scan. |
| Still hitting after the fix | Record a blocker in the audit and in `_progress.json`. **Do not report the deliverable as complete** — a file that cannot be uploaded has not been delivered. |

Insert the scan into the finalize ordering, never after the sidecar:

```text
outputs verified → §15.4 upload-safety scan → Memory Bank → _progress.json COMPLETED → §11 sidecar → final_response
```

Log the scan result in the audit (`Upload-safety scan: PASS` / `PASS after N fixes`) so a
reviewer can see it ran. A silent pass is indistinguishable from a skipped step.

**Markdown deliverables.** Out of the mandatory gate, because they are usually consumed
in-repo. But a markdown report that quotes an attack payload — a pentest finding, a security
review, a WAF post-mortem — becomes undeliverable the moment someone uploads it. If your
skill's report may quote payloads, run the same scan and apply §15.3's describe-don't-reproduce
rule.

---

### 15.5 Optional — verify against the live firewall

Only when a file passes §15.4 but an upload still fails. This asks the firewall for a verdict
against a non-existent project with no credentials: nothing is created and nothing is stored.

```bash
curl -sS -o /dev/null -D- -X POST \
  "https://api.beta.glob.ai/api/v1/projects/00000000-0000-0000-0000-000000000000/deliverables" \
  -F "files=@<file>;type=text/html" | grep -iE '^(HTTP/|content-type:|x-iinfo:)'
```

- `401` + `application/json` → the file cleared the firewall. The 401 is expected — there is no token.
- `403` + `text/html` → still blocked; the `x-iinfo` header names the rule that fired.

Each run generates a firewall incident. Use it to diagnose a specific file, not as a routine
step in the finalize sequence — §15.4 is the routine check.
