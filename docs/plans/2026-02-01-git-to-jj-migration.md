# Git to JJ Migration Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Replace all git references with jj equivalents to create a jj-native fork of get-shit-done.

**Architecture:** Mechanical translation of git commands to jj commands. No staging area in jj — working copy is always committed. Safety net shifts from selective staging to `jj st` review before `jj new`.

**Tech Stack:** jj (Jujutsu VCS), markdown documentation

---

## Translation Reference

Use this table for all conversions:

| Git | jj |
|-----|-----|
| `git init` | `jj git init` (non-colocated, intentional — use `--colocate` only if git tool compat needed) |
| `git add X && git commit -m "msg"` | `jj describe -m "msg"` then `jj new` |
| `git add -u && git commit -m "msg"` | `jj describe -m "msg"` then `jj new` |
| `git add -A && git commit -m "msg"` | `jj describe -m "msg"` then `jj new` |
| `git status` | `jj st` |
| `git log` | `jj log` |
| `git log --grep="X"` | `jj log -r 'description(X)'` |
| `git diff` | `jj diff` |
| `git show X` | `jj show X` |
| `git reset --hard` | `jj restore` or `jj undo` |
| `git reset .planning/` | `jj restore .planning/` |
| `git commit --amend` | `jj squash` |
| `git commit --amend --no-edit` | `jj squash` |
| `git push` | `jj git push` |
| `git fetch` | `jj git fetch` |
| `git branch` | `jj bookmark list` |
| `git checkout -b X` | Not directly applicable — jj uses anonymous changes. `jj new` to start work, `jj bookmark create X` only if you need a push target. |
| `git cherry-pick X` | `jj duplicate X -d @` |
| `git bisect` | `jj` has no direct equivalent (note in docs) |
| `git blame` | `jj file annotate` (or use git backend) |
| `[ -d .git ]` | `[ -d .jj ]` |
| `.git` directory references | `.jj` directory references |
| `.gitignore` | `.gitignore` (jj uses same file) |

**Behavioral changes:**
- Remove all "stage files individually" / "NEVER use git add ." instructions
- Replace with: "Review `jj st` before running `jj new` to verify only intended files changed"
- Add: "Use `jj restore <file>` to undo accidental modifications before `jj new`"

---

## Task 1: Create jj-integration.md (Core Reference)

**Files:**
- Delete: `get-shit-done/references/git-integration.md`
- Create: `get-shit-done/references/jj-integration.md`

**Step 1: Read the existing git-integration.md**

Read `get-shit-done/references/git-integration.md` to understand structure.

**Step 2: Create jj-integration.md with translated content**

Create `get-shit-done/references/jj-integration.md` with:

```markdown
<overview>
JJ integration for GSD framework.
</overview>

<core_principle>

**Commit outcomes, not process.**

The jj log should read like a changelog of what shipped, not a diary of planning activity.
</core_principle>

<commit_points>

| Event                   | Commit? | Why                                              |
| ----------------------- | ------- | ------------------------------------------------ |
| BRIEF + ROADMAP created | YES     | Project initialization                           |
| PLAN.md created         | NO      | Intermediate - commit with plan completion       |
| RESEARCH.md created     | NO      | Intermediate                                     |
| DISCOVERY.md created    | NO      | Intermediate                                     |
| **Task completed**      | YES     | Atomic unit of work (1 commit per task)         |
| **Plan completed**      | YES     | Metadata commit (SUMMARY + STATE + ROADMAP)     |
| Handoff created         | YES     | WIP state preserved                              |

</commit_points>

<jj_check>

```bash
[ -d .jj ] && echo "JJ_EXISTS" || echo "NO_JJ"
```

If NO_JJ: Run `jj git init` silently (non-colocated). GSD projects always get their own repo.
</jj_check>

<commit_formats>

<format name="initialization">
## Project Initialization (brief + roadmap together)

```
docs: initialize [project-name] ([N] phases)

[One-liner from PROJECT.md]

Phases:
1. [phase-name]: [goal]
2. [phase-name]: [goal]
3. [phase-name]: [goal]
```

How to commit:

```bash
jj describe -m "docs: initialize [project-name] ([N] phases)

