# Development Workflow — Pull Request

Generate a complete pull request description for the current feature by consolidating all workflow artifacts. Produces a PR title, summary, test plan checklist, and links to the feature documentation — ready to paste into GitHub, Azure DevOps, or any PR tool.

## Usage

```
/dev-workflow-pr
```

No arguments required. All inputs are read from the feature's `workflow-output/` folder.

---

## Step 0 — Resolve Output Directory

Follow the **Branch Resolution** procedure defined in `.claude/rules/WORKFLOW_CONVENTIONS.md`.

---

## Step 1 — Validate Prerequisites

Using the Read tool, check that required inputs exist:

| File | Required | Purpose |
|------|----------|---------|
| `OUTPUT_DIR/prd-review.md` | **Yes** | Objective, acceptance criteria, scope |
| `OUTPUT_DIR/review-report.md` | **Yes** | Verdict and flagged issues |
| `OUTPUT_DIR/tests-report.md` | No | Test coverage summary |
| `OUTPUT_DIR/code-report.md` | No | Implementation files list |
| `OUTPUT_DIR/INDEX.md` | No | Feature overview |

- If `prd-review.md` **does not exist** → stop:
  *"`OUTPUT_DIR/prd-review.md` not found. Run `/dev-workflow-init` first."*

- If `review-report.md` **does not exist** → stop:
  *"`OUTPUT_DIR/review-report.md` not found. Run `/dev-workflow-review` first."*

- If `review-report.md` has `status: needs-work` → warn before proceeding:
  *"Warning: the review verdict is NEEDS WORK. There are flagged issues that should be addressed before opening a PR. Proceed anyway? (yes/no)"*

Read all available files before composing the PR description.

---

## Step 2 — Compose PR Description

Using the content read from the artifacts, compose the PR description directly (no agent invocation needed — this is a synthesis task).

Build each section as follows:

### Title
- Extract the feature title from `prd-review.md`
- Format: `[type]: <concise description>` where type is one of `feat`, `fix`, `refactor`, `chore`
- Maximum 72 characters
- Example: `feat: add login with email and password with rate limiting`

### Summary (from prd-review.md)
- **What**: one sentence from the `## Objective` section
- **Why**: business motivation — infer from the objective and users affected
- **Scope**: bullet list of what is IN scope (acceptance criteria) and what is explicitly OUT of scope

### Implementation Notes (from code-report.md)
- Key decisions or trade-offs noted in `## Implementation Notes`
- List of main files changed
- Omit if code-report.md is unavailable

### Test Plan (from tests-report.md + bench-test.md)
- Convert the scenario coverage map into a markdown checklist
- Group by Happy Path / Sad Path / Edge Cases
- Include the command to run the test suite
- Omit if tests-report.md is unavailable

### Review Notes (from review-report.md)
- Verdict: APPROVED or NEEDS WORK
- Any flagged issues that reviewers should be aware of (listed as `> ⚠️ [issue]`)
- Omit flagged issues section if "None"

### Checklist
Standard pre-merge checklist:
- [ ] All tests passing
- [ ] Review verdict: APPROVED
- [ ] No flagged issues outstanding (or issues acknowledged and tracked)
- [ ] Feature documented in `workflow-output/<feature>/INDEX.md`

---

## Step 3 — Save PR Description

Write the composed PR description to `OUTPUT_DIR/pr-description.md` using the Write tool.

The file must follow this structure:

```
---
# PR Description — [Feature Title]
generated_at: [ISO date]
branch: [branch name]
status: [ready | draft]
---

## Title
[PR title]

## Summary
**What:** [objective]
**Why:** [motivation]

### In Scope
- [acceptance criterion 1]
- [acceptance criterion 2]

### Out of Scope
- [explicit exclusion]

## Implementation Notes
### Files Changed
- [path/to/file.ext] — [what it implements]

### Key Decisions
[notes from code-report, or omit section if none]

## Test Plan
### Happy Path
- [ ] [scenario name]

### Sad Path
- [ ] [scenario name]

### Edge Cases
- [ ] [scenario name]

**Run tests:** `[test command]`

## Review Notes
**Verdict:** [APPROVED / NEEDS WORK]

[> ⚠️ flagged issue — only if any exist]

## Pre-merge Checklist
- [ ] All tests passing
- [ ] Review verdict: APPROVED
- [ ] No flagged issues outstanding (or issues acknowledged and tracked)
- [ ] Feature documented in `workflow-output/<feature>/INDEX.md`
```

---

## Step 4 — Print PR Description

Print the full contents of `pr-description.md` to the user so they can copy and paste it directly into their PR tool. Format it cleanly without any surrounding commentary.

---

## Step 5 — Detect MCP PR Tools

Before asking the user how to proceed, probe the available MCP tools for any integration capable of creating a pull request. Look for tools whose name or description suggests PR creation — common patterns include:

- Tools from `github` MCP server (e.g., `create_pull_request`, `github_create_pr`)
- Tools from `azure-devops` / `azure_devops` MCP server (e.g., `create_pull_request`, `create_pr`)
- Tools from `gitlab` MCP server (e.g., `create_merge_request`)
- Any other MCP tool whose name contains `pull_request`, `create_pr`, or `merge_request`

Determine:
- **`mcp_available`**: `true` if at least one such tool is found, `false` otherwise
- **`mcp_tool`**: the name of the tool found (if any)
- **`mcp_server`**: the MCP server it belongs to (if identifiable)

---

## Step 6 — Ask User How to Proceed

Based on `mcp_available`, ask the user:

**If `mcp_available = true`:**
```
How would you like to create the PR?

  [1] Automatically via MCP (<mcp_server> / <mcp_tool>)
  [2] Manually — I'll paste the description myself

Enter 1 or 2:
```

**If `mcp_available = false`:**
```
No MCP tool for PR creation was found in this session.

The PR description has been saved to `<OUTPUT_DIR>/pr-description.md`.
Copy the content above and paste it into your PR tool manually.
```

In the `false` case, skip to Step 8.

---

## Step 7 — Create PR via MCP (if user chose option 1)

Use the MCP tool identified in Step 5 to create the pull request. Map the fields from `pr-description.md` as follows:

| PR field | Source |
|----------|--------|
| Title | `## Title` section |
| Description / Body | Full markdown body of `pr-description.md` (excluding the frontmatter and `## Title` line) |
| Source branch | Branch name resolved in Step 0 |
| Target branch | Ask the user: *"Which branch should this PR target? (e.g., main, develop)"* |

After the MCP call:
- If it **succeeds** → note the PR URL returned by the tool
- If it **fails** → inform the user of the error and fall back to manual: *"MCP call failed: `<error>`. The description is saved to `<OUTPUT_DIR>/pr-description.md` — please create the PR manually."*

---

## Step 8 — Summary

Report to the user:

```
## PR Ready

**Feature:** <branch/feature name>
**Description saved to:** `<OUTPUT_DIR>/pr-description.md`

<if created via MCP:>
**PR created:** <PR URL>

<if manual:>
**Next:** paste the description above into your PR tool.
```

---

## Notes

- The PR description is composed from the artifacts — no agent is invoked. The synthesis is done directly.
- If any artifact is missing, the corresponding section is omitted gracefully rather than blocking.
- `pr-description.md` is the final artifact in the `OUTPUT_DIR/` trail, completing the full documentation chain from PRD to PR.
- MCP tool detection is best-effort — if the tool is found but requires additional parameters (e.g., repo name, project), the tool's own schema will prompt for them during the call.
