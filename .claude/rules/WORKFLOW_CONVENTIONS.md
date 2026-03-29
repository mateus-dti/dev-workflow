# Workflow Conventions

Conventions shared across all `dev-workflow-*` commands. Edit this file to change behavior globally — every command that references it will pick up the change automatically.

---

## Branch Resolution (Step 0)

Every `dev-workflow-*` command starts by resolving the feature name that determines the output directory. Follow this procedure exactly:

### 1. Detect branch
Run `git branch --show-current` via the Bash tool.

### 2. Evaluate result

| Result | Action |
|--------|--------|
| Command fails or returns empty | Ask the user: *"Could not detect a git branch. What is the feature name for this workflow run?"* |
| Returns `main`, `master`, or `develop` | Ask the user: *"You're on `<branch>`. What is the feature name for this workflow run?"* |
| Any other branch name | Use it as-is |

### 3. Sanitize the name
Apply in order:
- Replace `/` with `-` (e.g. `feature/auth-login` → `feature-auth-login`)
- Replace spaces with `-`
- Lowercase everything
- Remove any character outside `[a-z0-9-_]`

### 4. Set output directory
```
OUTPUT_DIR = workflow-output/<sanitized-name>
```

### 5. Confirm to the user
> "Feature directory: `<OUTPUT_DIR>`"

---

## Output Directory Structure

All workflow artifacts for a feature are stored in `workflow-output/<feature-name>/`. No file outside this folder is ever created or modified by the workflow commands themselves — only by the agents they invoke.

```
workflow-output/
  <feature-name>/
    prd-review.md         ← dev-workflow-init
    bench-test.md         ← dev-workflow-init
    boilerplate-report.md ← dev-workflow-boilerplate (optional)
    tests-report.md       ← dev-workflow-tests
    code-report.md        ← dev-workflow-code
    review-report.md      ← dev-workflow-review
```

---

## Artifact Header Format

Every artifact written to `OUTPUT_DIR` must include this frontmatter block at the top:

```
---
# <Report Title> — <Feature Title>
generated_at: <ISO 8601 date>
branch: <branch name>
---
```

---

## Guard Clauses

Before invoking any agent, each command checks that its required input artifacts exist. If a required file is missing, the command **stops immediately** and tells the user which command to run first. It never proceeds with missing inputs.

---

## Session Resume Protocol

Every command checks for an interrupted session before invoking any agent.

### Pre-Invocation Check

After resolving `OUTPUT_DIR`, check if `{OUTPUT_DIR}/session-resume.md` exists:

- If it **exists**: read the file and show the user a summary:
  > "A previous session was interrupted at ~95% context. Agent: `<agent>`, interrupted at: `<interrupted step>`."
  > "Resume from where it left off, or start fresh? (resume/fresh)"
  - **Resume**: pass the full file contents to the agent as a `RESUME_CONTEXT` block in the task prompt
  - **Start fresh**: delete `{OUTPUT_DIR}/session-resume.md` and proceed normally
- If it **does not exist**: proceed normally

### Resume File Location

```
{OUTPUT_DIR}/session-resume.md
```

---

## Customization

| Convention | Where to change |
|------------|-----------------|
| Branch resolution rules | This file — `## Branch Resolution` section |
| Protected branch names (besides main/master/develop) | Add to the table in step 2 above |
| Output directory root (`workflow-output/`) | This file — `## Output Directory Structure` section — update all references |
| Artifact header format | This file — `## Artifact Header Format` section |
