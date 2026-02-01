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
