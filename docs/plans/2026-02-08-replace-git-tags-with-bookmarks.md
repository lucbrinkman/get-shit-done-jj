# Replace Git Tags with JJ Bookmarks

**Goal:** Remove all direct `git` tag usage from the milestone completion workflow. Use JJ bookmarks as release markers instead.

**Why:** The current implementation reaches into `.jj/repo/store/git` to run raw git commands, bypassing JJ. This is fragile and violates the project's non-colocated, JJ-native design. Additionally, `jj git push --tag` doesn't exist — that command will fail.

**Approach:** Use a permanent bookmark like `release/v1.0` instead of a git tag. Bookmarks can be pushed with `jj git push -b release/v1.0`.

**Files affected:**

- `get-shit-done/workflows/complete-milestone.md` — The `<step name="git_tag">` section (lines ~678-721) creates annotated git tags via `git -C .jj/repo/store/git tag -a` and attempts `jj git push --tag`. Replace with:
  ```bash
  jj bookmark create release/v[X.Y]
  jj git push -b release/v[X.Y]
  ```

- `commands/gsd/complete-milestone.md` — References "git tagged" in the output description (line 16). Update to "bookmarked".

- `commands/gsd/help.md` — Line 192 mentions "Creates jj bookmark for the release". Already correct in spirit, just verify it doesn't reference tags elsewhere.

**Also remove:** The fallback `|| git tag -a` pattern on line 696 of `complete-milestone.md`, which assumes a colocated repo.
