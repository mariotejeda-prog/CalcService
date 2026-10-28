# ABAP Code-to-Spec Artifact Contract

Version: 1.1.0

This contract is the acceptance specification for `sap-abap-code-to-spec`.
The generator must satisfy it and `verifying-artifacts` must score it. A missing
section is a defect, even if the correct conclusion for that section is that ARC-1
could not retrieve the required evidence.

## ABAP-CODE-TO-SPEC.md required structure

The document MUST contain each of the following H2 headings exactly once, in this
order. It may include additional subordinate headings, tables, diagrams, and
appendices, but must not rename or omit these headings.

## 1. Scope and Target Resolution

State the requested SAP system/client, original selector, resolved object identity
and type, package, and resolution result. For a wildcard selector, list the candidate
set and explain why the selected object is unambiguous. If it is ambiguous or absent,
state that outcome and do not fabricate a target.

## 2. Purpose and Business Context

Explain the observed technical purpose and user/business outcome. Keep supplied
business context separate from facts observed in SAP.

## 3. Evidence Basis and Confidence

List the retrieved objects and source locations that support the specification,
the analysis verdict, and a confidence assessment. Distinguish observed facts,
inferences, and gaps.

## 4. Component and Architecture Context

Describe the target's role, package/component boundary, superclasses, interfaces,
includes, enhancements, and material callers/callees.

## 5. Public Interface and Entry Points

Document public methods, function-module or report entry points, transactions/jobs,
input/output contracts, and preconditions that are evidenced.

## 6. Execution Flows

Describe the normal path and meaningful alternate paths from entry point through
major collaborators. Include dynamic dispatch/calls as explicit gaps if unresolved.

## 7. Business Rules and Validation

List code-derived rules, validations, decision conditions, and configuration-driven
behavior. Each rule must identify its supporting source evidence.

## 8. Data Model and Persistence

Describe DDIC tables, structures, views, CDS/RAP artifacts, read/write behavior,
and important data transformations. State explicit absence when no persistence is
evidenced.

## 9. Integrations and External Dependencies

Describe RFC/BAPI/IDoc/OData/HTTP/file/message dependencies and their direction,
contract, and error boundary when evidenced.

## 10. Authorization and Security Controls

Describe authorization checks, user-context handling, sensitive-data behavior, and
security-relevant gaps. Do not disclose secrets or credentials.

## 11. Transaction Boundaries and Side Effects

Describe commits, update tasks, enqueue/dequeue, database writes, messages, jobs,
and other irreversible or externally visible effects. State explicit absence when
none are evidenced.

## 12. Error Handling and Operational Behavior

Describe exceptions, return-code/message handling, logging, retries, and failure
propagation that are evidenced.

## 13. Dependency and Impact Map

Summarize incoming and outgoing dependencies, including where-used impacts and
the confidence/coverage of the map.

## 14. Gaps, Assumptions, and Recommended Validation

List missing source, inaccessible dependencies, dynamic behavior, unverified
inferences, and the precise SAP evidence or SME validation needed to close each gap.

## ABAP-CODE-EVIDENCE.json required shape

The JSON evidence ledger MUST be valid JSON with these top-level keys:

- `schema_version`
- `sap_system`
- `requested_selector`
- `resolution`
- `analysis_verdict`
- `retrievals`
- `objects`
- `relationships`
- `claims`
- `gaps`
- `confidence`

Each substantive `claims` entry MUST identify its claim text, classification
(`observed` or `inference`), and one or more supporting retrieval/object/source
references. Each gap MUST name the missing or unresolved item and the impact on
confidence or coverage.

## Optional artifact: dependency-graph vault

An implementation MAY additionally emit an Obsidian-renderable dependency-graph
vault. This artifact is OPTIONAL: its absence is not a contract defect, and its
presence does not change how the mandatory deliverables above are scored.

Rules when the vault is produced:

- It MUST be a deterministic, LLM-free projection of the `objects` and
  `relationships` arrays already present in `ABAP-CODE-EVIDENCE.json`, generated
  by the `sap-abap-projecting-graph-vault` skill's reference emitter
  (`.agents/skills/sap-abap-projecting-graph-vault/references/emit_vault.py`).
- It MUST be regenerated in full on every run so it can never drift from the
  evidence ledger. It MUST NOT introduce any claim that is not already evidenced
  in the ledger.
- It MUST write one Markdown page per analysed object with YAML frontmatter
  (`type`, `sap_type`, `sap_name`, `package`, `role`, `doc_level`, `depends_on`),
  render each relationship as a `[[wikilink]]` labelled by its edge kind and
  detail, stub each dangling (external or synthetic) target as an
  `external-reference` page with `[ ] | # ^` characters sanitised, and include an
  `_index.md` overview page.

This section adds a supplementary rendering only. It does NOT rename, reorder, or
omit any of the 14 mandatory `ABAP-CODE-TO-SPEC.md` H2 headings, and it does NOT
alter the 11 required `ABAP-CODE-EVIDENCE.json` top-level keys.
