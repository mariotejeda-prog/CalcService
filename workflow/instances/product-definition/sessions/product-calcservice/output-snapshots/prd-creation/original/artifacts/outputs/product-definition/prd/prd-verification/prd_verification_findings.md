# PRD Verification Findings

**Verdict:** ❌ FAILED  
**Blockers failed:** 1  
**Warnings failed:** 1  
**Checked at:** 2026-10-28T11:36:23Z

## Findings

| Check | Weight | Status | Description | Detail |
|-------|--------|--------|-------------|--------|
| RV1 | BLOCKER | PASS ✅ | No upstream blocker failures reported. |  |
| RV2 | BLOCKER | FAIL ❌ | project_info contains placeholder text in 1 field(s). | `client_name` = `not stated in inputs`. Fill these from the project brief — placeholders indicate the PRD was not tailored to the actual project. |
| RV5 | WARNING | FAIL ❌ | mvp_phasing contains 3 bare ID(s) without requirement names. | Bare IDs: ASM-07, ASM-07, ASM-07. The roadmap must be readable on its own — name each requirement alongside its ID so a reader does not need to cross-reference other sections. |
