# Extraction: PRD-SPEC-pd-20261027-calc001.md
source: artifacts/outputs/product-definition/prd/PRD-SPEC-pd-20261027-calc001.md (589 lines)
extracted_at: 2026-10-28T11:52:17Z
purpose: planning-epics Phase B input — every FR, NFR, JTBD, KPI, persona, risk, assumption, dependency, literal, scope item, phasing entry

## project_info (verbatim)
- client_name: Internal platform organization (client organization not named in source inputs; service built for internal platform teams)
- project_name: Math Operations Microservice (Calc-Service)
- project_description: A lightweight, stateless microservice that provides a centralized, deterministic mathematical computation capability for the platform, externalizing calculation logic from client applications to enable consistency, reuse, and controlled evolution of numerical behavior across the ecosystem.

## content_hash
- 25c7814b3af060433c725d5e0d92674538e4af564960493da328ea935e63f93a (transcribe verbatim as prd_source_hash)

## FRs (10)
- FR-01 Execute standard arithmetic operations (addition, subtraction, multiplication, division) | must_have | depends: none | PRD:FR-01
- FR-02 Execute advanced operations (exponentiation, roots, trigonometric functions) | should_have | depends: FR-01 | PRD:FR-02
- FR-03 Produce deterministic, high-precision results | must_have | depends: FR-01 | precision policy pending (PI-01) | PRD:FR-03
- FR-04 Expose a calculate operation via `POST /api/v1/math/calculate` | must_have | depends: FR-01 | field names assumption ASM-02 | PRD:FR-04
- FR-05 Return clear, fail-fast errors for invalid input | must_have | depends: FR-01 | PRD:FR-05
- FR-06 Record calculation history as optional, auxiliary local persistence | could_have | depends: FR-01 | embedded DB abstraction ASM-03 | PRD:FR-06
- FR-07 Calculation must succeed even if history persistence fails | must_have | depends: FR-06 | PRD:FR-07
- FR-08 Provide history retrieval of recent calculation records | could_have | depends: FR-06 | endpoint contract pending (PI-02 / OQ-025246e3) | PRD:FR-08
- FR-09 Support extensibility so new operations are added without breaking existing consumers | must_have | depends: FR-01 | PRD:FR-09
- FR-10 Operate as a stateless, independently deployable, horizontally scalable service | must_have | depends: none | stateless mechanism ASM-04 | PRD:FR-10

## NFRs (7)
- NFR-01 performance | millisecond-level response for standard operations [PENDING numeric threshold] | pending | PRD:NFR-01
- NFR-02 compliance | 100% identical resultValue on repeated identical requests | complete | PRD:NFR-02
- NFR-03 availability | 100% of valid calculate requests return a result when history persistence unavailable | complete | mirrored as FR-07 (ASM-05) | PRD:NFR-03
- NFR-04 scalability | horizontal scaling by adding instances, no behavior change [PENDING numeric target] | pending | PRD:NFR-04
- NFR-05 usability | observability readiness for future metrics/tracing [PENDING target] | pending | PRD:NFR-05
- NFR-06 security | [PENDING target — no security requirement in inputs] | pending | PRD:NFR-06
- NFR-07 usability | [PENDING target — simple payload baseline] | pending | PRD:NFR-07

## KPIs (7)
- KPI-01 at least 1 upstream system integrated | complete | PRD:KPI-01
- KPI-02 100% deterministic unit test coverage of all operations | complete | PRD:KPI-02
- KPI-03 optional history persistence validated (no response impact) | complete | PRD:KPI-03
- KPI-04 API contract documented for internal reuse | complete | PRD:KPI-04
- KPI-05 reduction of duplicated math logic [PENDING numeric target] | pending | PRD:KPI-05
- KPI-06 result consistency across systems [PENDING numeric target] | pending | PRD:KPI-06
- KPI-07 calculation traceability availability [PENDING numeric target] | pending | PRD:KPI-07

