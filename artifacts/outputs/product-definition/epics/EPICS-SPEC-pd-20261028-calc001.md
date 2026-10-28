# Math Operations Microservice (Calc-Service) — Epics Backlog Spec
version: 1.0.1
session: pd-20261028-calc001
mode: BUILD
date: 2026-10-28
language: en
prd_source: artifacts/outputs/product-definition/prd/PRD-SPEC-pd-20261027-calc001.md
contract_format: structured
prd_source_hash: 25c7814b3af060433c725d5e0d92674538e4af564960493da328ea935e63f93a
status: draft

---

## project_info
client_name: Internal platform organization (client organization not named in source inputs; service built for internal platform teams)
project_name: Math Operations Microservice (Calc-Service)
project_description: A lightweight, stateless microservice that provides a centralized, deterministic mathematical computation capability for the platform, externalizing calculation logic from client applications to enable consistency, reuse, and controlled evolution of numerical behavior across the ecosystem.

---

## prd_id_ownership

Complete projection of PRD IDs into the epic layer. Every PRD ID appears exactly once as `owned`, `cross_cutting`, or `deferred`.

| prd_id | prd_name | type | disposition | epic_ids | phase | reason | source_ref |
|--------|----------|------|-------------|----------|-------|--------|------------|
| FR-01 | Execute standard arithmetic operations (addition, subtraction, multiplication, division) | FR | owned | EPIC-01 | phase_1_mvp | primary deliverable of EPIC-01 (standard arithmetic calculation core) | PRD:FR-01 |
| FR-02 | Execute advanced operations (exponentiation, roots, trigonometric functions) | FR | owned | EPIC-04 | phase_2 | primary deliverable of EPIC-04 (advanced operations extension) | PRD:FR-02 |
| FR-03 | Produce deterministic, high-precision results | FR | owned | EPIC-01 | phase_1_mvp | determinism guarantee embedded in EPIC-01 (standard arithmetic calculation core) | PRD:FR-03 |
| FR-04 | Expose a calculate operation via `POST /api/v1/math/calculate` | FR | owned | EPIC-02 | phase_1_mvp | primary deliverable of EPIC-02 (calculate API contract) | PRD:FR-04 |
| FR-05 | Return clear, fail-fast errors for invalid input | FR | owned | EPIC-02 | phase_1_mvp | error handling embedded in EPIC-02 (calculate API contract) | PRD:FR-05 |
| FR-06 | Record calculation history as optional, auxiliary local persistence | FR | owned | EPIC-05 | phase_3 | primary deliverable of EPIC-05 (optional history persistence) | PRD:FR-06 |
| FR-07 | Calculation must succeed even if history persistence fails | FR | owned | EPIC-05 | phase_3 | reliability guarantee embedded in EPIC-05 (optional history persistence) | PRD:FR-07 |
| FR-08 | Provide history retrieval of recent calculation records | FR | owned | EPIC-06 | phase_3 | primary deliverable of EPIC-06 (history retrieval) | PRD:FR-08 |
| FR-09 | Support extensibility so new operations are added without breaking existing consumers | FR | owned | EPIC-03 | phase_1_mvp | primary deliverable of EPIC-03 (extensible operation registry) | PRD:FR-09 |
| FR-10 | Operate as a stateless, independently deployable, horizontally scalable service | FR | owned | EPIC-07 | phase_1_mvp | primary deliverable of EPIC-07 (stateless deployable service shell) | PRD:FR-10 |
| NFR-01 | Standard operations respond at millisecond-level latency | NFR | cross_cutting | EPIC-01, EPIC-02 | all | latency DoD constraint on the calculation core (EPIC-01) and its API exposure (EPIC-02) | PRD:NFR-01 |
| NFR-02 | Same input always produces the same result (determinism) | NFR | cross_cutting | EPIC-01, EPIC-02 | all | determinism DoD constraint on the calculation core (EPIC-01) and its API exposure (EPIC-02) | PRD:NFR-02 |
| NFR-03 | Calculation succeeds even when history persistence fails | NFR | cross_cutting | EPIC-05 | phase_3 | availability DoD constraint on EPIC-05 (optional history persistence); behavioral mirror of FR-07 per ASM-05 (OQ-abad2154) | PRD:NFR-03 |
| NFR-04 | Stateless design supports horizontal scaling | NFR | cross_cutting | EPIC-07 | all | scalability DoD constraint on EPIC-07 (stateless deployable service shell) | PRD:NFR-04 |
| NFR-05 | Observability readiness for future metrics and tracing integration | NFR | cross_cutting | EPIC-07 | all | observability-readiness DoD constraint on EPIC-07 (stateless deployable service shell) | PRD:NFR-05 |
| NFR-06 | Security and access control for the calculation API | NFR | cross_cutting | EPIC-02 | all | security DoD constraint on EPIC-02 (calculate API contract); target pending (OQ-8a3647b2) | PRD:NFR-06 |
| NFR-07 | API usability for internal consumers | NFR | cross_cutting | EPIC-02 | all | usability DoD constraint on EPIC-02 (calculate API contract); simple payload baseline per OQ-4d202bf8 | PRD:NFR-07 |
| KPI-01 | Upstream system integration adoption | KPI | cross_cutting | EPIC-02, EPIC-07 | all | first upstream integration (DEP-01, OQ-87c0d49a) is demonstrated through EPIC-02 (calculate API contract) running on EPIC-07 (stateless deployable service shell) | PRD:KPI-01 |
| KPI-02 | Deterministic test coverage of all operations | KPI | cross_cutting | EPIC-01, EPIC-03, EPIC-04 | all | deterministic unit test coverage is the DoD of EPIC-01 (standard arithmetic calculation core), EPIC-03 (extensible operation registry), EPIC-04 (advanced operations extension) | PRD:KPI-02 |
| KPI-03 | History persistence validation | KPI | owned | EPIC-05 | phase_3 | validation run confirming persistence works without impacting response delivery is the acceptance evidence of EPIC-05 (optional history persistence) | PRD:KPI-03 |
| KPI-04 | API contract documentation for internal reuse | KPI | owned | EPIC-08 | phase_1_mvp | published API contract documentation is the deliverable of EPIC-08 (API contract documentation) | PRD:KPI-04 |
| KPI-05 | Reduction of duplicated math logic across teams | KPI | cross_cutting | EPIC-01, EPIC-02, EPIC-08 | all | centralization of math logic (EPIC-01) behind a documented simple API (EPIC-02, EPIC-08) is the mechanism for reducing duplication; numeric target pending (OQ-c5ed68d9) | PRD:KPI-05 |
| KPI-06 | Result consistency across systems | KPI | cross_cutting | EPIC-01, EPIC-02 | all | deterministic identical results from the central engine (EPIC-01) delivered through the API (EPIC-02) produce cross-system consistency; numeric target pending (OQ-c5ed68d9) | PRD:KPI-06 |
| KPI-07 | Calculation traceability availability | KPI | owned | EPIC-05, EPIC-06 | phase_3 | traceability delivered by history recording (EPIC-05) and retrieval (EPIC-06); numeric target pending (OQ-c5ed68d9) | PRD:KPI-07 |
| JTBD-01 | core_functional | JTBD | cross_cutting | EPIC-01, EPIC-02 | all | single authoritative math engine (EPIC-01) consumed through the calculate API (EPIC-02) | PRD:JTBD-01 |
| JTBD-02 | related | JTBD | cross_cutting | EPIC-03 | phase_1_mvp | controlled evolution of numerical behavior without breaking consumers is delivered by EPIC-03 (extensible operation registry) | PRD:JTBD-02 |
| JTBD-03 | related | JTBD | cross_cutting | EPIC-05, EPIC-06 | phase_3 | trace and debug via recorded metadata delivered by EPIC-05 (optional history persistence) and EPIC-06 (history retrieval) | PRD:JTBD-03 |
| JTBD-04 | related | JTBD | cross_cutting | EPIC-02, EPIC-07 | all | lightweight deterministic calculations inside automated workflows served by the calculate API (EPIC-02) on the stateless service (EPIC-07) | PRD:JTBD-04 |
| RSK-01 | Hidden logic differences remain between systems after the service launches | risk | cross_cutting | EPIC-01, EPIC-02 | all | mitigated by deterministic central engine (EPIC-01) and consistent API results (EPIC-02) | PRD:RSK-01 |
| RSK-02 | Teams bypass the service and keep embedding their own calculation logic | risk | cross_cutting | EPIC-02, EPIC-08 | all | mitigated by simple payload design (EPIC-02) and published contract documentation (EPIC-08) | PRD:RSK-02 |
| RSK-03 | History persistence degrades calculation response delivery | risk | cross_cutting | EPIC-05 | phase_3 | mitigated by the persistence-never-blocks-calculation guarantee embedded in EPIC-05 (optional history persistence) | PRD:RSK-03 |
| RSK-04 | Precision policy gaps reintroduce floating-point surprises between systems | risk | cross_cutting | EPIC-01 | all | mitigated by the single high-precision computation strategy embedded in EPIC-01 (standard arithmetic calculation core); policy pending (PI-01) | PRD:RSK-04 |
| ASM-01 | Consumer personas are technically proficient engineering roles | assumption | cross_cutting | EPIC-01, EPIC-02, EPIC-04, EPIC-05, EPIC-06, EPIC-08 | all | persona proficiency underpins API usability prioritization across consumer-facing epics (OQ-74ae3574) | PRD:ASM-01 |
| ASM-02 | The brief's conceptual API contract field names are the intended contract names | assumption | owned | EPIC-02 | phase_1_mvp | field names used verbatim in EPIC-02 (calculate API contract) acceptance criteria (OQ-8501c5fa) | PRD:ASM-02 |
| ASM-03 | History persistence uses a lightweight local embedded database abstraction | assumption | owned | EPIC-05 | phase_3 | persistence abstraction design basis of EPIC-05 (optional history persistence) (OQ-4606eb39) | PRD:ASM-03 |
| ASM-04 | Stateless design is the mechanism that enables horizontal scaling | assumption | cross_cutting | EPIC-07 | all | statelessness is the design basis of EPIC-07 (stateless deployable service shell) (OQ-62a2df8e) | PRD:ASM-04 |
| ASM-05 | The reliability guarantee is stated at both FR and NFR level intentionally | assumption | cross_cutting | EPIC-05 | phase_3 | FR-07 and NFR-03 both verified against the same test in EPIC-05 (optional history persistence) (OQ-abad2154) | PRD:ASM-05 |
| ASM-06 | Synchronous request/response is the only interaction mode required at launch | assumption | cross_cutting | EPIC-02 | all | only synchronous request/response modeled in EPIC-02 (calculate API contract) (OQ-e44e5e82) | PRD:ASM-06 |
| ASM-07 | MoSCoW priorities inferred from source phrasing | assumption | cross_cutting | EPIC-01, EPIC-02, EPIC-03, EPIC-04, EPIC-05, EPIC-06, EPIC-07, EPIC-08 | all | priority tiers and phase mapping of all epics follow the inferred MoSCoW assignment (OQ-b450c1ca) | PRD:ASM-07 |
| DEP-01 | Upstream consumer systems | dependency | cross_cutting | EPIC-02, EPIC-07 | all | at least one upstream system must integrate via EPIC-02 (calculate API contract) on EPIC-07 (stateless deployable service shell); first target pending (OQ-87c0d49a) | PRD:DEP-01 |
| DEP-02 | Embedded persistence abstraction for optional history | dependency | owned | EPIC-05 | phase_3 | embedded persistence abstraction is the technical basis of EPIC-05 (optional history persistence) | PRD:DEP-02 |
| P-01 | Backend Service Developer | persona | cross_cutting | EPIC-01, EPIC-02, EPIC-03, EPIC-08 | all | primary consumer of the calculation core, API, extensibility, and documentation | PRD:P-01 |
| P-02 | Automation Workflow Engineer | persona | cross_cutting | EPIC-02, EPIC-07 | all | synchronous calculations inside automated workflows via the calculate API on the stateless service | PRD:P-02 |
| P-03 | AI Agent / AI-Driven Process Operator | persona | cross_cutting | EPIC-02, EPIC-07 | all | deterministic calculations for AI-driven processes via the calculate API on the stateless service | PRD:P-03 |
| P-04 | Reporting Pipeline Engineer | persona | cross_cutting | EPIC-01, EPIC-02 | all | consistent numerical results across reporting outputs via the deterministic engine and API | PRD:P-04 |
| LIT-01 | calculate endpoint route | literal | owned | EPIC-02 | phase_1_mvp | exact route used by EPIC-02 (calculate API contract) acceptance criteria | PRD:LIT-01 |
| LIT-02 | request field: operand1 | literal | owned | EPIC-02 | phase_1_mvp | exact request field name in EPIC-02 (calculate API contract) | PRD:LIT-02 |
| LIT-03 | request field: operand2 | literal | owned | EPIC-02 | phase_1_mvp | exact request field used by EPIC-02 (calculate API contract) | PRD:LIT-03 |
| LIT-04 | request field: operation | literal | owned | EPIC-02 | phase_1_mvp | exact request field used by EPIC-02 (calculate API contract) | PRD:LIT-04 |
| LIT-05 | response field: resultValue | literal | owned | EPIC-01, EPIC-02 | phase_1_mvp | exact response field produced by EPIC-01 (standard arithmetic calculation core) and exposed by EPIC-02 (calculate API contract) | PRD:LIT-05 |
| LIT-06 | response field: operationType | literal | owned | EPIC-01, EPIC-02 | phase_1_mvp | exact response field produced by EPIC-01 (standard arithmetic calculation core) and exposed by EPIC-02 (calculate API contract) | PRD:LIT-06 |
| LIT-07 | response field: timestamp | literal | owned | EPIC-02 | phase_1_mvp | exact response field exposed by EPIC-02 (calculate API contract) | PRD:LIT-07 |
| LIT-08 | operation enum: ADD | literal | owned | EPIC-01, EPIC-02 | phase_1_mvp | operation enum value exercised by EPIC-01 (standard arithmetic calculation core) acceptance criteria | PRD:LIT-08 |
| LIT-09 | example request values | literal | owned | EPIC-02 | phase_1_mvp | example request/response values used in EPIC-02 (calculate API contract) acceptance criteria | PRD:LIT-09 |
| LIT-10 | latency goal | literal | owned | EPIC-01, EPIC-02 | phase_1_mvp | millisecond-level latency goal carried as DoD constraint on EPIC-01 (standard arithmetic calculation core) and EPIC-02 (calculate API contract) | PRD:LIT-10 |
| LIT-11 | persistence strategy | literal | owned | EPIC-05 | phase_3 | H2-based lightweight persistence strategy referenced by EPIC-05 (optional history persistence) | PRD:LIT-11 |

