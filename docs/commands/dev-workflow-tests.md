# /dev-workflow-tests

Translates the bench test scenarios from `bench-test.md` into real, failing test files placed in the project source tree. All tests are written in TDD style — they must fail before implementation exists.

---

## Position in pipeline

```mermaid
flowchart LR
    BOIL["/dev-workflow-boilerplate\n(optional)"]
    TEST["**▶ /dev-workflow-tests**"]:::active
    CODE["/dev-workflow-code"]

    BOIL -->|"boilerplate-report.md"| TEST
    TEST -->|"tests-report.md\n+ test files"| CODE

    classDef active fill:#2d6a4f,color:#fff,stroke:#2d6a4f
```

---

## Usage

```
/dev-workflow-tests
```

No arguments required. All inputs are read from `workflow-output/<feature>/`.

---

## What it does

```mermaid
sequenceDiagram
    participant C as Command
    participant TDD as tdd-test-architect
    participant WS as Workspace (src/tests/)
    participant FS as workflow-output/

    C->>C: Resolve branch → OUTPUT_DIR
    C->>C: Validate bench-test.md exists
    C->>C: Detect stack (boilerplate-report OR project files)
    C->>TDD: bench-test.md + stack + PRD context
    TDD->>TDD: Discovery — detect framework & structure
    TDD->>TDD: Analysis — map scenarios to test categories
    TDD->>WS: Generate unit test files (failing)
    TDD->>WS: Generate integration tests (if monorepo)
    TDD->>TDD: Verification — confirm tests fail pre-implementation
    TDD-->>C: Test report
    C->>FS: writes tests-report.md
```

1. **Resolves the output directory** from the current git branch
2. **Detects the stack** — from `boilerplate-report.md` if it exists, or by scanning project files (`package.json`, `pyproject.toml`, jest/pytest configs, existing test files)
3. **Invokes `tdd-test-architect`** — maps every scenario in `bench-test.md` to unit or integration test categories, generates all test files, verifies they fail before implementation
4. **Writes `tests-report.md`** — maps each bench test scenario to its generated test file and test name

---

## Stack detection

```mermaid
flowchart TD
    A{boilerplate-report.md\nexists?}
    A -->|Yes| B["Use stack from\nboilerplate-report.md"]
    A -->|No| C["Scan project files\npackage.json, pyproject.toml\njest.config.*, pytest.ini\nexisting test files..."]
    C --> D{Stack\ndetected?}
    D -->|Yes| E["Proceed with\ndetected stack"]
    D -->|No| F["Ask user:\nWhat language and\ntest framework?"]
```

---

## Agents invoked

| Agent | Role |
|-------|------|
| `tdd-test-architect` | Parses `bench-test.md`, detects the testing ecosystem, generates unit and integration test files following TDD (failing before implementation). |

---

## Inputs

| File | Required | Purpose |
|------|----------|---------|
| `OUTPUT_DIR/bench-test.md` | **Yes** | Scenarios to translate into test code |
| `OUTPUT_DIR/prd-review.md` | No | Extra context for edge cases |
| `OUTPUT_DIR/boilerplate-report.md` | No | Stack reference (falls back to project detection) |

---

## Outputs

```mermaid
graph LR
    CMD["/dev-workflow-tests"]
    FILES["Test files\n──────────────\nUnit tests\nIntegration tests\nMocks & fixtures\n(in src/ or tests/)"]
    RPT["tests-report.md\n──────────────\nScenario coverage map\nFiles created\nMocks & fixtures\nHow to run tests\nAmbiguous scenarios"]

    CMD --> FILES
    CMD --> RPT
```

| Artifact | Path | Description |
|----------|------|-------------|
| Test files | Project source tree | Failing unit and integration tests, one per scenario group |
| `tests-report.md` | `workflow-output/<feature>/tests-report.md` | Coverage map: bench-test scenario → test file + test name |

---

## Navigation

| | |
|--|--|
| **← Previous** | [/dev-workflow-boilerplate](dev-workflow-boilerplate.md) — or [/dev-workflow-init](dev-workflow-init.md) if boilerplate was skipped |
| **Next →** | [/dev-workflow-code](dev-workflow-code.md) |
| **Status** | [/dev-workflow-status](dev-workflow-status.md) |
| **Home** | [README](../../README.md) |
