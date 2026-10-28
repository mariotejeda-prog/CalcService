# Math Operations Microservice (Calc-Service) — PRD Spec
version: 1.0.1
session: pd-20261027-calc001
mode: SYNTHESIZE
date: 2026-10-28
language: en
status: draft
contract_format: structured
content_hash: 25c7814b3af060433c725d5e0d92674538e4af564960493da328ea935e63f93a
---

## project_info
client_name: Internal platform organization (client organization not named in source inputs; service built for internal platform teams)
project_name: Math Operations Microservice (Calc-Service)
project_description: A lightweight, stateless microservice that provides a centralized, deterministic mathematical computation capability for the platform, externalizing calculation logic from client applications to enable consistency, reuse, and controlled evolution of numerical behavior across the ecosystem.

---

## scope

### in_scope
- Standard arithmetic operations: addition, subtraction, multiplication, division | source: project-brief.md §2 Core Capabilities; discovery-transcript.md §01:30 | status: complete
- Advanced operations: exponentiation, roots, trigonometric functions | source: project-brief.md §2 Core Capabilities; discovery-transcript.md §01:30 | status: complete
- High-precision internal computation producing deterministic outputs suitable for financial and scientific contexts | source: project-brief.md §2 Precision Strategy; discovery-transcript.md §03:00 | status: complete
- Optional history tracking of calculation metadata for traceability and diagnostics, as lightweight local persistence that must not impact response delivery | source: project-brief.md §2 History Tracking; discovery-transcript.md §04:30, §11:00 | status: complete
- Calculate operation API: `POST /api/v1/math/calculate` accepting operands and operation, returning result, operation type, and timestamp | source: project-brief.md §4 API Contract; discovery-transcript.md §07:30 | status: complete
- History retrieval capability providing recent calculation records for diagnostics and auditing | source: project-brief.md §4 History Retrieval | status: complete
- Clear, fail-fast error handling for invalid input (division by zero, unsupported operation) | source: discovery-transcript.md §10:00 | status: complete
- Extensibility: new operations added without breaking existing consumers | source: discovery-transcript.md §01:30 | status: complete
- Stateless microservice deployment model: independent deployment, horizontally scalable computation layer, no user session ownership | source: project-brief.md §3 Architectural Positioning | status: complete

### out_of_scope
- Batch calculations | source: discovery-transcript.md §07:30 ("Maybe later, not day one") | status: complete
- Expression evaluation (e.g., `"5 + (3 * 2)"`) | source: project-brief.md §2 Future Capability (Planned); discovery-transcript.md §07:30 ("not required now") | status: complete
- Workflow integration with automation agents | source: project-brief.md §2 Future Capability (Planned) | status: complete
- Event-based history instead of database persistence | source: project-brief.md §2 Future Capability (Planned) | status: complete
- Multi-instance history consistency | source: discovery-transcript.md §11:00 ("Not required now") | status: complete
- Regulatory-grade audit persistence of every calculation | source: discovery-transcript.md §04:30, §11:00 ("Not necessarily every one... more diagnostic than regulatory") | status: complete

### research_methodology
- Derived from project brief (v1.1) analysis and one 15-minute discovery transcript with the product owner; no primary user interviews, surveys, or meeting recordings provided; meetings and documentation input folders were empty | source: project-brief.md; discovery-transcript.md | status: complete

---

## personas

### P-01: Backend Service Developer
- type: primary
- description: Developer building backend services that need consistent mathematical logic and currently face duplicated calculation implementations across systems | source: project-brief.md §3 Primary Consumers; discovery-transcript.md §00:00, §06:00
- goals: consume a single, consistent math engine instead of embedding calculation logic in each service; get deterministic results for the same input | source: discovery-transcript.md §00:00, §03:00
- pain_points: calculations spread across frontend, backend, and reports; formulas changing break things in different places; rounding differences between systems | source: discovery-transcript.md §00:00, §03:00
- technical_proficiency: high (builds and integrates backend services) | status: assumption [ASM-01]
- usage_frequency: not stated in inputs
- demographics: backend service developer; high technical proficiency; works in a multi-team platform organization with frontend, backend, and reporting teams | source: discovery-transcript.md §00:00, §06:00 | status: assumption [ASM-01]
- prioritization_rationale: backend services are the first and primary consumers named by the product owner ("Backend services first") | source: discovery-transcript.md §06:00 | status: complete
- status: complete
- source: project-brief.md §3; discovery-transcript.md §00:00, §03:00, §06:00

### P-02: Automation Workflow Engineer
- type: secondary
- description: Engineer building automation workflows that call the service synchronously as part of pipelines, including validating outputs inside pipelines | source: project-brief.md §3 Primary Consumers; discovery-transcript.md §06:00, §12:00
- goals: reliable, lightweight synchronous calculations inside automated workflows; potential throughput spikes when automation kicks in | source: discovery-transcript.md §06:00, §09:00, §12:00
- pain_points: not stated in inputs
- technical_proficiency: high (builds automation workflows) | status: assumption [ASM-01]
- usage_frequency: not stated in inputs
- demographics: automation/workflow engineer; high technical proficiency; operates within platform automation teams | source: discovery-transcript.md §06:00, §12:00 | status: assumption [ASM-01]
- status: complete
- source: project-brief.md §3; discovery-transcript.md §06:00, §09:00, §12:00