[One-liner from PROJECT.md]

Phases:
1. [phase-name]: [goal]
2. [phase-name]: [goal]
3. [phase-name]: [goal]
"
jj new
```

</format>

<format name="task-completion">
## Task Completion (During Plan Execution)

Each task gets its own commit immediately after completion.

```
{type}({phase}-{plan}): {task-name}

- [Key change 1]
- [Key change 2]
- [Key change 3]
```

**Commit types:**
- `feat` - New feature/functionality
- `fix` - Bug fix
- `test` - Test-only (TDD RED phase)
- `refactor` - Code cleanup (TDD REFACTOR phase)
- `perf` - Performance improvement
- `chore` - Dependencies, config, tooling

**Before committing:**

```bash
jj st  # Review changes - verify only task-relevant files modified
# If unwanted files changed:
jj restore path/to/unwanted/file
```

**Examples:**

```bash
# Standard task
jj describe -m "feat(08-02): create user registration endpoint

- POST /auth/register validates email and password
- Checks for duplicate users
- Returns JWT token on success
"
jj new

# TDD task - RED phase
jj describe -m "test(07-02): add failing test for JWT generation

- Tests token contains user ID claim
- Tests token expires in 1 hour
- Tests signature verification
"
jj new

# TDD task - GREEN phase
jj describe -m "feat(07-02): implement JWT generation

- Uses jose library for signing
- Includes user ID and expiry claims
- Signs with HS256 algorithm
"
jj new
```

</format>

<format name="plan-completion">
## Plan Completion (After All Tasks Done)

After all tasks committed, one final metadata commit captures plan completion.

```
docs({phase}-{plan}): complete [plan-name] plan

Tasks completed: [N]/[N]
- [Task 1 name]
- [Task 2 name]
- [Task 3 name]

SUMMARY: .planning/phases/XX-name/{phase}-{plan}-SUMMARY.md
```

How to commit:

```bash
jj st  # Verify .planning/ files are the only changes
jj describe -m "docs({phase}-{plan}): complete [plan-name] plan

Tasks completed: [N]/[N]
- [Task 1 name]
- [Task 2 name]
- [Task 3 name]

SUMMARY: .planning/phases/XX-name/{phase}-{plan}-SUMMARY.md
"
jj new
```

**Note:** Code files NOT included - already committed per-task.

</format>

<format name="handoff">
## Handoff (WIP)

```
wip: [phase-name] paused at task [X]/[Y]

Current: [task name]
[If blocked:] Blocked: [reason]
```

How to commit:

```bash
jj describe -m "wip: [phase-name] paused at task [X]/[Y]

Current: [task name]
Blocked: [reason]
"
jj new
```

</format>
</commit_formats>

<safety_and_recovery>

## Safety & Recovery with Operation Log

jj tracks every operation. Unlike git's reflog, this is first-class and easy to use.

```bash
jj op log              # See all operations
jj undo                # Undo last operation
jj op restore <op-id>  # Restore to any previous state
```

**Before any destructive operation**, run `jj st`. This creates an operation log entry, making your current state recoverable with `jj undo`.

**Recovery scenarios:**

| Situation | How to fix |
|-----------|------------|
| Committed wrong files | `jj undo`, fix with `jj restore <file>`, `jj new` |
| Messed up a rebase | `jj undo` or `jj op restore <op-id>` |
| Accidentally ran `jj restore` | `jj undo` |
| Want to see what changed | `jj diff` or `jj show @` |
| Need to check history | `jj op log` shows what each operation did |

</safety_and_recovery>

<example_log>

**Example jj log (per-task commits):**
```
# Phase 04 - Checkout
1a2b3c docs(04-01): complete checkout flow plan
4d5e6f feat(04-01): add webhook signature verification
7g8h9i feat(04-01): implement payment session creation
0j1k2l feat(04-01): create checkout page component

# Phase 03 - Products
3m4n5o docs(03-02): complete product listing plan
6p7q8r feat(03-02): add pagination controls
9s0t1u feat(03-02): implement search and filters
2v3w4x feat(03-01): create product catalog schema