---

---

## themes

### T-01: deterministic calculation engine
- features:
  - standard arithmetic operations | fr_refs: FR-01 (Execute standard arithmetic operations (addition, subtraction, multiplication, division)), FR-03 (Produce deterministic, high-precision results) | priority: must_have
  - calculate API and error handling | fr_refs: FR-04 (Expose a calculate operation via `POST /api/v1/math/calculate`), FR-05 (Return clear, fail-fast errors for invalid input) | priority: must_have
  - operation extensibility | fr_refs: FR-09 (Support extensibility so new operations are added without breaking existing consumers) | priority: must_have
- status: complete

### T-02: advanced computation capabilities
- features:
  - advanced operations | fr_refs: FR-02 (Execute advanced operations (exponentiation, roots, trigonometric functions)) | priority: should_have
- status: complete

### T-03: calculation history and record lookup
- features:
  - optional history persistence | fr_refs: FR-06 (Record calculation history as optional, auxiliary local persistence), FR-07 (Calculation must succeed even if history persistence fails) | priority: could_have
  - history retrieval | fr_refs: FR-08 (Provide history retrieval of recent calculation records) | priority: could_have
- status: complete

### T-04: platform service foundation
- features:
  - stateless deployable service | fr_refs: FR-10 (Operate as a stateless, independently deployable, horizontally scalable service) | priority: must_have
  - API contract documentation | fr_refs: — | priority: must_have | [ASM: inferred grouping — documentation enabler traces to KPI-04 (API contract documentation for internal reuse), no FR]
