# Development Workflow — Review

Review and polish the implementation code produced by `dev-workflow-code`. Delegates to the `code-beautifier-reviewer` agent, which audits every file against the project's standards, applies corrections directly, and produces a review report. This is the final quality gate before the feature is considered done.

## Usage

```
/dev-workflow-review
```

No arguments required. All inputs are read from the feature's `workflow-output/` folder and the project source tree.

---

## Step 0 — Resolve Output Directory

Follow the **Branch Resolution** procedure defined in `.claude/rules/WORKFLOW_CONVENTIONS.md`.

---

## Step 1 — Validate Prerequisites

Using the Read tool, check that required inputs exist:

| File | Required | Purpose |
|------|----------|---------|
| `OUTPUT_DIR/code-report.md` | **Yes** | List of implementation files to review |
| `OUTPUT_DIR/tests-report.md` | No | Test file list (to avoid reviewing test files as implementation) |
| `OUTPUT_DIR/prd-review.md` | No | Acceptance criteria to cross-check against |

- If `code-report.md` **does not exist** → stop and inform the user:
  *"`OUTPUT_DIR/code-report.md` not found. Run `/dev-workflow-code` first to generate the implementation."*

- If `code-report.md` lists any failing tests → warn the user before proceeding:
  *"Warning: `code-report.md` shows failing tests. It is recommended to fix them before reviewing. Proceed anyway? (yes/no)"*

Read all available files before invoking the agent.

---

## Step 2 — Review Implementation (`code-beautifier-reviewer`)

Use the **Agent tool** to invoke the `code-beautifier-reviewer` subagent.

**Prompt to pass** (replace placeholders with resolved values):

```
Review and polish all implementation files for this feature.

<feature_branch>BRANCH_NAME</feature_branch>

<code_report>
CODE_REPORT_CONTENT_HERE
</code_report>

<prd_context>
PRD_CONTENT_HERE (or "Not available.")
</prd_context>

<tests_report>
TESTS_REPORT_CONTENT_HERE (or "Not available.")
</tests_report>

Follow your full review methodology:

1. Context Gathering: examine CLAUDE.md, linting configs, and exemplary existing files to internalize the project's standards, naming conventions, and architectural patterns.

2. Systematic Audit: review every implementation file listed in the code-report. Do NOT review or modify test files.

3. Make Changes: apply corrections directly to the files — naming, structure, abstraction, DRY/SOLID, error handling, imports, type safety. Fix what is wrong; do not rewrite what is acceptable.

4. Self-Verification: confirm all tests still pass after your changes by running the test suite.

5. Produce a review report with:
   - Overall quality assessment (2-5 sentences)
   - Changes made per file, grouped by category (Naming, Structure, Standards, etc.) with before/after for significant changes
   - Final test run result confirming no regressions
   - Flagged issues outside your scope (logic bugs, security concerns, missing coverage) — listed separately, not silently fixed
   - A final `status` verdict: `approved` if no flagged issues remain, `needs-work` if any flagged issues require attention before merging

Return this report as your final output.
```

Wait for the agent to complete and capture its final review report output.

---

## Step 3 — Save Review Report

Write the report returned by `code-beautifier-reviewer` to `OUTPUT_DIR/review-report.md` using the Write tool.

The file must follow this structure:

```
---
# Review Report — [Feature Title]
generated_at: [ISO date]
branch: [branch name]
source: OUTPUT_DIR/code-report.md
status: [approved | needs-work]
---

## Quality Assessment
[2-5 sentence overall summary]

## Changes Made
### Naming
- [file]: [what changed and why]

### Structure
- [file]: [what changed and why]

### Standards Compliance
- [file]: [what changed and why]

### Error Handling
- [file]: [what changed and why]

### Other
- [file]: [what changed and why]

## Final Test Run
[pass/fail counts confirming no regressions]

## Flagged Issues (Outside Scope)
[logic bugs, security concerns, missing coverage — or "None"]

## Verdict
**[APPROVED | NEEDS WORK]** — [one sentence justification]
```

---

## Step 4 — Generate Feature Index

Write `OUTPUT_DIR/INDEX.md` using the Write tool. This file is the navigable table of contents for the feature — generated once, at the end of the review.

Read the frontmatter of each existing artifact to extract `generated_at` and `status` fields. The file must follow this structure:

```
---
# Feature Index — [Feature Title]
branch: [branch name]
last_updated: [ISO date]
workflow_status: [approved | needs-work | in-progress]
---

## Feature Overview
[One-sentence objective extracted from prd-review.md]

## Workflow Artifacts

| Step | Artifact | Generated At | Status |
|------|----------|--------------|--------|
| 1 | [prd-review.md](prd-review.md) | [date] | [status or —] |
| 2 | [bench-test.md](bench-test.md) | [date] | — |
| 3 | [boilerplate-report.md](boilerplate-report.md) | [date or "skipped"] | — |
| 4 | [tests-report.md](tests-report.md) | [date] | — |
| 5 | [code-report.md](code-report.md) | [date] | — |
| 6 | [review-report.md](review-report.md) | [date] | [approved/needs-work] |

## Acceptance Criteria
[List extracted from prd-review.md]

## Next Steps
[If status is approved: "Ready for PR — run `/dev-workflow-pr`"]
[If status is needs-work: "Address flagged issues in review-report.md before proceeding"]
```

`workflow_status` rules:
- `approved` — review-report.md has `status: approved`
- `needs-work` — review-report.md has `status: needs-work`
- `in-progress` — review-report.md does not exist yet

---

## Step 5 — Summary

Report to the user:

```
## Review Complete

**Feature:** <branch/feature name>
**Output directory:** `<OUTPUT_DIR>/`

**Artifacts generated:**
- `review-report.md` — Quality assessment, changes applied, and verdict
- `INDEX.md`         — Navigable index of all feature artifacts

**Test status:** <X passing — no regressions>

**Verdict:** <APPROVED / NEEDS WORK> — <justification>

**Next steps available:**
- `/dev-workflow-pr`  — Generate PR title, description, and checklist from all artifacts
```

---

## Notes

- The reviewer applies changes **directly to source files** — not to `workflow-output/`.
- Test files are never touched. If the reviewer finds issues in a test, they are flagged in the report.
- `review-report.md` closes the audit trail: every decision from PRD to polished code is documented in `OUTPUT_DIR/`.
- If flagged issues exist, address them manually or re-run `/dev-workflow-code` before merging.
