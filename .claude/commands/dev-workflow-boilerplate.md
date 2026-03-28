# Development Workflow — Boilerplate

Scaffold a TDD-ready project structure for the current feature branch. Uses the validated PRD (from `dev-workflow-init`) as context to infer the stack, and delegates to the `project-initializer` agent to create the project.

## Usage

```
/dev-workflow-boilerplate [optional stack hints]
```

`$ARGUMENTS` is optional. Examples:
- `/dev-workflow-boilerplate` — infers stack from the PRD
- `/dev-workflow-boilerplate Node.js REST API with Express`
- `/dev-workflow-boilerplate Python CLI with FastAPI`

---

## Step 0 — Resolve Output Directory

Follow the **Branch Resolution** procedure defined in `.claude/rules/WORKFLOW_CONVENTIONS.md`.

---

## Step 1 — Load PRD Context

Check if `OUTPUT_DIR/prd-review.md` exists using the Read tool.

- **If it exists:** Read its contents. This will be passed as context to the `project-initializer` agent so it can infer the best stack, project type, and constraints from the validated requirements.
- **If it does not exist:** Proceed without it. The agent will rely on `$ARGUMENTS` or ask clarifying questions directly.

---

## Step 2 — Scaffold the Project (`project-initializer`)

Use the **Agent tool** to invoke the `project-initializer` subagent.

**Prompt to pass** (replace placeholders with resolved values):

```
You are scaffolding a new project for the following feature.

<feature_branch>BRANCH_NAME</feature_branch>

<prd_context>
PRD_CONTENT_HERE (or "No PRD available — use stack hints and ask clarifying questions as needed.")
</prd_context>

<stack_hints>
ARGUMENTS_HERE (or "None provided — infer from the PRD or ask.")
</stack_hints>

Follow your full initialization process:
1. Clarify language, project type, framework, testing framework, and package manager — if these can be inferred from the PRD or stack hints, proceed without asking; only ask when genuinely ambiguous.
2. Scaffold the full project structure in the current working directory.
3. Configure the testing framework.
4. Create a seed test file.
5. Install all dependencies.
6. Run the test suite to verify everything works.

At the end, produce a structured scaffold report with the following sections:
- Stack (language, framework, test framework, package manager)
- Project structure (directory tree)
- Key commands (test, dev, build)
- Test run result (pass/fail + output)
- Next steps

Return this report as your final output — it will be saved to the feature's workflow-output folder.
```

Wait for the agent to complete and capture its final scaffold report output.

---

## Step 3 — Save Scaffold Report

Write the scaffold report returned by `project-initializer` to `OUTPUT_DIR/boilerplate-report.md` using the Write tool.

The file must follow this structure:

```
---
# Boilerplate Report — [Feature Title]
generated_at: [ISO date]
branch: [branch name]
---

## Stack
- Language: ...
- Framework: ...
- Test framework: ...
- Package manager: ...

## Project Structure
[directory tree]

## Key Commands
- Run tests: ...
- Start dev server: ...
- Build: ...

## Test Run Result
[output from the initial test run]

## Next Steps
[suggestions from the agent]
```

---

## Step 4 — Summary

Report to the user:

```
## Boilerplate Complete

**Feature:** <branch/feature name>
**Output directory:** `<OUTPUT_DIR>/`

**Artifacts generated:**
- `boilerplate-report.md` — Scaffold summary (stack, structure, commands)

**Project scaffolded at:** project root

**Next steps available:**
- `/dev-workflow-tests` — Generate TDD unit and integration tests from bench-test.md
- `/dev-workflow-code`  — Write implementation code guided by the tests
```

---

## Notes

- This command operates on the **project root** (workspace), not inside `workflow-output/`. The scaffold happens where the code lives.
- If files already exist in the workspace, `project-initializer` will refuse and inform you — this command is for greenfield projects only.
- The `boilerplate-report.md` serves as a permanent record of the stack decision and initial setup for this feature.
- If `prd-review.md` is present, the stack inference will be guided by the requirements — no need to repeat yourself.
