# AI Pods Challenge - CalcService

This challenge falls in the **[Digital Delivery](https://ai-pods-handbook.globant.com/04-framework/04-playlists-catalog/)** standard playlist category of the [Standard Playlists Catalog](https://ai-pods-handbook.globant.com/04-framework/04-playlists-catalog/), running the **API Product Launch** playlist: a production-ready REST API is designed schema-first, hardened for security from the start, and validated end-to-end — downstream integrators get a stable contract and a tested, deployed API at cutover.

This guide walks you through the Basic Challenge of the AI Pods, during which you will use the AI Pods production system to orchestrate the end-to-end delivery of a stateless mathematical operations microservice. The challenge for you is not to write the service "manually", even by manually indicating to Coda CLI or other AI agents the tasks. Rather, the challenge is to **launch the API Product Launch playlist** and follow its agentic flow end to end. This means that you, as the **human expert** involved in the process, will validate, guide, and decide upon the plans and artifacts created by the AI agents at each quality gate.

**NOTE:** `CalcService` is a backend-only, stateless microservice with no UI. The API Product Launch playlist fits this cleanly — unlike a general product-launch playlist, it has no UX-design or prototyping steps to work around.

## Prerequisites

- Install [CODA & Stepwise](https://ai-pods-handbook.globant.com/06-tools-platforms/03a-stepwise-installation/)
- Follow the [Challenge Rules](https://docs.google.com/document/d/1uYsAySJhXaECfQ09dCCPDKclQ2Qg7B1SSAegMfN0dTg)
- Read [Playlists and Multi-Capability Orchestration](https://ai-pods-handbook.globant.com/06-tools-platforms/03-stepwise-user-manual/#playlists-and-multi-capability-orchestration) so you know what to expect from a Playlist run (DAG execution, quality gates, Tri-State Filter).

**NOTE:** This challenge is distributed as a local project folder, not a GitHub repository. Do not look for it, fork it, or request access to it on GitHub — follow the Challenge Rules above for how to obtain and submit it.

## Installation Steps

1. Extract the downloaded challenge folder to your local machine.
2. Run stepwise (cli or web) from the extracted folder.

    a. Go to folder:
    ```bash
    # This is an example, update to the correct path in your local machine.
    cd /path/to/extracted/folder
    ```

    b. Initialize a local git repository (if not already initiated). This is for your own local version control only — it is not connected to, and does not require, any GitHub repository.
    ```bash
    # This is an example, update to the correct path in your local machine.
    git status

    # If you get the following response:
    # fatal: not a git repository (or any of the parent directories): .git
    # Run:
    git init
    ```
    It is recommended to commit the steps executed and approved so you can rollback in case a session generates artifacts that are too corrupted.

    c. Run Stepwise:
    ```bash
    stepwise registry sync && \
    stepwise
    ```
    The `stepwise` command should launch a browser window to `https://127.0.0.1:3000`

**Reminder — LLM configuration:** before touching the Context Pack or launching the playlist, confirm the self-hosted **GLM-5.3-Flash** model is configured in both CODA and the Stepwise UI, as described in [Phase 1: Set Up Your Delivery Factory](https://ai-pods-handbook.globant.com/12-training-enablement/03-training-resources/#phase-1-set-up-your-delivery-factory) of the Training Resources. Use `stepwise registry list` to confirm the setup — any session run with the wrong model, including a Context Pack calibration pass, is not representative of the challenge.

## Verify Input & Context Pack

Before launching the playlist, review the provided `context-pack/` and `artifacts/inputs/` folders so you understand the constraints the AI agents are expected to follow:

- `context-pack/tech-policy.md` — Required technology stack (Java 8, Spring Boot 2.2.7).
- `context-pack/arch-standards.md` — Domain boundaries and system architecture for the microservice.
- `context-pack/coding-standards.md` — Naming and coding conventions to be enforced.
- `artifacts/inputs/project-brief.md` — The business brief the AI agents will use as their starting point.

**The Context Pack is not fixed.** If a quality gate review surfaces a repeatable gap — a hallucinated library, a style violation, a misunderstood business rule — update the relevant `context-pack/` file (e.g. `tech-policy.md`, `coding-standards.md`, `arch-standards.md`) rather than only rejecting the artifact and re-prompting manually. This is the **Calibration Loop**: see [Context Calibration](https://ai-pods-handbook.globant.com/05-delivery/06-calibration-cycle/) for the full pattern (identify the failing pattern → update the Context Pack → re-run the step to confirm the fix holds). Keep the edits explicit and commit them so the fix persists for the rest of the run.

## Launch the API Product Launch Playlist

1. Navigate to **Playlists** in the Stepwise sidebar.
2. Click **Launch** next to **API Product Launch**.
3. Provide a **global context** description of the CalcService project — this is injected as shared context across every capability in the run.
4. Provide a **Session name** when prompted (this becomes the Git branch Stepwise creates and auto-commits to for this run).

Once launched, the playlist auto-advances through the following capabilities. `software-architecture` and `product-delivery` run in parallel once `product-definition` is approved. You interact primarily at each quality gate.

**NOTE for non-specialist reviewers (product/architecture/QE):** if you are doing this challenge on your own and are not used to a given discipline, a strong quality gatekeeping is not expected at that gate — just verify the key artifacts exist and are reasonably coherent.

| # | Capability (as it runs in this playlist) | Key artifacts to verify |
|---|---|---|
| 1 | `product-definition` | PRD: `/artifacts/outputs/product-definition/prd/` · Epics: `/artifacts/outputs/product-definition/epics/` |
| 2a | `software-architecture` *(parallel with 2b)* | ADRs: `/artifacts/outputs/software-architecture/adrs/` · Target Architecture: `/artifacts/outputs/software-architecture/target-architecture/` |
| 2b | `product-delivery` *(parallel with 2a)* | User stories: `/artifacts/outputs/product-delivery/user-stories/` |
| 3 | `quality-engineering-planning` | QA Master Test Strategy & Plan, E2E Test Cases: `/artifacts/outputs/quality-engineering-planning/` |
| 4 | `quality-engineering-design` | QE Design Test Cases: `/artifacts/outputs/quality-engineering-design/test-cases/` |
| 5 | `code-development` | Research output: `/artifacts/outputs/code-development/research-output/` · Planning output: `/artifacts/outputs/code-development/code-task-planning/` · Code: `./source`. This capability consumes all upstream validated context (PRD, Architecture, User Stories, Test Plan) — verify the generated code aligns with the approved architecture. |
| 6 | `local-runtime-validation` | Confirms the service boots locally and runs without errors before automation. |
| 7 | `quality-engineering-web-automation` | Automation Analysis: `/artifacts/outputs/quality-engineering-web-automation/automation-analysis` · Validated test cases: `/artifacts/outputs/quality-engineering-web-automation/validated-test-cases`. Confirm this exercises the REST endpoints end-to-end. |

**Be aware of the prerequisites for step 7 (`quality-engineering-web-automation`):**
- Access to the running service (locally or deployed)
- MCP web-inspector server configured
- Network access to the target environment

Only after you have this done can the playlist continue with test-case validation.

## Submission

Per the Challenge Rules, your submission is this entire working folder, compressed into a `.zip` file — including every file under `artifacts/`, `./source`, and `./context-pack/`. If you modified any `context-pack/` file during calibration, keep those changes: they are part of the deliverable and show how you tuned the system, not just what the agents produced.
