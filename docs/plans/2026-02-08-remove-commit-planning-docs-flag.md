# Remove COMMIT_PLANNING_DOCS Flag

**Goal:** Remove the `COMMIT_PLANNING_DOCS` detection pattern and associated conditional logic from all files.

**Why:** JJ has no staging area. The working copy is always part of the current change. The `COMMIT_PLANNING_DOCS` flag was a git-ism that controlled whether planning files were `git add`-ed before committing. In JJ, file tracking is controlled entirely by `.gitignore` — if `.planning` is ignored, JJ won't track it. No framework logic needed.

**What to remove:**

The following pattern appears in 16 files and should be deleted along with any conditional branches that depend on `COMMIT_PLANNING_DOCS`:

```bash
COMMIT_PLANNING_DOCS=$(cat .planning/config.json 2>/dev/null | grep -o '"commit_docs"[[:space:]]*:[[:space:]]*[^,}]*' | grep -o 'true\|false' || echo "true")
git check-ignore -q .planning 2>/dev/null && COMMIT_PLANNING_DOCS=false
```

**Files affected:**

Agents (5):
- `agents/gsd-executor.md`
- `agents/gsd-planner.md`
- `agents/gsd-debugger.md`
- `agents/gsd-phase-researcher.md`
- `agents/gsd-research-synthesizer.md`

Commands (3):
- `commands/gsd/new-milestone.md`
- `commands/gsd/pause-work.md`
- `commands/gsd/plan-milestone-gaps.md`

Workflows (7):
- `get-shit-done/workflows/execute-plan.md`
- `get-shit-done/workflows/execute-phase.md`
- `get-shit-done/workflows/complete-milestone.md`
- `get-shit-done/workflows/verify-work.md`
- `get-shit-done/workflows/diagnose-issues.md`
- `get-shit-done/workflows/discuss-phase.md`
- `get-shit-done/workflows/map-codebase.md`

References (1):
- `get-shit-done/references/planning-config.md`

**Also remove:** The `commit_docs` field from the `.planning/config.json` schema documentation in `planning-config.md`, and any `If COMMIT_PLANNING_DOCS=false: skip` / `If COMMIT_PLANNING_DOCS=true:` conditional blocks downstream of the detection pattern.
