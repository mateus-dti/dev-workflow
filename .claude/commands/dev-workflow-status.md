# Development Workflow — Status

Show the current state of the workflow for the active feature branch. Checks which artifacts exist in `workflow-output/<feature>/` and displays what has been completed, what is in progress, and what comes next.

## Usage

```
/dev-workflow-status
```

No arguments required.

---

## Step 0 — Resolve Output Directory

Follow the **Branch Resolution** procedure defined in `.claude/rules/WORKFLOW_CONVENTIONS.md`.

---

## Step 1 — Check Artifacts

Using the Read tool, probe the existence and basic validity of each artifact in `OUTPUT_DIR`:

| Step | File | Command that generates it |
|------|------|--------------------------|
| 1 | `prd-review.md` | `/dev-workflow-init` |
| 2 | `bench-test.md` | `/dev-workflow-init` |
| 3 | `boilerplate-report.md` | `/dev-workflow-boilerplate` (optional) |
| 4 | `tests-report.md` | `/dev-workflow-tests` |
| 5 | `code-report.md` | `/dev-workflow-code` |
| 6 | `review-report.md` | `/dev-workflow-review` |

For each file:
- **Exists** → mark as `done`. Also read the `status:` field from the frontmatter if present.
- **Does not exist** → mark as `pending`.

---

## Step 2 — Determine Current Position

After checking all files, determine:

- **Last completed step**: the highest-numbered step whose file exists.
- **Next step**: the first pending file in sequence — this is what the user should run next.
- **Blockers**: any file that is marked `needs-work` or `Blocked` in its frontmatter `status:` field.

---

## Step 3 — Report to User

Print the status board using this format:

```
## Workflow Status — <feature-name>
Output directory: `<OUTPUT_DIR>/`

STEP  ARTIFACT                COMMAND                    STATUS
────  ──────────────────────  ─────────────────────────  ──────
 1    prd-review.md           /dev-workflow-init         ✅ done
 2    bench-test.md           /dev-workflow-init         ✅ done
 3    boilerplate-report.md   /dev-workflow-boilerplate  ⏭ skipped (optional)
 4    tests-report.md         /dev-workflow-tests        ✅ done
 5    code-report.md          /dev-workflow-code         🔄 next
 6    review-report.md        /dev-workflow-review       ⬜ pending

▶ Next: run `/dev-workflow-code` to write the implementation.
```

Use the following status icons:

| Icon | Meaning |
|------|---------|
| ✅ done | File exists, no blockers in frontmatter |
| ⚠️ needs-work | File exists but frontmatter `status` is `needs-work` or `Blocked` |
| 🔄 next | First pending step — what to run now |
| ⬜ pending | Not yet generated, not the immediate next step |
| ⏭ skipped | Optional step with no file (boilerplate-report only) |

Rules for the footer line:
- If all 6 steps are done with no `needs-work` → print: *"✅ Workflow complete. Feature is ready for PR."*
- If any step has `needs-work` or `Blocked` → list them: *"⚠️ Attention required in: `<file>` — <status value from frontmatter>."*
- Otherwise → print the next command to run.

---

## Notes

- `boilerplate-report.md` is the only optional step. If absent, it is shown as `⏭ skipped` rather than `🔄 next`.
- This command is read-only — it never modifies files or invokes agents.
- Run it any time to check where you are in the pipeline without opening files manually.
