# ABAP RAP Services Artifact Contract

Version: 1.0.0

This contract is the acceptance specification for `sap-abap-rap-services`. It
governs two deliverables: the **RAP BUILD PLAN** (produced before any live write,
approved at a human gate) and the **RAP RUN MANIFEST** (produced after the live
build, recording the ground-truth outcome). Unlike a reverse-engineering doc, the
acceptance ground truth for this capability is the LIVE result — activation, the
Gherkin-derived ABAP Unit tests, ATC, and a published OData service — captured in
the run manifest. A missing section is a defect, even when the correct conclusion
for that section is that the step could not complete (e.g. a transport was
unavailable) — the negative outcome must be stated, not omitted.

## RAP-BUILD-PLAN.md required structure

Produced by `planning-rap-build` from the spec + Gherkin bundle. The document MUST
contain each of the following H2 headings exactly once, in this order. It may add
subordinate headings, tables, and diagrams, but must not rename or omit these.

## 1. Overview

Business object(s) in scope, source bundle path, system type (BTP vs on-prem),
draft decision, provider contract, and a one-line purpose. If the bundle covers
entities out of `entity_scope`, state what was excluded.

## 2. Entity Model

Per entity: every field with name, ABAP type, and classification (key / admin /
business). Each business field and rule MUST trace to the spec prose or a Gherkin
scenario. Relationships and compositions between in-scope entities are listed.
Inferred types are marked as inferred.

## 3. CDS Reuse

Every reused or standard CDS view the plan builds on, confirmed to exist on the
system (with its read-back shape), and every entity built from a new table. A
claimed reuse target that does NOT exist on the system MUST be flagged here and in
Open Questions — never silently replaced by a new table.

## 4. Artifact Stack

The full ordered table of objects to create (type, name, description) in
dependency order, including the draft table when draft is enabled. Names follow
the SAP `ZI_`/`ZC_`/`ZSD_`/`ZSB_`/`ZBP_I_` conventions.

## 5. Behavior Contract

The declarations the interface BDEF must carry: CRUD operations, draft actions
(if draft), and every determination, validation, and action derived from the
Gherkin traceability. Every behavior in Section 6 MUST appear here so the stack
compiles.

## 6. Gherkin Traceability

A matrix mapping every Gherkin scenario to its behavior kind (CRUD /
determination / validation / action), target artifact and method, trigger,
fields, and the enforced rule. Each scenario is classified CRUD
(framework-provided; passes at the red baseline) or logic (needs implementation;
fails until `generate-rap-logic` runs). Any scenario that cannot be mapped MUST be
listed in Open Questions.

## 7. Package & Transport Readiness

Target package, transport status/decision (supplied / existing options / to be
created at build time / not required for `$TMP`), and an explicit ATC-skip warning
when the package is `$TMP` or local (ATC skips local objects).

## 8. Open Questions

The single consolidated registry of ambiguities, missing reuse targets, unmapped
Gherkin scenarios, contradictory spec-vs-Gherkin facts, and every
`default-applied` choice the reviewer should confirm. An empty registry is stated
explicitly as "none".

## rap_build_plan JSON required shape

The plan MUST be accompanied by a machine-readable `rap_build_plan` JSON sidecar so
downstream steps consume the stack and traceability without re-parsing prose. Valid
JSON with these top-level keys:

- `schema_version`
- `sap_system`
- `system_type`  (`btp` | `onprem`)
- `target_package`
- `transport`  (resolved value, existing options, `to_create`, or `not_required`)
- `entities`  (each: name, artifact names, fields with type + classification + source trace, draft flag)
- `cds_reuse`  (each: cds_name, exists boolean, role)
- `artifact_stack`  (ordered: type, name, description)
- `behavior_contract`  (per entity: crud, draft_actions, determinations, validations, actions)
- `gherkin_traceability`  (each: scenario, kind, target_method, trigger, fields, enforced_rule, classification)
- `open_questions`

Each `gherkin_traceability` entry classified `logic` MUST name a target method that
also appears in `behavior_contract`. Each entity's business fields MUST carry a
source trace to the spec bundle.

## RAP-RUN-MANIFEST.md required structure

Produced after the live build (across the generate-stack, generate-tests, and
implement-logic steps) and finalized at verification. It records the ground-truth
outcome. MUST contain each H2 heading exactly once, in order.

## 1. Build Outcome

Which artifacts were created and activated, in order, with per-object status
(created / activated / failed) and the transport used. A failed object states the
error, not silence.

## 2. Test Results

The Gherkin-derived ABAP Unit run: the red baseline (before logic) and the final
run (after logic), per test method, mapped back to its Gherkin scenario, with
pass/fail. Every `logic`-classified scenario MUST end green; any that does not is a
defect stated here.

## 3. ATC Results

The ATC run (variant, e.g. `ABAP_CLOUD_READINESS`) grouped by priority. When the
package is `$TMP`/local, state that ATC was skipped for local objects and why.

## 4. Published Service

The service binding name, `published` status, binding type, and the OData service
URL from the `SAPRead(type="SRVB")` verification. If publish failed, state the
failure and the runtime consequence.

## 5. Open Issues

Any residual failures, skipped checks, or reviewer-facing risks. Empty is stated
explicitly as "none".

## rap_run_manifest JSON required shape

Valid JSON with these top-level keys:

- `schema_version`
- `sap_system`
- `target_package`
- `transport`
- `created_objects`  (each: type, name, status, error?)
- `test_results`  (each: method, scenario, kind, red_baseline, final, status)
- `atc`  (variant, findings_by_priority, skipped boolean, skip_reason?)
- `service_binding`  (name, published boolean, binding_type, service_url?)
- `open_issues`

## Optional artifact: abapGit source mirror

An implementation MAY additionally emit an abapGit-style source mirror of the final
RAP stack into `output_folder` via `SAPGit`. This artifact is OPTIONAL: its absence
is not a contract defect (the mirror step runs with `on_failure: continue`), and its
presence does not change how the mandatory deliverables above are scored.

Rules when the mirror is produced:

- It MUST reflect the final ACTIVATED stack (post-implement-logic), so the mirrored
  source matches what is live on the system.
- It MUST NOT introduce any object not present in `rap_run_manifest.created_objects`.
- It is a reviewable, git-trackable snapshot only — the live system remains the
  source of truth; the mirror is a convenience for diffing and review.

This section adds a supplementary snapshot only. It does NOT rename, reorder, or omit
any mandatory `RAP-BUILD-PLAN.md` or `RAP-RUN-MANIFEST.md` H2 headings, and it does
NOT alter the required `rap_build_plan` or `rap_run_manifest` top-level keys.
