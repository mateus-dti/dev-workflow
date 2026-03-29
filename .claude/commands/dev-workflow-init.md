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

This step runs in **three phases** to enable interactive gap resolution with the user before the final artifact is written.

---

#### Phase 1a — Gap Analysis

Use the **Agent tool** to invoke the `prd-story-reviewer` subagent.

**Prompt to pass** (replace `OUTPUT_DIR` with the resolved path):
```
Perform a gap analysis on the following PRD or user story. Do NOT write the final prd-review.md yet.

--- BEGIN PRD ---
$ARGUMENTS
--- END PRD ---

Your output for this phase is a structured gap report written to `OUTPUT_DIR/prd-analysis-draft.md`.

The file must follow this exact structure:

---
# PRD Gap Analysis — [Feature Title]
generated_at: [ISO date]
phase: gap-analysis
---

## Summary
[2-3 sentence overview of what the PRD covers and its current maturity level]

## Strengths
- [What is well-defined]

## Blockers
[Each item on its own line, prefixed with "BLOCKER:" — critical gaps that prevent development from starting. If none, write "None."]

## Ambiguities
[Each item on its own line, prefixed with "AMBIGUITY:" — statements that can be interpreted in two or more ways. For each, list the competing interpretations. If none, write "None."]

## Assumption Risks
[Each item on its own line, prefixed with "ASSUMPTION RISK:" — details that are absent and would require inventing business rules. If none, write "None."]

## Questions for User
[Numbered list of questions, one per gap, ordered: Blockers first, then Ambiguities, then Assumption Risks. If no gaps exist, write "None — PRD is complete."]
```

Wait for Phase 1a to complete and confirm `OUTPUT_DIR/prd-analysis-draft.md` was written.

---

#### Phase 1b — Interactive Clarification

Read `OUTPUT_DIR/prd-analysis-draft.md` and check the **Questions for User** section.

**If there are no questions** (the section says "None — PRD is complete."), skip directly to Phase 1c.

**If there are questions**, present them to the user directly in this conversation — do NOT invoke another agent for this. Group them by category (Blockers → Ambiguities → Assumption Risks) and ask **at most 3 questions per round** to avoid overwhelming the user.

Use this format when presenting questions:
```
Before I finalize the PRD review, I need some clarifications:

**[Category: Blocker / Ambiguity / Assumption Risk]**
1. [Question]
2. [Question]
3. [Question]

(If there are more questions remaining after these, I'll ask them after you answer.)
```

Wait for the user's response before continuing. After each response:
- Acknowledge the answers
- Check if they resolve all remaining gaps or introduce new ones
- If new gaps appear, ask about them in the next round
- Repeat until all Blockers and Ambiguities are resolved (Assumption Risks that the user explicitly defers may remain as Open Items)

Collect all answers into a running context block for Phase 1c.

---

#### Phase 1c — Final Review

Use the **Agent tool** to invoke the `prd-story-reviewer` subagent again.

**Prompt to pass** (replace `OUTPUT_DIR` with the resolved path and `USER_ANSWERS` with the collected answers from Phase 1b — if no questions were asked, write "No clarifications were needed."):
```
Finalize the PRD review using the original PRD and the user's clarification answers below.

--- BEGIN PRD ---
$ARGUMENTS
--- END PRD ---

--- USER ANSWERS ---
USER_ANSWERS
--- END USER ANSWERS ---

Also read the gap analysis at `OUTPUT_DIR/prd-analysis-draft.md` for context on what was resolved.

Write the final validated PRD summary to `OUTPUT_DIR/prd-review.md`.

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
[Any Assumption Risks explicitly deferred by the user, or items still awaiting stakeholder input. Write "None" only if every gap was fully resolved.]

Do not skip writing the file — this output feeds the next agent in the pipeline.
```

Wait for Phase 1c to complete and confirm `OUTPUT_DIR/prd-review.md` was written.

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