### P-03: AI Agent / AI-Driven Process Operator
- type: secondary
- description: Operator or maintainer of AI agents performing deterministic calculations as part of AI-driven processes | source: project-brief.md §3 Primary Consumers ("AI agents performing deterministic calculations"); discovery-transcript.md §06:00 ("Possibly AI agents")
- goals: deterministic calculations from an AI-driven process with predictable, repeatable results | source: project-brief.md §3; discovery-transcript.md §03:00
- pain_points: not stated in inputs
- technical_proficiency: not stated in inputs
- usage_frequency: not stated in inputs
- demographics: AI agent operator; technical proficiency not stated in inputs; organizational context not stated in inputs | status: assumption [ASM-01]
- status: complete
- source: project-brief.md §3; discovery-transcript.md §06:00

### P-04: Reporting Pipeline Engineer
- type: secondary
- description: Engineer building reporting pipelines that consume the calculation service for consistent numerical behavior in reports | source: project-brief.md §3 Primary Consumers; discovery-transcript.md §00:00 (calculations spread across "even reports")
- goals: consistent numerical results across reporting outputs | source: discovery-transcript.md §00:00, §03:00
- pain_points: rounding differences between systems affecting reports | source: discovery-transcript.md §03:00
- technical_proficiency: high (builds reporting pipelines) | status: assumption [ASM-01]
- usage_frequency: not stated in inputs
- demographics: reporting pipeline engineer; high technical proficiency; part of platform reporting function | source: discovery-transcript.md §00:00 | status: assumption [ASM-01]
- status: complete
- source: project-brief.md §3; discovery-transcript.md §00:00, §03:00

---

## functional_requirements

### FR-01: Execute standard arithmetic operations (addition, subtraction, multiplication, division)
- priority: must_have
- source: project-brief.md §2 Standard Operations; discovery-transcript.md §01:30 ("Basic arithmetic first — add, subtract, multiply, divide. That covers most cases.")
- status: complete
- design_note: not stated in inputs
- depends_on: [none identified]
- acceptance_criteria:
  - FR-01-AC-01: given: the service is running | when: a calculate request is submitted with two numeric operands and operation `ADD` | then: the response returns the sum of the operands as `resultValue`, with `operationType` set to `ADD` and a `timestamp` present
  - FR-01-AC-02: given: the service is running | when: a calculate request is submitted with operation `SUBTRACT` | then: the response returns the difference of operand1 minus operand2 as `resultValue`
  - FR-01-AC-03: given: the service is running | when: a calculate request is submitted with operation `MULTIPLY` | then: the response returns the product of the operands as `resultValue`
  - FR-01-AC-04: given: the service is running | when: a calculate request is submitted with operation `DIVIDE` and a non-zero divisor | then: the response returns the quotient of the operands as `resultValue`

### FR-02: Execute advanced operations (exponentiation, roots, trigonometric functions)
- priority: should_have
- source: project-brief.md §2 Advanced Operations; discovery-transcript.md §01:30 ("powers, square roots, some trigonometry")
- status: complete
- design_note: not stated in inputs
- depends_on: [FR-01]
- acceptance_criteria:
  - FR-02-AC-01: given: the service is running | when: a calculate request is submitted with an exponentiation operation | then: the response returns the exponentiation result as `resultValue`
  - FR-02-AC-02: given: the service is running | when: a calculate request is submitted with a root operation | then: the response returns the root result as `resultValue`
  - FR-02-AC-03: given: the service is running | when: a calculate request is submitted with a trigonometric operation | then: the response returns the trigonometric result as `resultValue`
  - FR-02-AC-04: given: the service is running | when: a calculate request is submitted with an advanced operation not yet supported by the service | then: the service returns a clear error identifying the unsupported operation

### FR-03: Produce deterministic, high-precision results
- priority: must_have
- source: project-brief.md §2 Precision Strategy, §5 Determinism; discovery-transcript.md §03:00 ("deterministic outputs for the same input", "We don't want floating-point surprises")
- status: complete
- design_note: high-precision internal computation is the stated mechanism to reduce rounding inconsistencies; the concrete precision policy (decimal digits, rounding mode) is not stated in inputs | status: pending
- depends_on: [FR-01]
- acceptance_criteria:
  - FR-03-AC-01: given: any supported operation | when: the same request is submitted repeatedly | then: the response returns the identical `resultValue` every time
  - FR-03-AC-02: given: two different client systems submit the same calculation | when: both responses are compared | then: both return the identical `resultValue` with no rounding differences between systems

### FR-04: Expose a calculate operation via `POST /api/v1/math/calculate`
- priority: must_have
- source: project-brief.md §4 API Contract; discovery-transcript.md §07:30 ("Simple. Send operands and operation, get result. No complex payloads.")
- status: complete
- design_note: request/response field names `operand1`, `operand2`, `operation`, `resultValue`, `operationType`, `timestamp` are taken verbatim from the brief's conceptual contract | status: assumption [ASM-02]
- depends_on: [FR-01]
- acceptance_criteria:
  - FR-04-AC-01: given: the service is running | when: a request is sent to `POST /api/v1/math/calculate` with body `{"operand1": 10.5, "operand2": 5.0, "operation": "ADD"}` | then: the response contains `resultValue: 15.5`, `operationType: "ADD"`, and a `timestamp` value
  - FR-04-AC-02: given: the service is running | when: a calculate request is submitted | then: the response contains only the simple result payload (result, operation type, timestamp) with no complex payload structure

### FR-05: Return clear, fail-fast errors for invalid input
- priority: must_have
- source: discovery-transcript.md §10:00 ("Clear errors. Division by zero, unsupported operation — that kind of thing." / "Strict validation? Yes. Fail fast.")
- status: complete
- design_note: not stated in inputs
- depends_on: [FR-01]
- acceptance_criteria:
  - FR-05-AC-01: given: the service is running | when: a calculate request is submitted with operation `DIVIDE` and operand2 equal to zero | then: the service returns a clear error identifying division by zero and does not return a numeric result
  - FR-05-AC-02: given: the service is running | when: a calculate request is submitted with an operation value the service does not support | then: the service returns a clear error identifying the unsupported operation
  - FR-05-AC-03: given: the service is running | when: a calculate request is submitted with missing or non-numeric operands | then: the service rejects the request with a clear validation error before any computation