- status: complete

---

---

## epics

> **EPIC ID ORDER IS NOT BUILD ORDER.** Ids reflect authoring sequence; foundational
> enablers are routinely numbered last. The authoritative execution order for every
> downstream consumer (sprint planning, decomposition, implementation) is
> `## sequencing → topological_order`, with its edge table in `## dependencies` —
> both are at the END of this file. Do not derive a sequence from the epic list
> below.

### EPIC-01: Standard arithmetic calculation core
- type: feature
- theme: T-01 (deterministic calculation engine)
- personas: P-01 (Backend Service Developer), P-04 (Reporting Pipeline Engineer)
- priority_tier: must_have
- complexity: M
- traceability:
  - fr: FR-01 (Execute standard arithmetic operations (addition, subtraction, multiplication, division)), FR-03 (Produce deterministic, high-precision results)
  - jtbd: JTBD-01 (core_functional)
  - kpi: KPI-02 (Deterministic test coverage of all operations), KPI-06 (Result consistency across systems)
- nfr_dod:
  - NFR-01: millisecond-level response time for standard operations (interim baseline: sub-second, < 1000 ms, per OQ pending on NFR-01)
  - NFR-02: 100% of repeated identical requests return the identical resultValue
- rsk_refs: RSK-01 (Hidden logic differences remain between systems after the service launches), RSK-04 (Precision policy gaps reintroduce floating-point surprises between systems)
- asm_refs: ASM-07 (MoSCoW priorities inferred from source phrasing)
- scope:
  - in: deterministic computation engine implementing addition, subtraction, multiplication, and division with a single high-precision internal computation strategy applied service-wide; deterministic unit test coverage for every standard operation
  - out: advanced operations (EPIC-04 (advanced operations extension)), API exposure (EPIC-02 (calculate API contract)), history recording (EPIC-05 (optional history persistence))
