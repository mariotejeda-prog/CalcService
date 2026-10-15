# Domain Expert Workbench — Stepwise Validation Guide

**Purpose:**
This page exists to orient domain experts on how to use the Stepwise workbench to progress through the process and validate artifacts at each quality gate.

This is an execution surface, not documentation.

---

## What You Are Looking At

The workbench has **two functional areas**:

### Left: Stepwise Control & Logs

* **Stepwise panel**: Drives the process forward, step by step
* **Quality gates**: Where you approve or fail a step
* **Coda log console**: Execution and system logs for traceability (read-only)

### Right: Artifact Viewer (Web)

* Displays the **artifacts generated for the current step**
* Shows folders such as:

  * Inputs
  * Outputs
  * Transcripts
  * Intermediate artifacts
* This is where validation happens

---

## Your Role

You are the **domain authority** for correctness and fitness-for-purpose.

**You are responsible for:**

* Reviewing artifacts produced at each step
* Deciding whether they meet the step’s objective
* Approving or failing the quality gate

**You are not responsible for:**

* Running commands manually
* Fixing generation logic
* Guessing intent when something is unclear

If an artifact is unclear, it is invalid.

---

## How the Stepwise Flow Works

Each step follows the same pattern:

1. **Stepwise executes a step**
2. **Artifacts are generated and written to folders**
3. **You review those artifacts on the right**
4. **You decide at the quality gate on the left**

There is no automatic progression.
Nothing moves forward without your decision.

---

## How to Validate Artifacts

Validation is a **go / no-go decision**, not a review pass.

When reviewing artifacts, check:

* **Correctness**

  * Are the facts, assumptions, and logic correct?
* **Completeness**

  * Are all required sections or elements present?
* **Clarity**

  * Could another practitioner execute this as-is?
* **Risk**

  * Would an error here cascade into later steps?

Use the artifact folders directly. Do not rely on summaries.

---

## Your Two Quality Gate Decisions

At each gate, you must choose one.

### ✅ Pass (Confirm Validity)

Use when:

* Artifacts meet the step objective
* No blocking issues remain

**Result:** Stepwise advances to the next step.

---

### ❌ Fail (Provide Feedback)

Use when:

* Artifacts are incorrect, incomplete, or ambiguous
* Assumptions are wrong or missing
* Execution would be blocked downstream

**How to fail properly:**

* State what is wrong
* State what must change
* Be explicit and bounded

**Result:** Step is re-executed with your feedback applied.

Failing early is expected and correct.

---

## How to Use Artifact Folders

The folders (inputs, outputs, transcripts, etc.) are **not decorative**.

They exist so you can:

* Trace where information came from
* Verify transformations
* Confirm nothing critical was dropped or misinterpreted

If required context is missing from the folders, fail the gate.

---

## Escalation Rule

If you cannot validate because:

* You lack authority
* Information is missing
* The step objective is unclear

**Stop and escalate.**
Do not approve “for now.”

---

## Core Mental Model

Think of Stepwise as a **controlled execution line**:

* Stepwise runs steps
* Artifacts are produced
* You validate reality against artifacts
* Quality gates enforce correctness

**AI proposes.
You decide.
The system enforces.**

Your judgment is the control point.
