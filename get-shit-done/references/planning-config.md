<planning_config>

Configuration options for `.planning/` directory behavior.

<config_schema>
```json
"planning": {
  "commit_docs": true,
  "search_gitignored": false
},
"jj": {
  "branching_strategy": "none",
  "phase_bookmark_template": "gsd/phase-{phase}-{slug}",
  "milestone_bookmark_template": "gsd/{milestone}-{slug}"
}
```

| Option | Default | Description |
|--------|---------|-------------|
| `commit_docs` | `true` | Whether to commit planning artifacts to jj |
| `search_gitignored` | `false` | Add `--no-ignore` to broad rg searches |
| `jj.branching_strategy` | `"none"` | Jj bookmark approach: `"none"`, `"phase"`, or `"milestone"` |
| `jj.phase_bookmark_template` | `"gsd/phase-{phase}-{slug}"` | Bookmark template for phase strategy |
| `jj.milestone_bookmark_template` | `"gsd/{milestone}-{slug}"` | Bookmark template for milestone strategy |
</config_schema>

<commit_docs_behavior>

**When `commit_docs: true` (default):**
- Planning files committed normally
- SUMMARY.md, STATE.md, ROADMAP.md tracked in jj
- Full history of planning decisions preserved

**When `commit_docs: false`:**
- Skip all `jj describe`/`jj new` for `.planning/` files
- User must add `.planning/` to `.gitignore`
- Useful for: OSS contributions, client projects, keeping planning private

**Using gsd-tools.js (preferred):**

```bash
# Commit with automatic commit_docs + gitignore checks:
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs: update state" --files .planning/STATE.md

# Load config via state load (returns JSON):
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js state load)
# commit_docs is available in the JSON output

# Or use init commands which include commit_docs:
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init execute-phase "1")
# commit_docs is included in all init command outputs
```

**Auto-detection:** If `.planning/` is gitignored, `commit_docs` is automatically `false` regardless of config.json. This prevents errors when users have `.planning/` in `.gitignore`.

**Before committing:** Review `jj st` to verify only intended files changed. Use `jj restore <file>` to undo accidental modifications before `jj new`.

**Commit via CLI (handles checks automatically):**

```bash
node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs: update state" --files .planning/STATE.md
```

The CLI checks `commit_docs` config and gitignore status internally — no manual conditionals needed.

</commit_docs_behavior>

<search_behavior>

**When `search_gitignored: false` (default):**
- Standard rg behavior (respects .gitignore)
- Direct path searches work: `rg "pattern" .planning/` finds files
- Broad searches skip gitignored: `rg "pattern"` skips `.planning/`

**When `search_gitignored: true`:**
- Add `--no-ignore` to broad rg searches that should include `.planning/`
- Only needed when searching entire repo and expecting `.planning/` matches

**Note:** Most GSD operations use direct file reads or explicit paths, which work regardless of gitignore status.

</search_behavior>

<setup_uncommitted_mode>

To use uncommitted mode:

1. **Set config:**
   ```json
   "planning": {
     "commit_docs": false,
     "search_gitignored": true
   }
   ```

2. **Add to .gitignore:**
   ```
   .planning/
   ```

3. **Existing tracked files:** If `.planning/` was previously tracked:
   ```bash
   # Add .planning/ to .gitignore first, then jj will automatically
   # stop tracking these files in new changes
   echo ".planning/" >> .gitignore
   jj describe -m "chore: stop tracking planning docs"
   jj new
   ```

</setup_uncommitted_mode>

<branching_strategy_behavior>

**Bookmark Strategies:**

| Strategy | When bookmark created | Change scope | Merge point |
|----------|---------------------|--------------|-------------|
| `none` | Never | N/A | N/A |
| `phase` | At `execute-phase` start | Single phase | User squashes after phase |
| `milestone` | At first `execute-phase` of milestone | Entire milestone | At `complete-milestone` |

**When `jj.branching_strategy: "none"` (default):**
- All work in anonymous changes
- Standard GSD behavior

**When `jj.branching_strategy: "phase"`:**
- `execute-phase` creates a bookmark before execution
- Bookmark name from `phase_bookmark_template` (e.g., `gsd/phase-03-authentication`)
- All plan changes associated with that bookmark
- User squashes changes manually after phase completion
- `complete-milestone` offers to squash all phase changes

**When `jj.branching_strategy: "milestone"`:**
- First `execute-phase` of milestone creates the milestone bookmark
- Bookmark name from `milestone_bookmark_template` (e.g., `gsd/v1.0-mvp`)
- All phases in milestone commit to same bookmark
- `complete-milestone` offers to squash milestone changes to main

**Template variables:**

| Variable | Available in | Description |
|----------|--------------|-------------|
| `{phase}` | phase_bookmark_template | Zero-padded phase number (e.g., "03") |
| `{slug}` | Both | Lowercase, hyphenated name |
| `{milestone}` | milestone_bookmark_template | Milestone version (e.g., "v1.0") |

**Checking the config:**

Use `init execute-phase` which returns all config as JSON:
```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init execute-phase "1")
# JSON output includes: branching_strategy, phase_bookmark_template, milestone_bookmark_template
```

Or use `state load` for the config values:
```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js state load)
# Parse branching_strategy, phase_bookmark_template, milestone_bookmark_template from JSON
```

**Bookmark creation:**

```bash
# For phase strategy
if [ "$BRANCHING_STRATEGY" = "phase" ]; then
  PHASE_SLUG=$(echo "$PHASE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BOOKMARK_NAME=$(echo "$PHASE_BOOKMARK_TEMPLATE" | sed "s/{phase}/$PADDED_PHASE/g" | sed "s/{slug}/$PHASE_SLUG/g")
  jj bookmark create "$BOOKMARK_NAME" 2>/dev/null || jj bookmark set "$BOOKMARK_NAME"
fi

# For milestone strategy
if [ "$BRANCHING_STRATEGY" = "milestone" ]; then
  MILESTONE_SLUG=$(echo "$MILESTONE_NAME" | tr '[:upper:]' '[:lower:]' | sed 's/[^a-z0-9]/-/g' | sed 's/--*/-/g' | sed 's/^-//;s/-$//')
  BOOKMARK_NAME=$(echo "$MILESTONE_BOOKMARK_TEMPLATE" | sed "s/{milestone}/$MILESTONE_VERSION/g" | sed "s/{slug}/$MILESTONE_SLUG/g")
  jj bookmark create "$BOOKMARK_NAME" 2>/dev/null || jj bookmark set "$BOOKMARK_NAME"
fi
```

**Squash options at complete-milestone:**

| Option | Jj command | Result |
|--------|-------------|--------|
| Squash changes (recommended) | `jj squash` | Single clean change per bookmark |
| Keep all changes | (none) | Preserves all individual changes |
| Delete without squashing | `jj bookmark delete` | Discard bookmark (changes remain in history) |
| Keep bookmarks | (none) | Manual handling later |

Squashing is recommended — keeps main bookmark history clean while preserving the full development history in change IDs.

**Use cases:**

| Strategy | Best for |
|----------|----------|
| `none` | Solo development, simple projects |
| `phase` | Code review per phase, granular rollback, team collaboration |
| `milestone` | Release bookmarks, staging environments, PR per version |

</branching_strategy_behavior>

</planning_config>
