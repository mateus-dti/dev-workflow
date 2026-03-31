# Development Workflow — UI Prototype

Generate visual UI prototypes for frontend features using the Stitch MCP Server. This command is **optional** in the pipeline — it applies only when the feature includes visual or UI changes. Run after `dev-workflow-init` and before `dev-workflow-tests` to validate the visual design before writing tests and code.

## Usage

```
/dev-workflow-prototype
```

No arguments required. Feature context is read from `workflow-output/` artifacts.

---

## Step 0 — Resolve Output Directory

Follow the **Branch Resolution** procedure defined in `.claude/rules/WORKFLOW_CONVENTIONS.md`.

All subsequent file paths use `OUTPUT_DIR` as the base.

---

## Step 1 — Session Resume Check

Follow the **Session Resume Protocol** defined in `.claude/rules/WORKFLOW_CONVENTIONS.md`.

Check for `{OUTPUT_DIR}/session-resume.md` before proceeding.

---

## Step 2 — Validate Prerequisites

Using the Read tool, check that required inputs exist:

| File | Required | Purpose |
|------|----------|---------|
| `OUTPUT_DIR/prd-review.md` | **Yes** | Feature description and acceptance criteria for prototype scope |

- If `prd-review.md` **does not exist** → stop immediately:
  *"`OUTPUT_DIR/prd-review.md` not found. Run `/dev-workflow-init` first."*

Read the file. Scan the content for visual/frontend signals: UI, UX, screen, page, component, layout, design, visual, interface, frontend, dashboard, form, modal, flow, wizard, button, style, theme, responsive, display, view.

If **no visual signal** is found, warn the user:
> "This feature does not appear to involve UI or visual changes. `/dev-workflow-prototype` is only for frontend features. Continue anyway? (yes/no)"

Wait for confirmation. If the user says no, stop.

---

## Step 3 — Check Stitch MCP Availability

Attempt to call `mcp__stitch__list_projects` with no arguments.

**If the call succeeds** (even returning an empty list): Stitch is available — proceed to Step 4.

**If the call fails** (tool not found, connection error, or MCP error):

Show the user this message and stop:

```
## Stitch MCP Server Not Found

The Stitch MCP Server is not currently connected to Claude.
To enable UI prototype generation, integrate Stitch with Claude Code:

1. Add the Stitch MCP Server to your Claude configuration:
   - In Claude Code, run `/mcp` to open the MCP settings panel.
   - Add Stitch as a new MCP server with the required API credentials.
2. Restart Claude Code to activate the connection.
3. Re-run `/dev-workflow-prototype` once Stitch is connected.

Alternatively, you can skip this step and continue the workflow:
- `/dev-workflow-tests` — Generate TDD tests (visual prototypes are optional)
- `/dev-workflow-code`  — Write implementation code
```

---

## Step 4 — Generate Prototypes (`dev-workflow-prototype` agent)

Read the full content of `OUTPUT_DIR/prd-review.md`.

Use the **Agent tool** to invoke the `dev-workflow-prototype` subagent.

**Prompt to pass** (replace placeholders with resolved values):

```
Generate UI prototypes for this feature using the Stitch MCP Server. Stitch has already been confirmed available.

<feature_name>FEATURE_NAME</feature_name>
<output_dir>OUTPUT_DIR</output_dir>
<branch>BRANCH_NAME</branch>

<prd_content>
PRD_CONTENT_HERE
</prd_content>

Skip Steps 0, 1, 2, and 3 from your workflow — branch resolution and guard clauses have already been handled by the orchestrator. Stitch MCP is confirmed available.

Start directly at Step 4 (Prepare the Prototype Request).

Your task:
1. Analyze the PRD to identify all screens and UI states to prototype.
2. Build a detailed, specific Stitch prompt for each screen/component.
3. Create a Stitch project for this feature (or reuse one matching the feature name).
4. Generate each screen via Stitch. Capture all returned artifacts.
5. Save all artifacts to `OUTPUT_DIR/prototypes/` with descriptive filenames.
6. Evaluate each screen against the PRD acceptance criteria.
7. If a screen fails evaluation, request edits from Stitch (up to 3 revision cycles).
8. Write `OUTPUT_DIR/prototype-report.md` with the full evaluation and verdict.

Return a summary of: screens generated, revisions made, final verdict, and path to the report.
```

Wait for the agent to complete.

---

## Step 5 — Summary

After the agent completes, read `OUTPUT_DIR/prototype-report.md` and report to the user:

```
## Prototype Generation Complete

**Feature:** <branch/feature name>
**Output directory:** `<OUTPUT_DIR>/`

**Artifacts generated:**
- `prototypes/`          — Generated screen designs and assets
- `prototype-report.md`  — Evaluation log and final verdict

**Screens generated:** <count from report>
**Revisions made:** <count from report>

**Verdict:** <APPROVED / APPROVED WITH MINOR ISSUES / INSUFFICIENT>

**Next steps available:**
- `/dev-workflow-tests`  — Generate TDD tests (prototypes can inform test scenarios)
- `/dev-workflow-code`   — Write implementation code
- `/dev-workflow-review` — Review generated code for quality and standards
```

If the verdict is **INSUFFICIENT**: add this note:
> "After 3 revision cycles, some screens still have gaps. Review `prototype-report.md` for the detailed gap analysis before proceeding."

---

## Notes

- Prototype artifacts are saved inside `workflow-output/` — they are part of the feature audit trail, not the source tree.
- This command is **optional** and can be run at any point after `dev-workflow-init`.
- If Stitch is not available, the rest of the pipeline can still continue — visual prototypes are not a hard dependency for tests or code.
- Re-running this command on the same branch will overwrite existing prototype artifacts (previous versions are preserved with `-v2`, `-v3` suffixes by the agent).