### FR-06: Record calculation history as optional, auxiliary local persistence
- priority: could_have
- source: project-brief.md §2 History Tracking; discovery-transcript.md §04:30 ("we want the ability to log calculations when needed — mainly debugging and traceability"), §11:00 ("persistence is auxiliary")
- status: complete
- design_note: implemented as lightweight local persistence via an embedded database abstraction; persistence can be replaced without changing the API contract | status: assumption [ASM-03]
- depends_on: [FR-01]
- acceptance_criteria:
  - FR-06-AC-01: given: history tracking is enabled | when: a calculation is executed | then: calculation metadata is recorded for traceability and diagnostics
  - FR-06-AC-02: given: history tracking is enabled | when: a calculation is executed | then: the history write does not delay or block delivery of the calculation response
  - FR-06-AC-03: given: history tracking is disabled or not configured | when: a calculation is executed | then: the calculation still executes and returns a result normally

### FR-07: Calculation must succeed even if history persistence fails
- priority: must_have
- source: project-brief.md §5 Reliability; discovery-transcript.md §04:30 ("If persistence fails, should the calculation still return?" / "Yes, always."), §15:00 ("Persistence must never block calculation")
- status: complete
- design_note: not stated in inputs
- depends_on: [FR-06]
- acceptance_criteria:
  - FR-07-AC-01: given: history persistence is failing or unavailable | when: a valid calculate request is submitted | then: the calculation still succeeds and returns the correct result to the caller

### FR-08: Provide history retrieval of recent calculation records
- priority: could_have
- source: project-brief.md §4 History Retrieval ("Provides recent calculation records for diagnostics and auditing")
- status: complete
- design_note: the retrieval endpoint path and response shape are not stated in inputs | status: pending
- depends_on: [FR-06]
- acceptance_criteria:
  - FR-08-AC-01: given: history tracking is enabled and calculations have been executed | when: a history retrieval request is submitted | then: the service returns recent calculation records for diagnostics and auditing

### FR-09: Support extensibility so new operations are added without breaking existing consumers
- priority: must_have
- source: discovery-transcript.md §01:30 ("Definitely. New operations will be added without breaking existing consumers.")
- status: complete
- design_note: not stated in inputs
- depends_on: [FR-01]
- acceptance_criteria:
  - FR-09-AC-01: given: existing consumers are integrated against the calculate operation | when: a new operation is added to the service | then: existing consumers continue to receive correct results for previously supported operations with no contract change

### FR-10: Operate as a stateless, independently deployable, horizontally scalable service
- priority: must_have
- source: project-brief.md §3 Architectural Positioning ("Independent deployment", "Horizontally scalable computation layer", "No user session ownership")
- status: complete
- design_note: stateless design is the stated mechanism enabling horizontal scaling | status: assumption [ASM-04]
- depends_on: [none identified]
- acceptance_criteria:
  - FR-10-AC-01: given: multiple service instances are deployed | when: calculate requests are distributed across instances | then: each request is served independently with no user session ownership and identical results for identical inputs
  - FR-10-AC-02: given: the service is deployed | when: the service is deployed or scaled independently of other platform components | then: calculation behavior is unchanged

---

## non_functional_requirements

### NFR-01: Standard operations respond at millisecond-level latency
- category: performance
- target: response time at millisecond level for standard operations [PENDING: exact numeric threshold] 
- source: project-brief.md §5 Latency ("Millisecond-level response time for standard operations"); discovery-transcript.md §09:00 ("It should feel instant. This isn't heavy computation.")
- status: pending
- design_note: not stated in inputs

### NFR-02: Same input always produces the same result (determinism)
- category: compliance
- target: 100% of repeated identical requests return the identical resultValue
- source: project-brief.md §5 Determinism; discovery-transcript.md §03:00
- status: complete
- design_note: not stated in inputs

### NFR-03: Calculation succeeds even when history persistence fails
- category: availability
- target: 100% of valid calculate requests return a result when history persistence is unavailable
- source: project-brief.md §5 Reliability; discovery-transcript.md §04:30, §15:00
- status: complete
- design_note: behavioral guarantee mirrored as FR-07; retained here as the availability-level statement of the same guarantee | status: assumption [ASM-05]

### NFR-04: Stateless design supports horizontal scaling
- category: scalability
- target: service capacity scales horizontally by adding instances with no change to calculation behavior [PENDING: numeric concurrency/volume target]
- source: project-brief.md §5 Scalability, §3 Architectural Positioning; discovery-transcript.md §09:00 ("Possibly if automation kicks in, but calculations are lightweight")
- status: pending
- design_note: not stated in inputs

### NFR-05: Observability readiness for future metrics and tracing integration
- category: usability
- target: [PENDING: nfr_target — no measurable observability target stated in inputs]
- source: project-brief.md §5 Observability readiness ("Designed for future metrics and tracing integration")
- status: pending
- design_note: not stated in inputs

### NFR-06: Security and access control for the calculation API
- category: security
- target: [PENDING: nfr_target — no security requirement stated in inputs]
- source: not stated in inputs
- status: pending
- design_note: not stated in inputs

