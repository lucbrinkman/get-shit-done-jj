# jj

I use jj in all my repos. Don't be afraid to make destructive changes, as long as you run `jj status` before and after them.

## Core Concepts (Git vs jj)

**Working Copy as Commit**: Unlike Git's "dirty" working directory, jj automatically commits working copy changes on every command. No staging area - files are tracked and committed automatically.

**Change ID vs Commit ID**:

- **Change ID** (12 letters, z-k range): Stable identifier that persists through rewrites (like `ywnkulko`)
- **Commit ID** (hex): Changes when commit is rewritten (like `46b50ed7`)
- Use change IDs for most operations; commit IDs for specific snapshots

**No Current Branch**: Bookmarks don't automatically move. No equivalent to Git's HEAD following a branch. Update bookmarks manually with `jj bookmark set`.

**Always Succeeds**: Operations like rebase never fail - conflicts are recorded in commits and resolved later.

**Consistent Parameters**: jj cli parameters mean the same thing when possible

**Git Compatibility**: jj stores commits in .git and uses .gitignore

## Instant Tutorial

```bash
jj git init --colocate   # Create repo (git-backed)
echo "hello" > file.txt  # Edit files
jj describe -m "msg"     # Describe current change (and detect and track file.txt)
echo "hiya" > file.txt   # Edit again
jj new                   # Start new change (after saving "hiya" to change named "msg")
jj log                   # View history
jj log -p                # View history with patches
```

## File Management

```bash
jj status                # Status (like git status) - run often, updates current change and saves anonymous undo history
jj file list             # List tracked files
jj file untrack <file>   # Untrack ignored file (keep in working copy - requires .gitignore)
jj restore <file>        # Restore file from parent
jj restore               # Clear all working copy changes (jj undo if done by accident)
```

Files automatically tracked when added. Use `.gitignore` for exclusions.

## Basic Workflow

```bash
# Start work
jj new [parent] -m "msg"   # Create new change (somewhat like git checkout [parent] && git add . && git commit -m '' && git checkout [parent])
jj describe -m "msg"     # Set description of current WIP (anytime, not just at commit - like git commit --amend -m 'msg')
# or
jj commit -m "msg"       # describe and new combined - more familiar

# View work
jj show [rev]           # Show commit diff (like git show)
jj diff [--from X --to Y] # Show diffs
jj log                  # View history (like git log --graph)

# "jj commit" is not "git commit", "jj st" is - changes auto-committed on every command, just run "jj st" to commit to evolog and update current change
```

## History and Branching

**Anonymous Branches**: Don't need names - just create changes with `jj new`. View with `jj log -r 'heads(all())'`.

**Revsets** (like Git's rev syntax but more powerful):

```bash
@                       # Current working copy
@-                      # Parent of working copy
trunk()                 # Main branch (auto-detected)
heads(all())           # All branch heads
::@                    # Ancestors of @ (like git log)
```

**Branch Operations**:

```bash
jj new parent1 parent2  # Create merge (no special merge command)
jj rebase -r X -d Y     # Move single revision X to destination D
jj rebase -s X -d Y     # Move X and descendants to D
jj duplicate X -d Y     # Copy commit (like git cherry-pick)
```

## Change Manipulation

```bash
# Split/combine
jj split [-r X]         # Split commit interactively (unavailable to claude)
jj squash               # Move @ into parent (like git commit --amend)
jj squash -i            # Interactive selection (unavailable to claude)
jj squash --from X --into Y  # Move changes between arbitrary commits

# Edit
jj edit X               # Set working copy to commit X
jj next/prev            # Move working copy up/down
jj diffedit -r X        # Edit commit's diff directly (unavailable to claude)
```

## Conflict Handling

**Conflicts are Committable**: Can rebase/merge conflicted commits. Resolve when ready.

```bash
# Conflict markers differ from Git:
<<<<<<< Conflict 1 of 1
+++++++ Contents of side #1
content1
%%%%%%% Changes from base to side #2
-old line
+new line
>>>>>>> Conflict 1 of 1 ends
```

Resolution: Apply the diff to the snapshot. Use `jj resolve` for 3-way merge tools.

**Auto-rebase**: Descendants automatically rebase when you rewrite commits - even conflicted ones.

## Remote Operations

**Bookmarks = Git Branches**:

```bash
jj bookmark create name  # Create bookmark at @
jj bookmark set name     # Move bookmark to @
jj bookmark list         # List bookmarks
jj bookmark track name@remote  # Track remote bookmark
```

**Git Interop**:

```bash
jj git fetch            # Fetch from remotes
jj git push             # Push tracked bookmarks
jj git push -c @        # Create bookmark and push current change
jj git push --bookmark name  # Push specific bookmark
```

## Advanced Operations

**Operation Log** (like unlimited undo):

```bash
jj op log               # View operation history
jj undo                 # Undo last operation
jj op restore X         # Restore repo to operation X
```

**Working with Git repos**:

```bash
jj git clone URL        # Clone git repo
jj git init --colocate  # Initialize in existing git repo
```

## Two Workflow Patterns

**Squash Workflow** (like git add -p):

1. `jj describe -m "feature"` - describe planned work, DOES NOT MAKE A NEW CHANGE
2. `jj new` - create scratch space, DOES make a new change
3. Make changes, then `jj squash [-i]` to move into described commit

**Edit Workflow**:

1. `jj new -m "feature"` - create and start working
2. If need prep work: `jj new -B @ -m "prep"` (insert before current)
3. `jj next --edit` to return to feature work

## Cheat Sheet

| Task               | jj command              | Git equivalent                |
| ------------------ | ----------------------- | ----------------------------- |
| Status             | `jj st`                 | `git status`                  |
| Start new work     | `jj new`                | `git checkout -b branch`      |
| Commit message     | `jj describe -m "msg"`  | `git commit --amend -m "msg"` |
| View history       | `jj log`                | `git log --graph`             |
| Show commit        | `jj show X`             | `git show X`                  |
| Interactive rebase | `jj split`, `jj squash` | `git rebase -i`               |
| Cherry-pick        | `jj duplicate X -d Y`   | `git cherry-pick X`           |
| Stash              | `jj new @-`             | `git stash`                   |
| Undo last action   | `jj undo`               | N/A                           |
| Push current work  | `jj git push -c @`      | `git push -u origin branch`   |

## GitHub CLI with Non-Colocated Repos

The `gh` CLI can't find the git repo in non-colocated jj repos. Set `GIT_DIR` to fix:

```bash
# One-off command
GIT_DIR=.jj/repo/store/git gh pr create --base main --title "My PR"

# Or use direnv - create .envrc in repo root:
export GIT_DIR=$PWD/.jj/repo/store/git
```

## Key Files to Read More

In ~/jj/docs,

- **working-copy.md**: Auto-commit behavior, conflict handling
- **git-comparison.md**: Detailed Git equivalencies
- **revsets.md**: Query language for selecting commits
- **operation-log.md**: Undo system
- **bookmarks.md**: Branch management
- **conflicts.md**: First-class conflict support

## Remember for Git Users

1. Run `jj st` frequently - it's fast and shows what jj did
2. Changes auto-committed ≠ changes shared (still need push)
3. Bookmark movement is manual (`jj bookmark set`)
4. Conflicts don't block operations - resolve when convenient
5. Working copy is always a commit; you can `jj edit` others
6. `jj new` starts work, `jj describe` names current work, or `jj commit` does both; use new+describe or commit
7. Everything is undoable with `jj undo` (one op) or `jj op restore` (target op)

REMEMBER: always run `jj st` before and after edits
