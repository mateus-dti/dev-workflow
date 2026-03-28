# Development Workflow — Tests

Generate TDD unit and integration tests from the bench test suite produced by `dev-workflow-init`. Delegates to the `tdd-test-architect` agent, which translates every scenario in `bench-test.md` into production-grade, failing test files ready for the red-green-refactor cycle.

## Usage

```
/dev-workflow-tests
```

No arguments required. All inputs are read from the feature's `workflow-output/` folder.

---

## Step 0 — Resolve Output Directory

Follow the **Branch Resolution** procedure defined in `.claude/rules/WORKFLOW_CONVENTIONS.md`.

---

## Step 1 — Validate Prerequisites

Using the Read tool, check that both required inputs exist:

| File | Required | Purpose |
|------|----------|---------|
| `OUTPUT_DIR/bench-test.md` | **Yes** | Test scenarios to translate into code |
| `OUTPUT_DIR/prd-review.md` | No | Additional context for edge cases |
| `OUTPUT_DIR/boilerplate-report.md` | No | Stack/framework detection hint |

- If `bench-test.md` **does not exist** → stop and inform the user:
  *"`OUTPUT_DIR/bench-test.md` not found. Run `/dev-workflow-init` first to generate the bench test suite."*

- If `boilerplate-report.md` **exists** → read it and extract the stack (language, test framework, package manager). Set `STACK_SOURCE = boilerplate-report`.

- If `boilerplate-report.md` **does not exist** (existing project, boilerplate step was skipped) → detect the stack from the project files directly:
  1. Look for `package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `*.csproj` to identify the language and package manager.
  2. Look for test config files (`jest.config.*`, `vitest.config.*`, `pytest.ini`, `setup.cfg`, `build.gradle`, etc.) to identify the test framework.
  3. Look for existing test files to infer naming conventions and file placement patterns.
  4. Set `STACK_SOURCE = project-detection` and summarize what was found before proceeding.
  5. If the stack genuinely cannot be determined from project files, ask the user: *"Could not detect the project stack. What language and test framework are you using?"*

---

## Step 2 — Generate Tests (`tdd-test-architect`)

Use the **Agent tool** to invoke the `tdd-test-architect` subagent.

**Prompt to pass** (replace placeholders with resolved values):

```
Generate TDD unit and integration tests for the following feature.

<feature_branch>BRANCH_NAME</feature_branch>

<bench_test_path>OUTPUT_DIR/bench-test.md</bench_test_path>

<prd_context>
PRD_CONTENT_HERE (or "Not available.")
</prd_context>

<stack_context source="STACK_SOURCE">
STACK_CONTENT_HERE
</stack_context>
<!-- If STACK_SOURCE = boilerplate-report: paste boilerplate-report.md contents -->
<!-- If STACK_SOURCE = project-detection: paste the stack summary detected from project files -->

Follow your full operational workflow:

1. Discovery: read bench-test.md at the path above; detect language, framework, and project structure from the workspace.
2. Analysis: extract every testable scenario, map each to unit or integration test categories, list fixtures and mocks needed.
3. Generation: create all test files in the appropriate locations following the project's naming conventions. Tests must be written in TDD style — they must FAIL before any implementation exists.
4. Verification: confirm each test correctly captures the scenario intent and will fail pre-implementation.

At the end, produce a structured test report with:
- Scenario coverage map (bench-test scenario → test file + test name)
- List of all files created
- Mocks and fixtures created
- How to run the tests
- Any scenarios that were ambiguous or could not be tested as written

Return this report as your final output — it will be saved to the feature's workflow-output folder.
```

Wait for the agent to complete and capture its final test report output.

---

## Step 3 — Save Test Report

Write the report returned by `tdd-test-architect` to `OUTPUT_DIR/tests-report.md` using the Write tool.

The file must follow this structure:

```
---
# Tests Report — [Feature Title]
generated_at: [ISO date]
branch: [branch name]
source: OUTPUT_DIR/bench-test.md
---

## Scenario Coverage Map
| Bench Test Scenario | Test File | Test Name |
|---------------------|-----------|-----------|
| ...                 | ...       | ...       |

## Files Created
- [path/to/test-file.ext] — [brief description]

## Mocks & Fixtures
- [what was mocked and why]

## How to Run
[command to execute the test suite]

## Ambiguous or Untestable Scenarios
[list, or "None"]
```

---

## Step 4 — Summary

Report to the user:

```
## Tests Generated

**Feature:** <branch/feature name>
**Output directory:** `<OUTPUT_DIR>/`

**Artifacts generated:**
- `tests-report.md` — Coverage map and list of all test files created

**Status:** All tests written in TDD style — they will FAIL until implementation is complete.

**Next steps available:**
- `/dev-workflow-code`   — Write implementation code to make the tests pass
- `/dev-workflow-review` — Review generated code for quality and standards
```

---

## Notes

- Tests are placed in the **project source tree** (e.g., `src/`, `tests/`), not inside `workflow-output/`.
- `tests-report.md` is the audit trail: it maps every bench-test scenario to a concrete test so nothing gets lost.
- If the stack cannot be detected from `boilerplate-report.md` or project files, `tdd-test-architect` will ask before generating.
- Integration tests are generated automatically when a workspace/monorepo structure is detected.
