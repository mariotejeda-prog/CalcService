# Active Context — CalcService product-definition

## Session Status
- capability: product-definition
- session: product-calcservice
- current step: epics backlog generation (planning-epics) — COMPLETED 2026-10-28
- artifact type: 8 epics

## Key Artifacts
- PRD: artifacts/outputs/product-definition/prd/PRD-SPEC-pd-20261027-calc001.md (v1.0.1, content_hash 25c7814b3af060433c725d5e0d92674538e4af564960493da328ea935e63f93a)
- Epics backlog: artifacts/outputs/product-definition/epics/EPICS-SPEC-pd-20261028-calc001.md (v1.0.0, BUILD)
- Epics audit: artifacts/outputs/product-definition/epics/EPICS-AUDIT-pd-20261028-calc001.md

## Decisions
- 8 epics across 4 themes: EPIC-01 standard arithmetic calculation core, EPIC-02 calculate API contract, EPIC-03 extensible operation registry, EPIC-04 advanced operations extension, EPIC-05 optional history persistence, EPIC-06 history retrieval, EPIC-07 stateless deployable service shell (enabler), EPIC-08 API contract documentation (enabler)
- Build order (topological): EPIC-01 → EPIC-07 → EPIC-02 → EPIC-03 → EPIC-08 → EPIC-04 → EPIC-05 → EPIC-06
- FR coverage 10/10; all 7 NFRs embedded as DoD constraints; 45 PRD IDs owned exactly once
- FR-07/NFR-03 dual statement retained, verified against the same test in EPIC-05 (per OQ-abad2154)
- FR-08 history retrieval: no endpoint contract; excluded from MVP per OQ-025246e3

## Blockers / Open Items
- 1 assumption raised this run: ASM-E01 (T-04 theme grouping of EPIC-07 + EPIC-08) — needs product-owner review
- Upstream PRD pending inputs PI-01..PI-11 remain open in the PRD ledger (cited, not re-raised)

## Next Steps
- Downstream: user stories / architecture / implementation consume EPICS-SPEC-pd-20261028-calc001.md
- Adopt `## sequencing → topological_order` as the authoritative build order (epic id order is NOT build order)