# Phase 02 - Auth
5y6z7a docs(02-02): complete token refresh plan
8b9c0d feat(02-02): implement refresh token rotation
1e2f3g test(02-02): add failing test for token refresh
4h5i6j docs(02-01): complete JWT setup plan
7k8l9m feat(02-01): add JWT generation and validation
0n1o2p chore(02-01): install jose library

# Phase 01 - Foundation
3q4r5s docs(01-01): complete scaffold plan
6t7u8v feat(01-01): configure Tailwind and globals
9w0x1y feat(01-01): set up Prisma with database
2z3a4b feat(01-01): create Next.js 15 project

# Initialization
5c6d7e docs: initialize ecommerce-app (5 phases)
```

Each plan produces 2-4 commits (tasks + metadata). Clear, granular, bisectable.

</example_log>

<anti_patterns>

**Still don't commit (intermediate artifacts):**
- PLAN.md creation (commit with plan completion)
- RESEARCH.md (intermediate)
- DISCOVERY.md (intermediate)
- Minor planning tweaks
- "Fixed typo in roadmap"

**Do commit (outcomes):**
- Each task completion (feat/fix/test/refactor)
- Plan completion metadata (docs)
- Project initialization (docs)

**Key principle:** Commit working code and shipped outcomes, not planning process.

</anti_patterns>

<commit_strategy_rationale>

## Why Per-Task Commits?

**Context engineering for AI:**
- JJ history becomes primary context source for future Claude sessions
- `jj log -r 'description("{phase}-{plan}")'` shows all work for a plan
- `jj show <change-id>` shows exact changes per task
- Less reliance on parsing SUMMARY.md = more context for actual work

**Failure recovery:**
- Task 1 committed, Task 2 failed
- Claude in next session: sees task 1 complete, can retry task 2
- Can `jj undo` or `jj restore` to recover

**Debugging:**
- Each commit is independently viewable with `jj show`
- `jj file annotate` traces line to specific task context
- Each change is independently recoverable

**Observability:**
- Solo developer + Claude workflow benefits from granular attribution
- Atomic commits are VCS best practice
- "Commit noise" irrelevant when consumer is Claude, not humans

</commit_strategy_rationale>

<gh_cli_compatibility>

## GitHub CLI Compatibility

Since jj repos are non-colocated by default, the `gh` CLI won't find the git directory. Use this workaround:

```bash
# One-off command
GIT_DIR=.jj/repo/store/git gh pr create --title "My PR"

# Or add to .envrc for direnv users
export GIT_DIR=$PWD/.jj/repo/store/git
```

</gh_cli_compatibility>
```

**Step 3: Delete git-integration.md**

```bash
rm get-shit-done/references/git-integration.md
```

**Step 4: Verify the new file exists**

```bash
cat get-shit-done/references/jj-integration.md | head -20
```

**Step 5: Commit**

```bash
jj describe -m "docs: replace git-integration with jj-integration

- Translates all git commands to jj equivalents
- Removes staging area instructions (jj has no staging)
- Adds jj op log safety and recovery section
- Adds gh CLI compatibility for non-colocated repos
"
jj new
```

---

## Task 2: Update planning-config.md

**Files:**
- Modify: `get-shit-done/references/planning-config.md`

**Step 1: Read the file**

Read `get-shit-done/references/planning-config.md` to find git references.

**Step 2: Replace git references**

Find and replace:
- `git add`/`git commit` → `jj describe` + `jj new`
- `.gitignore` stays as-is (jj uses same file)
- `git` references in explanatory text → `jj`

**Step 3: Commit**

```bash
jj describe -m "docs: update planning-config for jj"
jj new
```

---

## Task 3: Update Commands (Batch 1 - High Git Usage)

**Files:**
- Modify: `commands/gsd/execute-phase.md`
- Modify: `commands/gsd/new-project.md`
- Modify: `commands/gsd/new-milestone.md`

These have the most git references. Apply translation table.

**Step 1: Update execute-phase.md**

Key changes:
- `git add -u && git commit` → `jj describe -m "..." && jj new`
- `git add .planning/ROADMAP.md` → remove (jj tracks automatically)
- Remove "Stage:" instructions → replace with "Review `jj st` before `jj new`"
- Remove "NEVER use `git add .`" → replace with "Review `jj st` to verify only intended files changed"
- `git add` individual files → remove entirely

