# Development Workflow Pipeline

Orchestrate the full development workflow pipeline sequentially. Each execution is isolated in its own feature folder under `workflow-output/`, preserving history and serving as documentation.

## Usage

```
/dev-workflow <PRD content, feature description, or card number>
```

`$ARGUMENTS` contains the PRD content or card number provided by the user.

---

## Step 0 — Resolve Output Directory

Follow the **Branch Resolution** procedure defined in `.claude/rules/WORKFLOW_CONVENTIONS.md`.

All subsequent file paths use `OUTPUT_DIR` as the base.

---

## Pipeline Execution

Execute the following steps **sequentially** — wait for each to complete before starting the next.

---

### Step 1 — PRD Review (`prd-story-reviewer`)

Use the **Agent tool** to invoke the `prd-story-reviewer` subagent.

**Prompt to pass** (replace `OUTPUT_DIR` with the resolved path):
```
Review and validate the following PRD or user story:

$ARGUMENTS

After completing the full review and clarification dialogue, write the final validated PRD summary to the file `OUTPUT_DIR/prd-review.md` using the Write tool.

The file must follow this exact structure:

---
# PRD Review — [Feature Title]
generated_at: [ISO date]
status: [Ready for Development | Needs Further Input | Blocked]
---

## Objective
[One-sentence business goal]

## Users Affected
[Who this impacts]

## Acceptance Criteria (Validated)
1. [Criterion 1]
2. [Criterion 2]

## Out of Scope
[Explicit exclusions]

## Dependencies & Constraints
[Key dependencies]

## Open Items
[Unresolved items, or "None"]

Do not skip writing the file — this output feeds the next agent in the pipeline.
```

Wait for this step to complete and confirm `OUTPUT_DIR/prd-review.md` was written.

---

### Step 2 — Bench Test Design (`bench-test-architect`)

Use the **Agent tool** to invoke the `bench-test-architect` subagent.

**Prompt to pass** (replace `OUTPUT_DIR` with the resolved path):
```
Read the validated PRD at `OUTPUT_DIR/prd-review.md` and generate a comprehensive bench test suite based on it.

After generating the full test suite, write it to `OUTPUT_DIR/bench-test.md` using the Write tool.

The file must follow this exact structure:

---
# Bench Test Suite — [Feature Title]
generated_at: [ISO date]
source: OUTPUT_DIR/prd-review.md
---

## Test Suite: [Feature Name]

### Group: Happy Path
[tests...]

### Group: Sad Path — Invalid Inputs
[tests...]

### Group: Sad Path — Error States
[tests...]

### Group: Edge Cases
[tests...]

### Group: Regression Guards
[tests...]

## Coverage Summary
[Brief coverage summary and any assumptions made]

Do not skip writing the file — this output feeds the next agent in the pipeline.
```

Wait for this step to complete and confirm `OUTPUT_DIR/bench-test.md` was written.

---

### Step 3 — Pipeline Summary

After both steps complete, report to the user:

```
## Pipeline Complete

**Feature:** <branch/feature name>
**Output directory:** `<OUTPUT_DIR>/`

**Artifacts generated:**
- `prd-review.md`  — Validated PRD summary
- `bench-test.md`  — Bench test suite

**Next steps available:**
- `/dev-workflow-tests`  — Generate TDD unit and integration tests from bench-test.md
- `/dev-workflow-code`   — Write implementation code (after tests are in place)
- `/dev-workflow-review` — Review generated code for quality and standards

To continue this workflow later, all artifacts are in `<OUTPUT_DIR>/`.
```

---

## Notes

- **One folder per feature:** Each run produces an isolated `workflow-output/<branch>/` folder. Nothing is ever overwritten across features.
- **Resumable:** If you need to rerun a step, delete or replace the corresponding `.md` file in the feature folder and rerun.
- **Card numbers:** If `$ARGUMENTS` is a card number (e.g., `#4521`, `US-892`), `prd-story-reviewer` will fetch it from Azure DevOps.
- **Folder as documentation:** The `workflow-output/` tree is a historical record of every feature's design and test artifacts.
