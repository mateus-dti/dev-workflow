# Workflow

A development workflow built on top of Claude Code's agent system.

---

## Installation

The workflow is installed per-project by copying the `.claude/` folder into the root of your repository.

### 1. Clone this repository

```bash
git clone <repo-url> workflow
```

### 2. Copy `.claude/` into your project

```bash
cp -r workflow/.claude /path/to/your-project/
```

On Windows (PowerShell):
```powershell
Copy-Item -Recurse workflow\.claude your-project\
```

### 3. Verify the structure

After copying, your project should have:

```
your-project/
  .claude/
    agents/
      bench-test-architect.md
      code-beautifier-reviewer.md
      code-writer.md
      prd-story-reviewer.md
      project-initializer.md
      tdd-test-architect.md
    commands/
      dev-workflow-boilerplate.md
      dev-workflow-code.md
      dev-workflow-init.md
      dev-workflow-pr.md
      dev-workflow-review.md
      dev-workflow-status.md
      dev-workflow-tests.md
    rules/
      WORKFLOW_CONVENTIONS.md
```

### 4. Open your project in Claude Code

```bash
cd your-project
claude
```

The `/dev-workflow-*` commands will be available immediately.

> **Note:** `workflow-output/` is created automatically the first time you run a command. Add it to your `.gitignore` if you don't want to commit the generated artifacts.

--- Each feature goes through a structured pipeline — from requirements review to pull request — with every decision documented in a per-feature folder that serves as the audit trail for the entire development cycle.

---

## Pipeline

```mermaid
flowchart TD
    INIT["/dev-workflow-init\n─────────────────\nprd-story-reviewer\nbench-test-architect"]
    BOIL["/dev-workflow-boilerplate\n─────────────────\nproject-initializer"]
    TEST["/dev-workflow-tests\n─────────────────\ntdd-test-architect"]
    CODE["/dev-workflow-code\n─────────────────\ncode-writer"]
    REV["/dev-workflow-review\n─────────────────\ncode-beautifier-reviewer"]
    PR["/dev-workflow-pr\n─────────────────\n(synthesis — no agent)"]
    STATUS["/dev-workflow-status\n─────────────────\n(read-only)"]

    INIT -->|prd-review.md\nbench-test.md| BOIL
    BOIL -->|boilerplate-report.md\n— optional —| TEST
    INIT -.->|skip boilerplate\nfor existing projects| TEST
    TEST -->|tests-report.md| CODE
    CODE -->|code-report.md| REV
    REV -->|review-report.md\nINDEX.md| PR

    STATUS -. "check anytime" .-> INIT
    STATUS -. "check anytime" .-> TEST
    STATUS -. "check anytime" .-> CODE
```

---

## Output structure

Each feature produces an isolated folder derived from the git branch name.

```mermaid
graph TD
    ROOT["workflow-output/"]
    F1["feature-auth-login/"]
    F2["feature-payment-gateway/"]
    F3["fix-cart-overflow/"]

    ROOT --> F1
    ROOT --> F2
    ROOT --> F3

    F1 --> A1["INDEX.md"]
    F1 --> A2["prd-review.md"]
    F1 --> A3["bench-test.md"]
    F1 --> A4["boilerplate-report.md ¹"]
    F1 --> A5["tests-report.md"]
    F1 --> A6["code-report.md"]
    F1 --> A7["review-report.md"]
    F1 --> A8["pr-description.md ²"]
```

> ¹ Only present if `/dev-workflow-boilerplate` was run (greenfield projects).
> ² Only present if `/dev-workflow-pr` was run.

---

## Agents

Each command delegates work to one or more specialized agents defined in `.claude/agents/`.

```mermaid
flowchart LR
    subgraph "Requirements"
        PRD["prd-story-reviewer\n───────────────\nValidates PRDs and\nuser stories"]
        BENCH["bench-test-architect\n───────────────\nDesigns test scenarios\nfrom requirements"]
    end

    subgraph "Setup"
        SCAFF["project-initializer\n───────────────\nScaffolds TDD-ready\nprojects from scratch"]
    end

    subgraph "Implementation"
        TDD["tdd-test-architect\n───────────────\nGenerates failing unit\nand integration tests"]
        WRITE["code-writer\n───────────────\nWrites implementation\ncode to pass tests"]
    end

    subgraph "Quality"
        REVIEW["code-beautifier-reviewer\n───────────────\nReviews and polishes\ncode to meet standards"]
    end

    PRD --> BENCH
    BENCH --> TDD
    SCAFF -.-> TDD
    TDD --> WRITE
    WRITE --> REVIEW
```

---

## Commands

| Command | Description | Docs |
|---------|-------------|------|
| `/dev-workflow-init` | Review requirements and generate bench tests | [→](docs/commands/dev-workflow-init.md) |
| `/dev-workflow-boilerplate` | Scaffold a TDD-ready project | [→](docs/commands/dev-workflow-boilerplate.md) |
| `/dev-workflow-tests` | Generate failing TDD test files | [→](docs/commands/dev-workflow-tests.md) |
| `/dev-workflow-code` | Write implementation to pass tests | [→](docs/commands/dev-workflow-code.md) |
| `/dev-workflow-review` | Review and polish the implementation | [→](docs/commands/dev-workflow-review.md) |
| `/dev-workflow-pr` | Generate and optionally create the PR | [→](docs/commands/dev-workflow-pr.md) |
| `/dev-workflow-status` | Check pipeline progress at any time | [→](docs/commands/dev-workflow-status.md) |

---

## Configuration

Shared conventions — branch resolution logic, output directory structure, artifact header format — are defined in `.claude/rules/WORKFLOW_CONVENTIONS.md`. Edit that file to change behavior across all commands at once.