**Step 2: Update new-project.md**

Key changes:
- `git init` → `jj git init`
- All `git add X && git commit` → `jj describe -m "..." && jj new`

**Step 3: Update new-milestone.md**

Same pattern as new-project.md.

**Step 4: Commit**

```bash
jj describe -m "docs: update high-traffic commands for jj

- execute-phase.md
- new-project.md
- new-milestone.md
"
jj new
```

---

## Task 4: Update Commands (Batch 2 - Medium Git Usage)

**Files:**
- Modify: `commands/gsd/pause-work.md`
- Modify: `commands/gsd/quick.md`
- Modify: `commands/gsd/plan-milestone-gaps.md`
- Modify: `commands/gsd/complete-milestone.md`

Apply translation table to all.

**Step 1-4: Update each file**

Same pattern: `git add X && git commit` → `jj describe -m "..." && jj new`

**Step 5: Commit**

```bash
jj describe -m "docs: update medium-traffic commands for jj

- pause-work.md
- quick.md
- plan-milestone-gaps.md
- complete-milestone.md
"
jj new
```

---

## Task 5: Update Commands (Batch 3 - Low Git Usage)

**Files:**
- Modify: `commands/gsd/add-todo.md`
- Modify: `commands/gsd/check-todos.md`
- Modify: `commands/gsd/remove-phase.md`
- Modify: `commands/gsd/settings.md`
- Modify: `commands/gsd/help.md`

**Step 1-5: Update each file**

Same translation pattern.

**Step 6: Commit**

```bash
jj describe -m "docs: update low-traffic commands for jj

- add-todo.md
- check-todos.md
- remove-phase.md
- settings.md
- help.md
"
jj new
```

---

## Task 6: Update Agents (Batch 1 - High Git Usage)

**Files:**
- Modify: `agents/gsd-executor.md`
- Modify: `agents/gsd-planner.md`
- Modify: `agents/gsd-debugger.md`

These have the most git references among agents.

**Step 1: Update gsd-executor.md**

Key changes:
- Remove "Stage each file individually (NEVER use `git add .`)"
- Replace with "Review `jj st` before `jj new` to verify changes"
- All `git add` + `git commit` → `jj describe` + `jj new`

**Step 2: Update gsd-planner.md**

Same pattern.

**Step 3: Update gsd-debugger.md**

Note: This file uses `git add -A` and `git reset .planning/`.
Replace with: `jj restore .planning/` if needed, then `jj describe` + `jj new`.

**Step 4: Commit**

```bash
jj describe -m "docs: update high-traffic agents for jj

- gsd-executor.md
- gsd-planner.md
- gsd-debugger.md
"
jj new
```

---

## Task 7: Update Agents (Batch 2 - Low Git Usage)

**Files:**
- Modify: `agents/gsd-codebase-mapper.md`
- Modify: `agents/gsd-phase-researcher.md`
- Modify: `agents/gsd-research-synthesizer.md`

**Step 1-3: Update each file**

Same translation pattern.

**Step 4: Commit**

```bash
jj describe -m "docs: update low-traffic agents for jj

- gsd-codebase-mapper.md
- gsd-phase-researcher.md
- gsd-research-synthesizer.md
"
jj new
```

---

## Task 8: Update Workflows (Batch 1)

**Files:**
- Modify: `get-shit-done/workflows/execute-plan.md`
- Modify: `get-shit-done/workflows/execute-phase.md`
- Modify: `get-shit-done/workflows/complete-milestone.md`

These have the most git references among workflows.

**Step 1-3: Update each file**

Same translation pattern. Pay attention to:
- Remove all "stage files individually" instructions
- Replace `git commit --amend --no-edit` with `jj squash`

**Step 4: Commit**

```bash
jj describe -m "docs: update high-traffic workflows for jj

- execute-plan.md
- execute-phase.md
- complete-milestone.md
"
jj new
```

---

## Task 9: Update Workflows (Batch 2)

**Files:**
- Modify: `get-shit-done/workflows/diagnose-issues.md`
- Modify: `get-shit-done/workflows/discuss-phase.md`
- Modify: `get-shit-done/workflows/map-codebase.md`
- Modify: `get-shit-done/workflows/verify-work.md`

