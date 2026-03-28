# /dev-workflow-boilerplate

Scaffolds a complete TDD-ready project from scratch. Optional — only needed for greenfield projects. For existing codebases, skip directly to [/dev-workflow-tests](dev-workflow-tests.md).

---

## Position in pipeline

```mermaid
flowchart LR
    INIT["/dev-workflow-init"]
    BOIL["**▶ /dev-workflow-boilerplate**"]:::active
    TEST["/dev-workflow-tests"]

    INIT -->|"prd-review.md\nbench-test.md"| BOIL
    BOIL -->|"boilerplate-report.md"| TEST

    classDef active fill:#2d6a4f,color:#fff,stroke:#2d6a4f
```

---

## Usage

```
/dev-workflow-boilerplate [optional stack hints]
```

**Examples:**
```
/dev-workflow-boilerplate

/dev-workflow-boilerplate Node.js REST API with Express

/dev-workflow-boilerplate Python CLI with pytest
```

If no hints are provided and `prd-review.md` exists, the stack is inferred from the requirements automatically.

---

## What it does

```mermaid
sequenceDiagram
    participant C as Command
    participant SCAFF as project-initializer
    participant WS as Workspace (project root)
    participant FS as workflow-output/

    C->>C: Resolve branch → OUTPUT_DIR
    C->>C: Read prd-review.md (if exists)
    C->>SCAFF: PRD context + stack hints
    SCAFF->>SCAFF: Clarify stack if ambiguous
    SCAFF->>WS: Scaffold src/, tests/, configs
    SCAFF->>WS: Install dependencies
    SCAFF->>WS: Run test suite (verify setup)
    SCAFF-->>C: Scaffold report
    C->>FS: writes boilerplate-report.md
```

1. **Resolves the output directory** from the current git branch
2. **Reads `prd-review.md`** if it exists — uses it to infer the appropriate stack without asking
3. **Invokes `project-initializer`** — clarifies language, framework, test framework, and package manager if not inferable; then scaffolds the full project
4. **Verifies the setup** — installs dependencies and runs the test suite to confirm everything works out of the box
5. **Writes `boilerplate-report.md`** — documents the stack chosen, directory structure, and key commands

---

## Agents invoked

| Agent | Role |
|-------|------|
| `project-initializer` | Scaffolds the project structure, configures the test framework, creates a seed test, installs dependencies, and verifies the setup. |

---

## Inputs

| File | Required | Purpose |
|------|----------|---------|
| `OUTPUT_DIR/prd-review.md` | No | Stack inference from requirements |
| `$ARGUMENTS` | No | Explicit stack hints override inference |

> **Constraint:** `project-initializer` only operates on empty workspaces. If files already exist in the project root, it will stop and inform you.

---

## Outputs

```mermaid
graph LR
    CMD["/dev-workflow-boilerplate"]
    PROJ["Project root\n──────────────\nsrc/\ntests/\npackage.json etc.\n.gitignore\nREADME.md\nSeed test (passing)"]
    RPT["boilerplate-report.md\n──────────────\nStack\nDirectory tree\nKey commands\nTest run result\nNext steps"]

    CMD --> PROJ
    CMD --> RPT
```

| Artifact | Path | Description |
|----------|------|-------------|
| Project files | Project root | Full TDD-ready scaffold with passing seed test |
| `boilerplate-report.md` | `workflow-output/<feature>/boilerplate-report.md` | Stack reference used by subsequent commands |

---

## Navigation

| | |
|--|--|
| **← Previous** | [/dev-workflow-init](dev-workflow-init.md) |
| **Next →** | [/dev-workflow-tests](dev-workflow-tests.md) |
| **Status** | [/dev-workflow-status](dev-workflow-status.md) |
| **Home** | [README](../../README.md) |