### NFR-07: API usability for internal consumers
- category: usability
- target: [PENDING: nfr_target — no usability/accessibility standard stated in inputs]
- source: discovery-transcript.md §07:30 ("Simple. Send operands and operation, get result."), §13:30 ("it must be simple or teams won't use it")
- status: pending
- design_note: not stated in inputs

---

## kpis

### KPI-01: Upstream system integration adoption
- target: at least 1 upstream system successfully integrated
- measurement: count of upstream systems integrated with the calculate operation
- timeframe: not stated in inputs
- source: project-brief.md §7 Success Criteria ("Service successfully integrated by at least one upstream system")
- status: complete

### KPI-02: Deterministic test coverage of all operations
- target: 100% of supported operations covered by deterministic unit tests
- measurement: deterministic unit test coverage report across all operations
- timeframe: not stated in inputs
- source: project-brief.md §7 Success Criteria ("Deterministic unit test coverage for all operations")
- status: complete

### KPI-03: History persistence validation
- target: optional history persistence validated
- measurement: validation run confirming history persistence works without impacting response delivery
- timeframe: not stated in inputs
- source: project-brief.md §7 Success Criteria ("Optional history persistence validated")
- status: complete

### KPI-04: API contract documentation for internal reuse
- target: API contract documented for internal reuse
- measurement: published API contract documentation available to internal consumers
- timeframe: not stated in inputs
- source: project-brief.md §7 Success Criteria ("API contract documented for internal reuse")
- status: complete

### KPI-05: Reduction of duplicated math logic across teams
- target: [PENDING: numeric target — success criterion stated qualitatively ("Teams stop duplicating math logic")]
- measurement: not stated in inputs
- timeframe: not stated in inputs
- source: discovery-transcript.md §14:30 ("Teams stop duplicating math logic")
- status: pending

### KPI-06: Result consistency across systems
- target: [PENDING: numeric target — success criterion stated qualitatively ("Results become consistent")]
- measurement: not stated in inputs
- timeframe: not stated in inputs
- source: discovery-transcript.md §14:30 ("Results become consistent")
- status: pending

### KPI-07: Calculation traceability availability
- target: [PENDING: numeric target — success criterion stated qualitatively ("We can trace calculations when something looks wrong")]
- measurement: not stated in inputs
- timeframe: not stated in inputs
- source: discovery-transcript.md §14:30 ("We can trace calculations when something looks wrong")
- status: pending

---

## jtbd

### JTBD-01: core_functional
- situation: when a backend service, report, or workflow needs a calculation performed, the calculation logic is currently duplicated across frontend, backend, and reporting systems | source: discovery-transcript.md §00:00
- motivation: obtain a consistent, correct numerical result from a single authoritative math engine instead of maintaining formula logic in each consumer | source: discovery-transcript.md §00:00; project-brief.md §1 Business Objective
- outcome: the caller receives a deterministic result for the operation requested, identical to what any other system would receive for the same input | source: discovery-transcript.md §03:00; project-brief.md §5 Determinism
- source: discovery-transcript.md §00:00, §03:00; project-brief.md §1, §5
- status: complete

### JTBD-02: related
- situation: when a formula or numerical behavior changes, each system that embedded its own calculation logic must be found and fixed individually, and changes break things in different places | source: discovery-transcript.md §00:00
- motivation: evolve numerical behavior in one controlled place so changes propagate consistently across the ecosystem | source: project-brief.md §1 ("controlled evolution of numerical behavior"); discovery-transcript.md §00:00
- outcome: calculation logic is updated once in the central service and all consumers immediately use the same updated behavior without breaking existing consumers | source: discovery-transcript.md §01:30; project-brief.md §1
- source: discovery-transcript.md §00:00, §01:30; project-brief.md §1
- status: complete

### JTBD-03: related
- situation: when a calculation result looks wrong, an engineer needs to understand what the service computed and why | source: discovery-transcript.md §04:30, §14:30
- motivation: trace and debug calculations through recorded calculation metadata without coupling persistence to computation | source: discovery-transcript.md §04:30 ("mainly debugging and traceability"); project-brief.md §2 History Tracking
- outcome: recent calculation records are available for diagnostics and auditing, and retrieving them never degrades the calculation itself | source: project-brief.md §2 History Tracking, §4 History Retrieval; discovery-transcript.md §04:30, §11:00
- source: discovery-transcript.md §04:30, §11:00, §14:30; project-brief.md §2, §4
- status: complete

### JTBD-04: related
- situation: when an automated workflow or AI-driven process runs, it needs lightweight deterministic calculations performed synchronously as part of the pipeline | source: discovery-transcript.md §06:00, §09:00, §12:00
- motivation: validate outputs and perform computations inside pipelines using a stable, reusable computation service rather than bespoke logic | source: discovery-transcript.md §12:00 ("Potentially as part of automated workflows — like validating outputs inside pipelines"); project-brief.md §1
- outcome: automation workflows and AI agents obtain instant, deterministic calculation results as a foundational building block for AI-assisted workflows | source: discovery-transcript.md §06:00, §09:00; project-brief.md §6
- source: discovery-transcript.md §06:00, §09:00, §12:00; project-brief.md §1, §6
- status: complete

---

## traceability

