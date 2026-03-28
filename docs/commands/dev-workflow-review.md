# /dev-workflow-review

Reviews and polishes the implementation against the project's coding standards. Applies corrections directly to source files, verifies no regressions, issues a formal verdict (`approved` / `needs-work`), and generates the feature's navigable `INDEX.md`.

---

## Position in pipeline

```mermaid
flowchart LR
    CODE["/dev-workflow-code"]
    REV["**▶ /dev-workflow-review**"]:::active
    PR["/dev-workflow-pr"]

    CODE -->|"code-report.md\n+ implementation"| REV
    REV -->|"review-report.md\nINDEX.md"| PR

    classDef active fill:#2d6a4f,color:#fff,stroke:#2d6a4f
```

---

## Usage

```
/dev-workflow-review
```

No arguments required. All inputs are read from `workflow-output/<feature>/` and the project source tree.

---

## What it does

```mermaid
sequenceDiagram
    participant C as Command
    participant REV as code-beautifier-reviewer
    participant WS as Workspace (src/)
    participant FS as workflow-output/

    C->>C: Resolve branch → OUTPUT_DIR
    C->>C: Validate code-report.md
    C->>C: Check for failing tests (warn if found)
    C->>REV: code-report + PRD + tests-report
    REV->>WS: Read CLAUDE.md, linting configs, existing code
    REV->>WS: Audit implementation files
    REV->>WS: Apply corrections directly
    REV->>WS: Run test suite (verify no regressions)
    REV-->>C: Review report with verdict
    C->>FS: writes review-report.md
    C->>FS: writes INDEX.md
```

1. **Resolves the output directory** from the current git branch
2. **Guards against unresolved failures** — warns if `code-report.md` shows failing tests before proceeding
3. **Invokes `code-beautifier-reviewer`** — reads CLAUDE.md and existing code to internalize project standards, audits every implementation file, applies corrections, runs the suite to confirm no regressions
4. **Never touches test files** — issues in tests are flagged in the report, not silently changed
5. **Issues a formal verdict** — `approved` (no blocking issues) or `needs-work` (flagged issues require attention)
6. **Writes `review-report.md`** with changes made and verdict
7. **Writes `INDEX.md`** — navigable table of contents for all feature artifacts

---

## Review dimensions

```mermaid
mindmap
  root((Code Review))
    Naming
      Variables
      Functions
      Files
      Classes
    Structure
      Logical grouping
      Nesting depth
      Module boundaries
    Standards
      CLAUDE.md rules
      Linting config
      Existing patterns
    Abstractions
      DRY
      SOLID
      Leaky abstractions
    Error Handling
      Consistency
      Edge cases
    Type Safety
      Correct usage
      Interface adherence
    Performance
      Obvious footguns
```

---

## Agents invoked

| Agent | Role |
|-------|------|
| `code-beautifier-reviewer` | Senior code quality agent. Reads project standards, audits and corrects implementation files, verifies tests still pass, issues a formal verdict. |

---

## Inputs

| File | Required | Purpose |
|------|----------|---------|
| `OUTPUT_DIR/code-report.md` | **Yes** | List of implementation files to review |
| `OUTPUT_DIR/tests-report.md` | No | Test file list (to avoid reviewing test files) |
| `OUTPUT_DIR/prd-review.md` | No | Acceptance criteria to cross-check against |

---

## Outputs

```mermaid
graph LR
    CMD["/dev-workflow-review"]
    PATCHED["Patched source files\n──────────────\nNaming fixed\nStructure improved\nStandards applied"]
    RPT["review-report.md\n──────────────\nQuality assessment\nChanges per category\nTest run result\nFlagged issues\nstatus: approved / needs-work\nVerdict"]
    IDX["INDEX.md\n──────────────\nFeature overview\nArtifacts table\nAcceptance criteria\nNext steps"]

    CMD --> PATCHED
    CMD --> RPT
    CMD --> IDX
```

| Artifact | Path | Description |
|----------|------|-------------|
| Patched source files | Project source tree | Implementation corrected to match project standards |
| `review-report.md` | `workflow-output/<feature>/review-report.md` | Changes made, test result, flagged issues, verdict |
| `INDEX.md` | `workflow-output/<feature>/INDEX.md` | Navigable index of all feature artifacts |

### Verdict rules

| Verdict | Condition |
|---------|-----------|
| `approved` | No flagged issues; all tests passing |
| `needs-work` | One or more flagged issues require attention before merging |

---

## Navigation

| | |
|--|--|
| **← Previous** | [/dev-workflow-code](dev-workflow-code.md) |
| **Next →** | [/dev-workflow-pr](dev-workflow-pr.md) |
| **Status** | [/dev-workflow-status](dev-workflow-status.md) |
| **Home** | [README](../../README.md) |