**Step 1-4: Update each file**

Same translation pattern.

**Step 5: Commit**

```bash
jj describe -m "docs: update remaining workflows for jj

- diagnose-issues.md
- discuss-phase.md
- map-codebase.md
- verify-work.md
"
jj new
```

---

## Task 10: Update Templates

**Files:**
- Modify: `get-shit-done/templates/milestone.md`
- Modify: `get-shit-done/templates/codebase/conventions.md`

**Step 1-2: Update each file**

Find any git references and translate.

**Step 3: Commit**

```bash
jj describe -m "docs: update templates for jj"
jj new
```

---

## Task 11: Update GSD-STYLE.md

**Files:**
- Modify: `GSD-STYLE.md`

**Step 1: Read and update**

Find the git reference (line ~308: "Stage files individually (never `git add .`)") and replace with jj equivalent.

**Step 2: Commit**

```bash
jj describe -m "docs: update GSD-STYLE for jj"
jj new
```

---

## Task 12: Update CLAUDE.md

**Files:**
- Modify: `CLAUDE.md`

**Step 1: Review current content**

Note: CLAUDE.md already contains jj documentation (it's a copy of jj.md). Check if any git references remain that need updating.

**Step 2: Update if needed**

Remove any remaining git-specific instructions.

**Step 3: Commit**

```bash
jj describe -m "docs: finalize CLAUDE.md for jj"
jj new
```

---

## Task 13: Update Project Docs

**Files:**
- Modify: `README.md`
- Modify: `CONTRIBUTING.md`
- Modify: `MAINTAINERS.md`

**Step 1: Update README.md**

- Update any git examples to jj
- Note this is a jj-native fork if adding a header note

**Step 2: Update CONTRIBUTING.md**

- Release process uses git; update to jj

**Step 3: Update MAINTAINERS.md**

- Hotfix process uses git; update to jj

**Step 4: Commit**

```bash
jj describe -m "docs: update project docs for jj

- README.md
- CONTRIBUTING.md
- MAINTAINERS.md
"
jj new
```

---

## Task 14: Clean Up

**Files:**
- Delete: `jj.md` (content now in CLAUDE.md and jj-integration.md)
- Delete: `jj.md:Zone.Identifier` (Windows artifact)

**Step 1: Remove redundant files**

```bash
rm jj.md jj.md:Zone.Identifier
```

**Step 2: Final verification**

```bash
# Search for remaining git references
grep -r '\bgit\b' --include='*.md' . | grep -v CHANGELOG.md | head -20
```

Review any remaining references. CHANGELOG.md can keep historical git references.

**Step 3: Commit**

```bash
jj describe -m "chore: remove redundant jj.md files"
jj new
```

---

## Task 15: Update CHANGELOG.md

**Files:**
- Modify: `CHANGELOG.md`

**Step 1: Add migration entry**

Add entry at top:

```markdown
## [Unreleased] - jj Migration

### Changed
- Migrated from git to jj (Jujutsu) as the version control system
- Replaced all git commands with jj equivalents
- Removed staging area instructions (jj has no staging)
- Renamed git-integration.md to jj-integration.md

### Added
- jj operation log documentation for safety and recovery
- gh CLI compatibility instructions for non-colocated repos
```

**Step 2: Commit**

```bash
jj describe -m "docs: add jj migration to CHANGELOG"
jj new
```

---

## Verification

After all tasks complete:

1. **Search for remaining git references:**
   ```bash
   grep -rn '\bgit\b' --include='*.md' . | grep -v CHANGELOG.md
   ```
   Should return no results (except historical CHANGELOG entries).

2. **Verify jj-integration.md exists:**
   ```bash
   [ -f get-shit-done/references/jj-integration.md ] && echo "OK" || echo "MISSING"
   ```

3. **Verify git-integration.md deleted:**
   ```bash
   [ ! -f get-shit-done/references/git-integration.md ] && echo "OK" || echo "STILL EXISTS"
   ```

4. **Review jj log:**
   ```bash
   jj log -r '::@' --limit 20
   ```
   Should show clean migration commits.