| FR | fr_name | JTBD | jtbd_name | direction | status |
|----|---------|------|-----------|-----------|--------|
| FR-01 | Execute standard arithmetic operations (addition, subtraction, multiplication, division) | JTBD-01 | core_functional | forward | mapped |
| FR-02 | Execute advanced operations (exponentiation, roots, trigonometric functions) | JTBD-01 | core_functional | forward | mapped |
| FR-03 | Produce deterministic, high-precision results | JTBD-01 | core_functional | forward | mapped |
| FR-04 | Expose a calculate operation via `POST /api/v1/math/calculate` | JTBD-01 | core_functional | forward | mapped |
| FR-05 | Return clear, fail-fast errors for invalid input | JTBD-01 | core_functional | forward | mapped |
| FR-06 | Record calculation history as optional, auxiliary local persistence | JTBD-03 | related | forward | mapped |
| FR-07 | Calculation must succeed even if history persistence fails | JTBD-03 | related | forward | mapped |
| FR-08 | Provide history retrieval of recent calculation records | JTBD-03 | related | forward | mapped |
| FR-09 | Support extensibility so new operations are added without breaking existing consumers | JTBD-02 | related | forward | mapped |
| FR-10 | Operate as a stateless, independently deployable, horizontally scalable service | JTBD-04 | related | forward | mapped |
| — | — | JTBD-01 | core_functional | reverse | mapped |
| — | — | JTBD-02 | related | reverse | mapped |
| — | — | JTBD-03 | related | reverse | mapped |
| — | — | JTBD-04 | related | reverse | mapped |

---

## dependencies

### DEP-01: Upstream consumer systems
- type: external
- description: at least one upstream system (backend service, reporting pipeline, automation workflow, or AI agent) must integrate with the calculate operation for the service to deliver value | source: project-brief.md §7 Success Criteria
- owner: not stated in inputs
- impact_if_unavailable: no consumer integration means the service delivers no business value regardless of correctness | source: project-brief.md §7
- source: project-brief.md §7 Success Criteria
- status: pending

### DEP-02: Embedded persistence abstraction for optional history
- type: technical
- description: optional history tracking relies on a lightweight local persistence behind an embedded database abstraction, replaceable without changing the API contract | source: project-brief.md §2 History Tracking, §3 Architectural Positioning
- owner: not stated in inputs
- impact_if_unavailable: history tracking and history retrieval (FR-06, FR-08) cannot be delivered; calculations themselves are unaffected because persistence must never block calculation (FR-07) | source: project-brief.md §2, §5
- source: project-brief.md §2, §3, §5
- status: complete

### inter_feature_dependencies
- FR-02 (Execute advanced operations) depends_on FR-01 (Execute standard arithmetic operations): advanced operations extend the same calculate operation contract established by standard operations | source: project-brief.md §2; discovery-transcript.md §01:30 | status: complete
- FR-03 (Produce deterministic, high-precision results) depends_on FR-01 (Execute standard arithmetic operations): determinism applies to the operations FR-01 defines | source: project-brief.md §2, §5 | status: complete
- FR-04 (Expose a calculate operation via `POST /api/v1/math/calculate`) depends_on FR-01 (Execute standard arithmetic operations): the endpoint exposes the operations FR-01 defines | source: project-brief.md §4 | status: complete
- FR-05 (Return clear, fail-fast errors for invalid input) depends_on FR-01 (Execute standard arithmetic operations): error handling governs requests against the calculate operation | source: discovery-transcript.md §10:00 | status: complete
- FR-06 (Record calculation history as optional, auxiliary local persistence) depends_on FR-01 (Execute standard arithmetic operations): history records metadata of calculations performed via the calculate operation | source: project-brief.md §2 | status: complete
- FR-07 (Calculation must succeed even if history persistence fails) depends_on FR-06 (Record calculation history as optional, auxiliary local persistence): the persistence-failure guarantee is meaningful only when history recording exists | source: project-brief.md §5 Reliability | status: complete
- FR-08 (Provide history retrieval of recent calculation records) depends_on FR-06 (Record calculation history as optional, auxiliary local persistence): retrieval requires recorded history | source: project-brief.md §4 | status: complete
- FR-09 (Support extensibility so new operations are added without breaking existing consumers) depends_on FR-01 (Execute standard arithmetic operations): extensibility preserves the contract FR-01 establishes | source: discovery-transcript.md §01:30 | status: complete

---

## risks

### RSK-01: Hidden logic differences remain between systems after the service launches
- likelihood: medium
- impact: high
- mitigation: centralize all calculation logic in the service and enforce deterministic, high-precision computation (FR-03) so all systems receive identical results; validate with deterministic unit test coverage for all operations (KPI-02)
- source: discovery-transcript.md §13:30 ("Hidden logic differences between systems")
- status: complete

### RSK-02: Teams bypass the service and keep embedding their own calculation logic
- likelihood: medium
- impact: high
- mitigation: keep the API simple — send operands and operation, get result — because adoption depends on simplicity ("it must be simple or teams won't use it"); document the API contract for internal reuse (KPI-04)
- source: discovery-transcript.md §13:30 ("people bypassing the service", "it must be simple or teams won't use it")
- status: complete

### RSK-03: History persistence degrades calculation response delivery
- likelihood: low
- impact: medium
- mitigation: treat persistence as auxiliary storage that must not impact response delivery (FR-06, FR-07); calculation must always succeed even if persistence fails
- source: project-brief.md §2 History Tracking, §5 Reliability; discovery-transcript.md §04:30, §15:00
- status: complete

### RSK-04: Precision policy gaps reintroduce floating-point surprises between systems
- likelihood: medium
- impact: medium
- mitigation: define and apply a single high-precision internal computation strategy (FR-03); the concrete precision policy (decimal digits, rounding mode) still needs to be fixed before implementation (PI-01)
- source: discovery-transcript.md §03:00 ("Some teams complained about rounding differences between systems", "We don't want floating-point surprises")
- status: complete

---

## assumptions