- status: complete

### EPIC-02: Calculate API contract
- type: feature
- theme: T-01 (deterministic calculation engine)
- personas: P-01 (Backend Service Developer), P-02 (Automation Workflow Engineer), P-03 (AI Agent / AI-Driven Process Operator), P-04 (Reporting Pipeline Engineer)
- priority_tier: must_have
- complexity: M
- traceability:
  - fr: FR-04 (Expose a calculate operation via `POST /api/v1/math/calculate`), FR-05 (Return clear, fail-fast errors for invalid input)
  - jtbd: JTBD-01 (core_functional), JTBD-04 (related)
  - kpi: KPI-01 (Upstream system integration adoption), KPI-05 (Reduction of duplicated math logic across teams)
- nfr_dod:
  - NFR-01: millisecond-level response time for standard operations (interim baseline: sub-second, < 1000 ms)
  - NFR-02: 100% of repeated identical requests return the identical resultValue
  - NFR-06: security and access control target pending (OQ-8a3647b2); interim baseline: internal platform component, no additional authentication assumed
  - NFR-07: simple payload design (operands + operation in, result out) as the usability baseline per OQ-4d202bf8
- rsk_refs: RSK-01 (Hidden logic differences remain between systems after the service launches), RSK-02 (Teams bypass the service and keep embedding their own calculation logic)
- asm_refs: ASM-02 (The brief's conceptual API contract field names are the intended contract names), ASM-06 (Synchronous request/response is the only interaction mode required at launch)
- scope:
  - in: synchronous `POST /api/v1/math/calculate` endpoint accepting operand1, operand2, operation and returning resultValue, operationType, timestamp; fail-fast validation errors for division by zero, unsupported operation, and missing or non-numeric operands
  - out: asynchronous or batch interaction modes (excluded per ASM-06 (Synchronous request/response is the only interaction mode required at launch)); history retrieval endpoints
- status: complete

### EPIC-03: Extensible operation registry
- type: feature
- theme: T-01 (deterministic calculation engine)
- personas: P-01 (Backend Service Developer)
- priority_tier: must_have
- complexity: M
- traceability:
  - fr: FR-09 (Support extensibility so new operations are added without breaking existing consumers)
  - jtbd: JTBD-02 (related)
  - kpi: KPI-02 (Deterministic test coverage of all operations)
- nfr_dod:
  - NFR-02: 100% of repeated identical requests return the identical resultValue (new operations inherit the determinism guarantee)
- rsk_refs: —
- asm_refs: ASM-07 (MoSCoW priorities inferred from source phrasing)
- scope:
  - in: operation registry mechanism through which new operations are registered and dispatched, demonstrating that adding an operation requires no change to the calculate contract and no break to existing consumers
  - out: any specific advanced operation (delivered by EPIC-04 (advanced operations extension))
- status: complete

### EPIC-04: Advanced operations extension
- type: feature
- theme: T-02 (advanced computation capabilities)
- personas: P-01 (Backend Service Developer), P-04 (Reporting Pipeline Engineer)
- priority_tier: should_have
- complexity: M
- traceability:
  - fr: FR-02 (Execute advanced operations (exponentiation, roots, trigonometric functions))
  - jtbd: JTBD-01 (core_functional)
  - kpi: KPI-02 (Deterministic test coverage of all operations)
- nfr_dod:
  - NFR-02: 100% of repeated identical requests return the identical resultValue (extended to advanced operations)
- rsk_refs: —
- asm_refs: ASM-07 (MoSCoW priorities inferred from source phrasing)
- scope:
  - in: exponentiation, root, and trigonometric operations added through the operation registry (EPIC-03 (extensible operation registry)) with deterministic unit test coverage; clear error for a requested advanced operation not yet supported
  - out: expression evaluation (out of scope in the PRD)
- status: complete

### EPIC-05: Optional history persistence
- type: feature
- theme: T-03 (calculation history and record lookup)
- personas: P-01 (Backend Service Developer)
- priority_tier: could_have
- complexity: M
- traceability:
  - fr: FR-06 (Record calculation history as optional, auxiliary local persistence), FR-07 (Calculation must succeed even if history persistence fails)
  - jtbd: JTBD-03 (related)
  - kpi: KPI-03 (History persistence validation), KPI-07 (Calculation traceability availability)
- nfr_dod:
  - NFR-03: 100% of valid calculate requests return a result when history persistence is unavailable (behavioral mirror of FR-07, verified against the same test per ASM-05 (OQ-abad2154))
- rsk_refs: RSK-03 (History persistence degrades calculation response delivery)
- asm_refs: ASM-03 (History persistence uses a lightweight local embedded database abstraction), ASM-05 (The reliability guarantee is stated at both FR and NFR level intentionally)
- scope:
  - in: optional, toggleable recording of calculation metadata behind a replaceable embedded database abstraction (LIT-11 (persistence strategy): H2-based lightweight persistence strategy); history write must never delay or block the calculation response; calculation succeeds normally when tracking is disabled or persistence fails
  - out: multi-instance history consistency (out of scope in the PRD); regulatory-grade audit persistence (out of scope in the PRD); event-based history (wont_have in the PRD)
- status: complete

### EPIC-06: History retrieval
- type: feature
- theme: T-03 (calculation history and record lookup)
- personas: P-01 (Backend Service Developer)
- priority_tier: could_have
- complexity: S
- traceability:
  - fr: FR-08 (Provide history retrieval of recent calculation records)
  - jtbd: JTBD-03 (related)
  - kpi: KPI-07 (Calculation traceability availability)
- nfr_dod: —
- rsk_refs: —
- asm_refs: ASM-03 (History persistence uses a lightweight local embedded database abstraction)
- scope:
  - in: retrieval of recent calculation records for diagnostics and auditing, reading from the history store delivered by EPIC-05 (optional history persistence)
  - out: a committed endpoint path and response shape — FR-08 is emitted with status pending and no endpoint contract; history retrieval is excluded from the MVP phase per OQ-025246e3
- status: complete

### EPIC-07: Stateless deployable service shell
- type: enabler
- theme: T-04 (platform service foundation)
- personas: P-02 (Automation Workflow Engineer), P-03 (AI Agent / AI-Driven Process Operator)
- priority_tier: must_have
- complexity: M
- traceability:
  - fr: FR-10 (Operate as a stateless, independently deployable, horizontally scalable service)
  - jtbd: JTBD-04 (related)
  - kpi: KPI-01 (Upstream system integration adoption)
- nfr_dod:
  - NFR-04: service capacity scales horizontally by adding instances with no change to calculation behavior (no numeric concurrency target committed, per OQ pending on NFR-04)
  - NFR-05: observability readiness — structured logging sufficient for diagnostics; metrics and tracing integration deferred (interim baseline per OQ pending on NFR-05)
- rsk_refs: —
- asm_refs: ASM-04 (Stateless design is the mechanism that enables horizontal scaling)
- scope:
  - in: stateless service shell with independent deployment and horizontally scalable computation layer, no user session ownership; verification that identical inputs produce identical results across multiple instances
  - out: concrete deployment platform selection and infrastructure provisioning (architecture-phase concern); metrics and tracing integration (readiness only)
- status: complete

### EPIC-08: API contract documentation
- type: enabler
- theme: T-04 (platform service foundation)
- personas: P-01 (Backend Service Developer), P-02 (Automation Workflow Engineer), P-03 (AI Agent / AI-Driven Process Operator), P-04 (Reporting Pipeline Engineer)
- priority_tier: must_have
- complexity: S
- traceability:
  - fr: —
  - jtbd: —
  - kpi: KPI-04 (API contract documentation for internal reuse)
- nfr_dod:
  - NFR-07: simple payload design (operands + operation in, result out) documented as the usability baseline
- rsk_refs: RSK-02 (Teams bypass the service and keep embedding their own calculation logic)
- asm_refs: ASM-02 (The brief's conceptual API contract field names are the intended contract names)
- scope:
  - in: published API contract documentation for the calculate operation (endpoint, request/response fields, operation enum, error responses) made available to internal consumers
  - out: developer portal or interactive documentation tooling beyond a published contract document
- status: complete

---

---

## dependencies

| blocked | blocked_description | blocker | blocker_description | type | impact | mitigation |
|---------|----------------------|--------|----------------------|------|--------|------------|
| EPIC-01 | Standard arithmetic calculation core | — | — | none | — | — |
| EPIC-02 | Calculate API contract | EPIC-01 | Standard arithmetic calculation core | epic_to_epic | the endpoint exposes operations the calculation core defines; without EPIC-01 there is nothing to calculate | sequence EPIC-01 first; contract design can proceed in parallel against the agreed operation set |
| EPIC-03 | Extensible operation registry | EPIC-01 | Standard arithmetic calculation core | epic_to_epic | the registry dispatches the operations the calculation core implements | build the registry on top of the completed standard operations |
| EPIC-04 | Advanced operations extension | EPIC-03 | Extensible operation registry | epic_to_epic | advanced operations are added through the registry mechanism | deliver EPIC-03 before starting EPIC-04 |
| EPIC-05 | Optional history persistence | EPIC-01 | Standard arithmetic calculation core | epic_to_epic | history records metadata of calculations performed by the calculation core | deliver EPIC-01 first; persistence layer is independent of the API layer |
| EPIC-06 | History retrieval | EPIC-05 | Optional history persistence | epic_to_epic | retrieval requires recorded history to read | deliver EPIC-05 before EPIC-06 |
| EPIC-07 | Stateless deployable service shell | — | — | none | — | — |
| EPIC-08 | API contract documentation | EPIC-02 | Calculate API contract | epic_to_epic | documentation describes the contract EPIC-02 implements | draft from the agreed contract design in parallel; publish after EPIC-02 lands |
| DEP-01 | Upstream consumer systems | — | — | external | no consumer integration means the service delivers no business value regardless of correctness (OQ-87c0d49a) | integration sequencing left to delivery planning; any one upstream system qualifies |

### circular_resolutions
- none

---

---

## sequencing

> **This section is the authoritative build order.** Downstream planners MUST adopt
> `topological_order` as given (restricted to the units in their scope, relative
> positions preserved) rather than re-deriving a sequence from the epic list or from
> id numbers. Re-derivation is how ordering contradictions get introduced.

### topological_order
1. EPIC-01 — Standard arithmetic calculation core (feature, no blockers)
2. EPIC-07 — Stateless deployable service shell (enabler, no blockers)
3. EPIC-02 — Calculate API contract (blocked by EPIC-01)
4. EPIC-03 — Extensible operation registry (blocked by EPIC-01)
5. EPIC-08 — API contract documentation (blocked by EPIC-02)
6. EPIC-04 — Advanced operations extension (blocked by EPIC-03)
7. EPIC-05 — Optional history persistence (blocked by EPIC-01)
8. EPIC-06 — History retrieval (blocked by EPIC-05)

### phase_mapping
- phase_1_mvp (must_have, no/resolved blockers): EPIC-01 (Standard arithmetic calculation core), EPIC-07 (Stateless deployable service shell), EPIC-02 (Calculate API contract), EPIC-03 (Extensible operation registry), EPIC-08 (API contract documentation)
- phase_2 (should_have): EPIC-04 (Advanced operations extension)
- phase_3 (could_have): EPIC-05 (Optional history persistence), EPIC-06 (History retrieval)
- deferred: batch calculations, expression evaluation, workflow integration with automation agents, event-based history (PRD out_of_scope / wont_have items)

### sequencing_validations
- edges_checked: 6 of 6 rows in `## dependencies` (epic_to_epic edges; the external DEP-01 row is carried, not ordered)
- blocked_before_blocker_violations: 0
- prd_conflict_notes: none — phase_mapping follows the PRD mvp_phasing (ASM-07 (MoSCoW priorities inferred from source phrasing), OQ-b450c1ca); dependency-aware sequencing is consistent with it

#### ordering_proof

| blocked | blocked_description | blocker | blocker_description | blocker_position | blocked_position | ok |
|---------|----------------------|--------|----------------------|-------------------|-------------------|-----|
| EPIC-02 | Calculate API contract | EPIC-01 | Standard arithmetic calculation core | 1 | 3 | yes |
| EPIC-03 | Extensible operation registry | EPIC-01 | Standard arithmetic calculation core | 1 | 4 | yes |
| EPIC-04 | Advanced operations extension | EPIC-03 | Extensible operation registry | 4 | 6 | yes |
| EPIC-05 | Optional history persistence | EPIC-01 | Standard arithmetic calculation core | 1 | 7 | yes |
| EPIC-06 | History retrieval | EPIC-05 | Optional history persistence | 7 | 8 | yes |
| EPIC-08 | API contract documentation | EPIC-02 | Calculate API contract | 3 | 5 | yes |

---

---

## reclassification_log

| candidate | gate_failed | reclassified_as | redistributed_to | reason |
|-----------|-------------|-----------------|-------------------|--------|
| NFR-01 (Standard operations respond at millisecond-level latency) | gate_2_nfr | dod_constraint | EPIC-01, EPIC-02 | NFR overlap: latency is a quality constraint on the calculation core and its API, not a demoable deliverable |
| NFR-02 (Same input always produces the same result (determinism)) | gate_2_nfr | dod_constraint | EPIC-01, EPIC-02, EPIC-03, EPIC-04 | NFR overlap: determinism is a DoD constraint on every computation-bearing epic |
| NFR-03 (Calculation succeeds even when history persistence fails) | gate_2_nfr | dod_constraint | EPIC-05 | NFR overlap: availability guarantee embedded in the history persistence epic (ASM-05 (OQ-abad2154)) |
| NFR-04 (Stateless design supports horizontal scaling) | gate_2_nfr | dod_constraint | EPIC-07 | NFR overlap: scalability is the DoD of the stateless service shell enabler |
| NFR-05 (Observability readiness for future metrics and tracing integration) | gate_2_nfr | dod_constraint | EPIC-07 | NFR overlap: observability readiness is a DoD constraint on the service shell |
| NFR-06 (Security and access control for the calculation API) | gate_2_nfr | dod_constraint | EPIC-02 | NFR overlap: security applies to the calculate API surface |
| NFR-07 (API usability for internal consumers) | gate_2_nfr | dod_constraint | EPIC-02, EPIC-08 | NFR overlap: usability applies to the API contract and its documentation |
| API contract documentation (from KPI-04) | gate_3_enabler | enabler EPIC-08 | — | documentation/tooling, traces to KPI-04 (API contract documentation for internal reuse), no FR trace |
| Stateless service shell (from FR-10) | gate_3_enabler | enabler EPIC-07 | — | infrastructure/platform foundation; carries FR-10 trace and KPI-01 alignment, no independent business demo |
| Precision policy definition (from FR-03 design_note) | gate_1_demo | dod_constraint | EPIC-01 | not independently demoable; embedded as the single high-precision computation strategy DoD of EPIC-01; concrete policy pending (PI-01) |

---

---

## persona_coverage

| persona | persona_name | epic_count | epic_ids | status |
|---------|--------------|-----------|----------|--------|
| P-01 | Backend Service Developer | 6 | EPIC-01, EPIC-02, EPIC-03, EPIC-05, EPIC-06, EPIC-08 | covered |
| P-02 | Automation Workflow Engineer | 3 | EPIC-02, EPIC-07, EPIC-08 | covered |
| P-03 | AI Agent / AI-Driven Process Operator | 3 | EPIC-02, EPIC-07, EPIC-08 | covered |
| P-04 | Reporting Pipeline Engineer | 3 | EPIC-01, EPIC-02, EPIC-04, EPIC-08 | covered |

---

---

## goal_coverage

Goals are read from the PRD's JTBD outcomes and KPI targets (the PRD defines no separate goals section).

| prd_goal | epic_ids | status |
|----------|----------|--------|
| JTBD-01 (core_functional): deterministic result from a single authoritative math engine | EPIC-01, EPIC-02 | covered |
| JTBD-02 (related): evolve numerical behavior in one controlled place without breaking consumers | EPIC-03 | covered |
| JTBD-03 (related): trace and debug calculations through recorded metadata | EPIC-05, EPIC-06 | covered |
| JTBD-04 (related): lightweight deterministic calculations inside automated workflows and AI processes | EPIC-02, EPIC-07 | covered |
| KPI-01 (Upstream system integration adoption): at least 1 upstream system integrated | EPIC-02, EPIC-07 | covered |
| KPI-02 (Deterministic test coverage of all operations): 100% deterministic unit test coverage | EPIC-01, EPIC-03, EPIC-04 | covered |
| KPI-03 (History persistence validation): optional history persistence validated | EPIC-05 | covered |
| KPI-04 (API contract documentation for internal reuse): API contract documented | EPIC-08 | covered |
| KPI-05 (Reduction of duplicated math logic across teams): teams stop duplicating math logic | EPIC-01, EPIC-02, EPIC-08 | covered |
| KPI-06 (Result consistency across systems): results become consistent | EPIC-01, EPIC-02 | covered |
| KPI-07 (Calculation traceability availability): calculations traceable when something looks wrong | EPIC-05, EPIC-06 | covered |

---

---

## open_questions

### pending_inputs

| id | affected_ids | blocking | type | section | item_id | field | impact | question | fallback_behavior | source_ref |
|----|--------------|----------|------|---------|---------|-------|--------|----------|-------------------|------------|
| — | — | — | — | — | — | — | — | — | — | — |

No pending inputs raised by this run. Upstream PRD pending inputs (PI-01 through PI-11) are inherited and tracked in the PRD ledger; this backlog cites them where an epic relies on one (PI-01 in EPIC-01, OQ-025246e3 in EPIC-06, OQ-8a3647b2 and OQ-4d202bf8 in EPIC-02, OQ-87c0d49a in DEP-01, OQ-c5ed68d9 in KPI ownership rows).

### alignment_gaps

| id | affected_ids | blocking | type | item_id | impact | description | fallback_behavior | source_ref |
|----|--------------|----------|------|---------|--------|-------------|-------------------|------------|
| — | — | — | — | — | — | — | — | — |

No alignment gaps: every PRD persona is served by at least one epic, every JTBD and KPI traces to at least one epic, and no epic is unaligned (EPIC-07 and EPIC-08 are KPI-backed enablers per the reclassification log).

### assumptions_to_validate

| id | affected_ids | blocking | assumption | source | impact_if_wrong | validation_method | fallback_behavior |
|----|--------------|----------|------------|--------|-----------------|-------------------|-------------------|
| ASM-E01 | T-04, EPIC-07, EPIC-08 | no | T-04 (platform service foundation) groups the stateless service shell (EPIC-07) and the API contract documentation (EPIC-08) as one foundation theme | theme generation (this run) | Epics may be misgrouped; EPIC-08 is a documentation enabler while EPIC-07 is an infrastructure enabler, so a reviewer may prefer separate themes | Review theme grouping with the product owner during backlog review | Current grouping used; both epics keep their own traceability and sequencing regardless of theme membership |

### summary
- total_themes: 4
- total_epics: 8
- total_enablers: 2
- total_spikes: 0
- total_reclassified: 10
- unaligned_epics: 0
- uncovered_goals: 0
- uncovered_personas: 0
- pending_inputs: 0
- assumptions: 1
- status: complete

---
