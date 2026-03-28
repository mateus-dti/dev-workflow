# Development Workflow — Code

Write the implementation code to make the failing TDD tests pass. Delegates to the `code-writer` agent, which reads the test files and all available feature context to produce working code as directly as possible.

## Usage

```
/dev-workflow-code
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
| `OUTPUT_DIR/tests-report.md` | **Yes** | Map of test files to implement against |
| `OUTPUT_DIR/bench-test.md` | **Yes** | Scenario specs to guide implementation |
| `OUTPUT_DIR/prd-review.md` | No | Business rules and acceptance criteria |
| `OUTPUT_DIR/boilerplate-report.md` | No | Stack and project structure reference |

- If `tests-report.md` **does not exist** → stop and inform the user:
  *"`OUTPUT_DIR/tests-report.md` not found. Run `/dev-workflow-tests` first to generate the test suite."*

- If `bench-test.md` **does not exist** → stop and inform the user:
  *"`OUTPUT_DIR/bench-test.md` not found. Run `/dev-workflow-init` first."*

Read all available files and pass their contents to the agent.

---

## Step 2 — Write Implementation Code (`code-writer`)

Use the **Agent tool** to invoke the `code-writer` subagent.

**Prompt to pass** (replace placeholders with resolved values):

```
Write the implementation code to make the failing tests pass for this feature.

<feature_branch>BRANCH_NAME</feature_branch>

<tests_report>
TESTS_REPORT_CONTENT_HERE
</tests_report>

<bench_test_spec>
BENCH_TEST_CONTENT_HERE
</bench_test_spec>

<prd_context>
PRD_CONTENT_HERE (or "Not available.")
</prd_context>

<stack_context>
BOILERPLATE_REPORT_CONTENT_HERE (or "Not available — detect from project files.")
</stack_context>

Your goal is to make every test in the tests-report pass. Follow this process:

1. Read each test file listed in the tests-report to understand exactly what is being tested.
2. Identify the implementation files that need to be created or modified (infer from test imports and the project structure).
3. Write the implementation code — one module or file at a time — making the simplest code that satisfies the tests.
4. After writing all implementation files, run the test suite using the appropriate command (from boilerplate-report or detect it). Report the results.
5. If any tests still fail after the first pass, fix the implementation until all tests pass.

Rules:
- Write code that WORKS. Correctness over elegance.
- Do not modify the test files — only write implementation code.
- Use the stack and conventions from boilerplate-report if available.
- Hardcode values or use simple approaches if they make the tests pass faster.

At the end, produce a structured code report with:
- List of implementation files created or modified
- Final test run result (pass/fail counts)
- Any tests that could not be made to pass and why
- Brief notes on key implementation decisions

Return this report as your final output.
```

Wait for the agent to complete and capture its final code report output.

---

## Step 3 — Save Code Report

Write the report returned by `code-writer` to `OUTPUT_DIR/code-report.md` using the Write tool.

The file must follow this structure:

```
---
# Code Report — [Feature Title]
generated_at: [ISO date]
branch: [branch name]
source: OUTPUT_DIR/tests-report.md
---

## Implementation Files
- [path/to/file.ext] — [what it implements]

## Test Run Result
[pass/fail counts and test runner output]

## Failing Tests (if any)
[list with reason, or "None — all tests passing"]

## Implementation Notes
[key decisions, assumptions, or trade-offs made]
```

---

## Step 4 — Summary

Report to the user:

```
## Code Complete

**Feature:** <branch/feature name>
**Output directory:** `<OUTPUT_DIR>/`

**Artifacts generated:**
- `code-report.md` — List of files written and test run results

**Test status:** <X passing / Y failing>

**Next steps available:**
- `/dev-workflow-review` — Review and polish the implementation for quality and standards
```

---

## Notes

- Implementation files are written to the **project source tree**, not inside `workflow-output/`.
- `code-report.md` is the audit trail linking implementation files back to the test scenarios.
- The agent will not modify test files — if a test seems wrong, it flags it in the report rather than changing it.
- If any tests remain failing after the implementation pass, they are listed in `code-report.md` for review before moving to `/dev-workflow-review`.