### ASM-01: Consumer personas are technically proficient engineering roles
- impact_if_wrong: persona goals, pain points, and usage patterns may not reflect actual consumer behavior, skewing prioritization of API usability work
- source_basis: inferred from the consumer types named in project-brief.md §3 (backend services, reporting pipelines, automation workflows, AI agents) and discovery-transcript.md §06:00
- status: active

### ASM-02: The brief's conceptual API contract field names are the intended contract names
- impact_if_wrong: FR-04 acceptance criteria and downstream API documentation would use field names that differ from the final implemented contract
- source_basis: project-brief.md §4 API Contract (input fields `operand1`, `operand2`, `operation`; output fields `resultValue`, `operationType`, `timestamp`) — labeled "Conceptual" in the brief
- status: active

### ASM-03: History persistence uses a lightweight local embedded database abstraction
- impact_if_wrong: the persistence design, deployment footprint, and replaceability guarantees in FR-06 would need rework
- source_basis: project-brief.md §2 History Tracking ("lightweight local persistence"), §3 ("Optional local persistence via embedded database abstraction"), and the brief's alignment note ("H2-based lightweight persistence strategy")
- status: active

### ASM-04: Stateless design is the mechanism that enables horizontal scaling
- impact_if_wrong: scaling behavior would depend on stateful components, invalidating FR-10 acceptance criteria
- source_basis: project-brief.md §3 ("stateless microservice model", "Horizontally scalable computation layer"), §5 ("Stateless design allows horizontal scaling")
- status: active

### ASM-05: The reliability guarantee is stated at both FR and NFR level intentionally
- impact_if_wrong: duplicate coverage of the same guarantee could cause inconsistent verification; if unintended, NFR-03 should be merged into FR-07
- source_basis: project-brief.md §5 Reliability states the guarantee as a non-functional goal; FR-07 carries the behavioral statement
- status: active

### ASM-06: Synchronous request/response is the only interaction mode required at launch
- impact_if_wrong: asynchronous or batch interaction modes would require additional API surface not covered by FR-04
- source_basis: discovery-transcript.md §06:00 ("Real-time usage?" / "Mostly synchronous, yes."); batch calculations explicitly deferred ("Maybe later, not day one")
- status: active

### ASM-07: MoSCoW priorities inferred from source phrasing
- impact_if_wrong: phasing in mvp_phasing would change; operations or capabilities assumed must-have could be deferred or vice versa
- source_basis: inferred from project-brief.md §2 ("Standard Operations" listed first, "History Tracking (Optional)", "Future Capability (Planned)") and discovery-transcript.md §01:30 ("Basic arithmetic first"), §07:30 ("Maybe later, not day one")
- status: active

---

## constraints

### budget
- [PENDING: budget_constraints] | value: "unknown — stakeholder input required" | status: pending

### timeline
- [PENDING: timeline_constraints] | value: "unknown — stakeholder input required" | status: pending

### resource
- [PENDING: resource_constraints] | value: "unknown — stakeholder input required" | status: pending

### regulatory
- [PENDING: regulatory_constraints] | status: pending

### organizational
- History is diagnostic, not regulatory: history tracking is not required to meet a regulatory audit obligation | source: discovery-transcript.md §11:00 ("History is more diagnostic than regulatory") | status: complete
- Persistence must be replaceable without changing the API contract | source: project-brief.md §3 ("persistence can be replaced without changing the API contract") | status: complete

---

## mvp_phasing

### phase_1_mvp
- must_have: FR-01 (Execute standard arithmetic operations), FR-03 (Produce deterministic, high-precision results), FR-04 (Expose a calculate operation via `POST /api/v1/math/calculate`), FR-05 (Return clear, fail-fast errors for invalid input), FR-09 (Support extensibility so new operations are added without breaking existing consumers), FR-10 (Operate as a stateless, independently deployable, horizontally scalable service) | source: project-brief.md §2 Standard Operations, §4 API Contract, §5; discovery-transcript.md §01:30 ("Basic arithmetic first"), §10:00, §13:30 | status: assumption — priorities inferred from source phrasing, pending product-owner confirmation (see assumptions_to_validate)

### phase_2
- should_have: FR-02 (Execute advanced operations) | source: project-brief.md §2 Advanced Operations; discovery-transcript.md §01:30 ("powers, square roots, some trigonometry" listed after basic arithmetic) | status: assumption — priorities inferred from source phrasing, pending product-owner confirmation (see assumptions_to_validate)

### phase_3
- could_have: FR-06 (Record calculation history as optional, auxiliary local persistence), FR-08 (Provide history retrieval of recent calculation records) | source: project-brief.md §2 History Tracking (Optional), §4 History Retrieval (Optional); discovery-transcript.md §04:30 ("Not necessarily every one") | status: assumption — priorities inferred from source phrasing, pending product-owner confirmation (see assumptions_to_validate)

### deferred
- wont_have: batch calculations, expression evaluation, workflow integration with automation agents, event-based history | source: project-brief.md §2 Future Capability (Planned); discovery-transcript.md §07:30 ("Maybe later, not day one", "not required now") | status: complete

### sequencing_notes
- FR-07 (Calculation must succeed even if history persistence fails) requires FR-06 (Record calculation history as optional, auxiliary local persistence) to exist before its guarantee is testable | source: project-brief.md §5 Reliability | status: complete
- FR-08 (Provide history retrieval of recent calculation records) depends on FR-06 (Record calculation history as optional, auxiliary local persistence) completion | source: project-brief.md §4 | status: complete
- FR-02 (Execute advanced operations) extends FR-01 (Execute standard arithmetic operations) and follows it in delivery order | source: discovery-transcript.md §01:30 | status: complete

---

## literal_registry