## JTBDs (4)
- JTBD-01 core_functional: single authoritative math engine, deterministic result for any caller | PRD:JTBD-01
- JTBD-02 related: evolve numerical behavior in one controlled place without breaking consumers | PRD:JTBD-02
- JTBD-03 related: trace/debug calculations via recorded metadata; retrieval never degrades calculation | PRD:JTBD-03
- JTBD-04 related: lightweight deterministic calculations inside automated workflows / AI processes | PRD:JTBD-04

## Personas (4)
- P-01 Backend Service Developer (primary) | PRD:P-01
- P-02 Automation Workflow Engineer (secondary) | PRD:P-02
- P-03 AI Agent / AI-Driven Process Operator (secondary) | PRD:P-03
- P-04 Reporting Pipeline Engineer (secondary) | PRD:P-04

## Risks (4)
- RSK-01 Hidden logic differences remain between systems after launch (medium/high) | PRD:RSK-01
- RSK-02 Teams bypass the service and keep embedding their own logic (medium/high) | PRD:RSK-02
- RSK-03 History persistence degrades calculation response delivery (low/medium) | PRD:RSK-03
- RSK-04 Precision policy gaps reintroduce floating-point surprises (medium/medium) | PRD:RSK-04

## Assumptions (7)
- ASM-01 personas are technically proficient engineering roles | PRD:ASM-01
- ASM-02 brief's conceptual API contract field names are intended contract names | PRD:ASM-02
- ASM-03 history persistence uses lightweight local embedded database abstraction | PRD:ASM-03
- ASM-04 stateless design enables horizontal scaling | PRD:ASM-04
- ASM-05 reliability guarantee stated at both FR and NFR level intentionally | PRD:ASM-05
- ASM-06 synchronous request/response only interaction mode at launch | PRD:ASM-06
- ASM-07 MoSCoW priorities inferred from source phrasing | PRD:ASM-07

## Dependencies (2)
- DEP-01 external: at least one upstream system must integrate (owner pending, PI-09) | PRD:DEP-01
- DEP-02 technical: embedded persistence abstraction for optional history | PRD:DEP-02

## Literal registry (11)
- LIT-01 route POST /api/v1/math/calculate | LIT-02 operand1 | LIT-03 operand2 | LIT-04 operation | LIT-05 resultValue | LIT-06 operationType | LIT-07 timestamp | LIT-08 enum ADD | LIT-09 example values 10.5/5.0/ADD→15.5 | LIT-10 latency goal millisecond-level | LIT-11 H2-based lightweight persistence strategy

## mvp_phasing
- phase_1_mvp (must_have): FR-01, FR-03, FR-04, FR-05, FR-09, FR-10 (assumption ASM-07)
- phase_2 (should_have): FR-02
- phase_3 (could_have): FR-06, FR-08
- deferred (wont_have): batch calculations, expression evaluation, workflow integration with automation agents, event-based history

## out_of_scope
- batch calculations; expression evaluation; workflow integration with automation agents; event-based history; multi-instance history consistency; regulatory-grade audit persistence

## constraints
- budget/timeline/resource/regulatory: PENDING (PI-10, PI-11)
- organizational: history diagnostic not regulatory; persistence replaceable without API contract change

## PRD open questions (pending_inputs PI-01..PI-11) — inherited, cite OQ- ids, never re-list
- PI-01 precision policy (FR-03) — OQ raised by prd-creation
- PI-02 history retrieval contract (FR-08) — OQ-025246e3
- PI-03 latency threshold (NFR-01)
- PI-04 concurrency target (NFR-04)
- PI-05 observability target (NFR-05)
- PI-06 security requirements (NFR-06) — OQ-8a3647b2
- PI-07 usability standard (NFR-07)
- PI-08 numeric KPI targets (KPI-05/06/07) — OQ-c5ed68d9
- PI-09 first integration target (DEP-01, KPI-01) — OQ-87c0d49a
- PI-10 budget/timeline/staffing — OQ-43d9b8b1
- PI-11 regulatory obligations — OQ-7046793c
- assumptions_to_validate: ASM-01..ASM-07 (all active, none blocking)
