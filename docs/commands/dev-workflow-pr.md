# /dev-workflow-pr

Consolidates all workflow artifacts into a complete pull request description. Detects available MCP integrations and offers to create the PR automatically — or falls back to a manual copy-paste flow.

---

## Position in pipeline

```mermaid
flowchart LR
    REV["/dev-workflow-review"]
    PR["**▶ /dev-workflow-pr**"]:::active

    REV -->|"review-report.md\nINDEX.md"| PR
    PR -->|"pr-description.md"| DONE["PR created / ready"]

    classDef active fill:#2d6a4f,color:#fff,stroke:#2d6a4f
```

---

## Usage

```
/dev-workflow-pr
```

No arguments required. All inputs are read from `workflow-output/<feature>/`.

---

## What it does

```mermaid
flowchart TD
    A["Resolve OUTPUT_DIR"]
    B["Validate prerequisites\nprd-review.md + review-report.md"]
    C{"review status\n= needs-work?"}
    D["Warn user\nProceed anyway?"]
    E["Compose PR description\nfrom all artifacts"]
    F["Save pr-description.md"]
    G["Print PR description"]
    H["Detect MCP tools\nfor PR creation"]
    I{"MCP tool\nfound?"}
    J["Ask: Automatic via MCP\nor Manual?"]
    K["Manual flow\nCopy-paste instructions"]
    L{"User chose\nMCP?"}
    M["Ask for target branch"]
    N["Call MCP tool"]
    O{"MCP call\nsucceeded?"}
    P["Show PR URL"]
    Q["Fallback to manual\nshow error"]

    A --> B --> C
    C -->|Yes| D --> E
    C -->|No| E
    E --> F --> G --> H --> I
    I -->|No| K
    I -->|Yes| J --> L
    L -->|No| K
    L -->|Yes| M --> N --> O
    O -->|Yes| P
    O -->|No| Q
```

---

## PR description structure

The description is composed directly from the artifacts — no agent is invoked.

```mermaid
graph LR
    PRD["prd-review.md"]
    CODE["code-report.md"]
    TEST["tests-report.md"]
    REV["review-report.md"]

    PRD -->|"Objective\nAcceptance criteria\nOut of scope"| COMP["pr-description.md"]
    CODE -->|"Files changed\nKey decisions"| COMP
    TEST -->|"Test plan checklist\nHow to run"| COMP
    REV -->|"Verdict\nFlagged issues"| COMP
```

| Section | Source | Description |
|---------|--------|-------------|
| Title | `prd-review.md` | `[type]: <feature description>`, max 72 chars |
| What / Why | `prd-review.md` | Objective and business motivation |
| In Scope | `prd-review.md` | Acceptance criteria as bullet list |
| Out of Scope | `prd-review.md` | Explicit exclusions |
| Files Changed | `code-report.md` | Implementation files and what they do |
| Key Decisions | `code-report.md` | Trade-offs and notable choices |
| Test Plan | `tests-report.md` | Scenarios as checkboxes, grouped by category |
| Review Notes | `review-report.md` | Verdict and any flagged issues |
| Pre-merge Checklist | — | Standard checklist |

---

## MCP integration

```mermaid
flowchart TD
    DETECT["Scan available MCP tools"]
    DETECT --> FOUND{"Pattern match:\ncreate_pull_request\ncreate_pr\ncreate_merge_request"}
    FOUND -->|"GitHub MCP"| G["github / create_pull_request"]
    FOUND -->|"Azure DevOps MCP"| A["azure_devops / create_pull_request"]
    FOUND -->|"GitLab MCP"| GL["gitlab / create_merge_request"]
    FOUND -->|None| MANUAL["Manual mode"]
    G & A & GL --> ASK["Ask user:\n[1] Auto via MCP\n[2] Manual"]
```

If a MCP tool is found, the command asks:
```
How would you like to create the PR?

  [1] Automatically via MCP (github / create_pull_request)
  [2] Manually — I'll paste the description myself
```

---

## Agents invoked

None. The PR description is synthesized directly from the artifacts without invoking an agent.

---

## Inputs

| File | Required | Purpose |
|------|----------|---------|
| `OUTPUT_DIR/prd-review.md` | **Yes** | Objective, acceptance criteria, scope |
| `OUTPUT_DIR/review-report.md` | **Yes** | Verdict and flagged issues |
| `OUTPUT_DIR/tests-report.md` | No | Test plan for checklist |
| `OUTPUT_DIR/code-report.md` | No | Files changed and implementation notes |
| `OUTPUT_DIR/INDEX.md` | No | Feature overview reference |

---

## Outputs

| Artifact | Path | Description |
|----------|------|-------------|
| `pr-description.md` | `workflow-output/<feature>/pr-description.md` | Complete PR description ready to use |
| PR (optional) | Remote repository | Created via MCP if the user chooses option 1 |

---

## Navigation

| | |
|--|--|
| **← Previous** | [/dev-workflow-review](dev-workflow-review.md) |
| **Status** | [/dev-workflow-status](dev-workflow-status.md) |
| **Home** | [README](../../README.md) |