| id | name | exact_value | type | applies_to_ids | source_ref |
|----|------|-------------|------|----------------|------------|
| LIT-01 | calculate endpoint route | POST /api/v1/math/calculate | route | FR-04, FR-04-AC-01 | project-brief.md §4 API Contract |
| LIT-02 | request field: operand1 | operand1 | field | FR-04, FR-04-AC-01 | project-brief.md §4 API Contract |
| LIT-03 | request field: operand2 | operand2 | field | FR-04, FR-04-AC-01 | project-brief.md §4 API Contract |
| LIT-04 | request field: operation | operation | field | FR-04, FR-04-AC-01 | project-brief.md §4 API Contract |
| LIT-05 | response field: resultValue | resultValue | field | FR-01, FR-04, FR-04-AC-01 | project-brief.md §4 API Contract |
| LIT-06 | response field: operationType | operationType | field | FR-01, FR-04, FR-04-AC-01 | project-brief.md §4 API Contract |
| LIT-07 | response field: timestamp | timestamp | field | FR-04, FR-04-AC-01 | project-brief.md §4 API Contract |
| LIT-08 | operation enum: ADD | ADD | enum | FR-01, FR-01-AC-01, FR-04-AC-01 | project-brief.md §4 API Contract |
| LIT-09 | example request values | operand1: 10.5, operand2: 5.0, operation: ADD → resultValue: 15.5 | other | FR-04-AC-01 | project-brief.md §4 API Contract |
| LIT-10 | latency goal | millisecond-level response time for standard operations | constraint | NFR-01 | project-brief.md §5 Non-Functional Goals |
| LIT-11 | persistence strategy | H2-based lightweight persistence strategy | constraint | FR-06, ASM-03 | project-brief.md alignment note (v1.1) |

---

## glossary

| term_id | term | definition | first_used | source_ref |
|---------|------|------------|------------|------------|
| TERM-01 | Calc-Service | The math operations microservice: a lightweight, stateless microservice providing centralized mathematical computation for the platform | project_info | project-brief.md Overview |
| TERM-02 | math engine | The deterministic computation core of Calc-Service consumed by backend services, automation workflows, and AI-driven processes | project_info | project-brief.md Overview |
| TERM-03 | standard operations | The four basic arithmetic operations: addition, subtraction, multiplication, division | scope | project-brief.md §2 Standard Operations |
| TERM-04 | advanced operations | Exponentiation, roots, and trigonometric functions beyond the standard operations | scope | project-brief.md §2 Advanced Operations |
| TERM-05 | history tracking | Optional recording of calculation metadata for traceability and diagnostics, implemented as lightweight local auxiliary persistence | scope | project-brief.md §2 History Tracking |
| TERM-06 | history retrieval | Capability providing recent calculation records for diagnostics and auditing | scope | project-brief.md §4 History Retrieval |
| TERM-07 | determinism | Property that the same input always produces the same result | non_functional_requirements | project-brief.md §5 Determinism |
| TERM-08 | embedded database abstraction | The replaceable local persistence layer behind history tracking; persistence can be replaced without changing the API contract | dependencies | project-brief.md §3 Architectural Positioning |
| TERM-09 | calculate operation | The primary API operation exposed at `POST /api/v1/math/calculate` accepting operands and an operation and returning a result | functional_requirements | project-brief.md §4 API Contract |

---

## open_questions

This section is the **single consolidated registry** of all unresolved items.
Every `[PENDING]` marker, every `orphan` traceability entry, every `assumption`
that needs validation — all appear here once.

Downstream skills (Epics, User Stories, Architecture, Implementation) read
ONLY this section to understand what is unresolved. They do NOT scan other
sections for [PENDING] markers — this registry IS the source of truth.

### pending_inputs

| id | affected_ids | blocking | type | section | item_id | field | impact | question | fallback_behavior | source_ref |
|----|--------------|----------|------|---------|---------|-------|--------|----------|-------------------|------------|
| PI-01 | FR-03, FR-03-AC-01, FR-03-AC-02, RSK-04 | no | missing | functional_requirements | FR-03 | design_note (precision policy) | HIGH | What precision policy (number of decimal digits and rounding mode) must the high-precision internal computation apply so results are identical across systems? | Deferred to Discovery; interim baseline: a single high-precision decimal computation strategy is applied service-wide and treated as an implementation detail until stated | project-brief.md §2 Precision Strategy; discovery-transcript.md §03:00 |
| PI-02 | FR-08, FR-08-AC-01 | no | missing | functional_requirements | FR-08 | design_note (retrieval contract) | MEDIUM | What is the history retrieval contract — endpoint path, response shape, and how many recent records are returned? | Deferred to Discovery; interim baseline: FR-08 is emitted with status pending and no endpoint contract; history retrieval is excluded from the MVP phase | project-brief.md §4 History Retrieval |
| PI-03 | NFR-01 | no | missing | non_functional_requirements | NFR-01 | target | HIGH | What numeric latency threshold should "millisecond-level response time" commit to for standard operations? | Deferred to Discovery; interim baseline: millisecond-level interpreted as sub-second response (< 1000 ms) for standard operations | project-brief.md §5 Latency; discovery-transcript.md §09:00 |
| PI-04 | NFR-04 | no | missing | non_functional_requirements | NFR-04 | target | MEDIUM | What concurrency or request-volume target must horizontal scaling support? | Deferred to Discovery; interim baseline: horizontal scaling by adding instances with no numeric concurrency target committed | project-brief.md §5 Scalability; discovery-transcript.md §09:00 |
| PI-05 | NFR-05 | no | missing | non_functional_requirements | NFR-05 | target | MEDIUM | What observability capability must be ready at launch — structured logs only, or also metrics and tracing hooks? | Deferred to Discovery; interim baseline: structured logging sufficient for diagnostics; metrics and tracing integration deferred | project-brief.md §5 Observability readiness |
| PI-06 | NFR-06 | no | missing | non_functional_requirements | NFR-06 | target | HIGH | What security and access-control requirements apply to the calculation API (authentication, network exposure, input limits)? | Deferred to Discovery; interim baseline: service treated as an internal platform component with no additional authentication assumed | not stated in inputs |
| PI-07 | NFR-07 | no | missing | non_functional_requirements | NFR-07 | target | LOW | What usability standard should the API contract commit to, given adoption depends on simplicity? | Deferred to Discovery; interim baseline: simple payload design (operands + operation in, result out) treated as the usability baseline | discovery-transcript.md §07:30, §13:30 |
| PI-08 | KPI-05, KPI-06, KPI-07 | no | missing | kpis | KPI-05, KPI-06, KPI-07 | target | MEDIUM | What numeric targets should replace the qualitative success criteria (teams stop duplicating math logic, results become consistent, calculations traceable)? | Deferred to Discovery; interim baseline: qualitative success criteria tracked as binary (met / not met) until numeric targets are set | discovery-transcript.md §14:30 |
| PI-09 | DEP-01, KPI-01 | no | missing | dependencies | DEP-01 | owner | MEDIUM | Which upstream system is the first integration target for the calculate operation? | Deferred to Discovery; interim baseline: any one upstream system qualifies for the success criterion; integration sequencing left to delivery planning | project-brief.md §7 Success Criteria |
| PI-10 | constraints.budget, constraints.timeline, constraints.resource | no | missing | constraints | budget, timeline, resource | value | MEDIUM | What are the budget, timeline, and staffing constraints for delivering the service? | Deferred to Discovery; interim baseline: no budget ceiling, no calendar deadline, and no staffing cap assumed; scope treated as uncapped until stated | not stated in inputs |
| PI-11 | constraints.regulatory | no | missing | constraints | regulatory | constraint | LOW | Are there regulatory or compliance obligations that apply to the service or its calculation records? | Deferred to Discovery; interim baseline: no regulatory constraints assumed; history treated as diagnostic-only per the discovery transcript | discovery-transcript.md §11:00 ("History is more diagnostic than regulatory") |

### traceability_gaps

| id | affected_ids | blocking | direction | fr_id | jtbd_id | impact | note | fallback_behavior | source_ref |
|----|--------------|----------|-----------|-------|---------|--------|------|-------------------|------------|

### assumptions_to_validate

| id | affected_ids | blocking | assumption | impact_if_wrong | validation_method | fallback_behavior | source_ref |
|----|--------------|----------|------------|-----------------|-------------------|-------------------|------------|
| ASM-01 | P-01, P-02, P-03, P-04 | no | Consumer personas are technically proficient engineering roles | Persona goals and pain points may not reflect actual consumer behavior, skewing API usability prioritization | Confirm consumer roles and proficiency with the product owner | Personas treated as engineering-role consumers for prioritization | project-brief.md §3; discovery-transcript.md §06:00 |
| ASM-02 | FR-04, FR-04-AC-01, LIT-02, LIT-03, LIT-04, LIT-05, LIT-06, LIT-07 | no | The brief's conceptual API contract field names are the intended contract names | Acceptance criteria and API documentation would use field names differing from the final implemented contract | Confirm field names when the API contract is finalized | Field names from project-brief.md §4 used verbatim in acceptance criteria | project-brief.md §4 API Contract |
| ASM-03 | FR-06, DEP-02, LIT-11 | no | History persistence uses a lightweight local embedded database abstraction | Persistence design, deployment footprint, and replaceability guarantees in FR-06 would need rework | Confirm persistence technology choice during architecture definition | Persistence kept behind a replaceable abstraction with no specific engine mandated | project-brief.md §2, §3, alignment note |
| ASM-04 | FR-10, NFR-04 | no | Stateless design is the mechanism that enables horizontal scaling | Scaling behavior would depend on stateful components, invalidating FR-10 acceptance criteria | Confirm no session or in-memory state is introduced during architecture definition | Service designed and verified stateless | project-brief.md §3, §5 |
| ASM-05 | FR-07, NFR-03 | no | The reliability guarantee is stated at both FR and NFR level intentionally | Duplicate coverage could cause inconsistent verification; if unintended, NFR-03 should be merged into FR-07 | Reviewer confirms whether the dual statement is intentional | Both statements retained and verified against the same test (calculation succeeds while persistence fails) | project-brief.md §5 Reliability |
| ASM-06 | FR-04, scope.out_of_scope | no | Synchronous request/response is the only interaction mode required at launch | Asynchronous or batch interaction modes would require additional API surface not covered by FR-04 | Confirm with the product owner that no async or batch mode is needed for the MVP | Only synchronous request/response modeled; batch and async excluded from scope | discovery-transcript.md §06:00, §07:30 |
| ASM-07 | FR-01, FR-02, FR-06, FR-08, mvp_phasing | no | MoSCoW priorities inferred from source phrasing (standard operations first, history optional, future capabilities planned) | Phasing would change; capabilities assumed must-have could be deferred or vice versa | Confirm priorities with the product owner | Phasing follows the inferred MoSCoW assignment in mvp_phasing | project-brief.md §2; discovery-transcript.md §01:30, §07:30 |

### summary
- total_frs: 10
- total_nfrs: 7
- total_jtbds: 4
- total_kpis: 7
- pending_inputs: 11
- traceability_gaps: 0
- assumptions: 7
- risks: 4
- dependencies: 2
- status: draft