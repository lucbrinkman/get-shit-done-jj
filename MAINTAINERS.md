# GSD Maintainer Guide

Quick reference for release workflows and maintenance tasks.

## Release Workflow

### Standard Release

```bash
/gsd-publish-version
```

The command walks you through:
1. Check uncommitted changes
2. Generate changelog from commits
3. Review and approve changelog
4. Update CHANGELOG.md
5. Bump version (`npm version patch|minor|major`)
6. Push to GitHub with tags

GitHub Actions then:
- Creates GitHub Release from CHANGELOG.md
- Publishes to npm

### Pre-release (Experimental Features)

For risky features, ship as alpha first:

```bash
# Bump to alpha
npm version prerelease --preid=alpha

# Push
jj git push
```

Pre-release tags (`v1.10.0-alpha.0`) don't trigger npm publish or GitHub Release creation. Users opt-in explicitly.

If it works, promote to stable:
```bash
npm version minor  # or patch
jj git push
```

If it fails, delete the tag and move on.

### Hotfix

Production broken? Skip changelog ceremony:

```bash
# Fix the issue
jj describe -m "fix(install): handle Windows UNC paths"
jj new

# Bump and push
npm version patch
jj git push
```

## Version Cadence

| Type | When | Example |
|------|------|---------|
| MAJOR | Breaking changes | Command removed, format changed |
| MINOR | New features | New command, new capability |
| PATCH | Bug fixes | Batch weekly, or immediately if critical |

## Changelog Format

Follow [Keep a Changelog](https://keepachangelog.com/):

```markdown
## [1.10.0] - 2025-01-22

### Added
- New `/gsd:whats-new` command

### Changed
- Improved parallel execution

### Fixed
- STATE.md progress calculation

### Removed
- **BREAKING:** Deprecated ISSUES.md system
```

## Dependency Policy

Before adding dependencies:
1. Check bundle size impact
2. Evaluate if it's worth the weight
3. Consider if the functionality can be implemented without it

The codebase intelligence system was removed partly because sql.js added 21MB.

## Recovery Procedures

### Broken npm Release

Within 72 hours:
```bash
npm unpublish get-shit-done-cc@1.9.5
```

After 72 hours: Publish a fix as new patch version.

### Wrong Tag

```bash
# Delete and recreate bookmark
jj bookmark delete v1.9.5
jj bookmark create v1.9.5
jj git push
```

### Missing Changelog Entry

Either amend the release commit or add a follow-up commit with the missing content.

## CI/CD Setup

### Required Secrets

In GitHub repo settings → Secrets → Actions:

- `NPM_TOKEN`: npm automation token with publish access

`GITHUB_TOKEN` is provided automatically.

### Branch Protection (Optional)

Settings → Branches → Add rule for `main`:
- Require status checks: `test`, `lint`
- Disable force pushes

## Reviewing Contributor PRs

Checklist:
- [ ] Follows conventional commit format
- [ ] No enterprise patterns or filler
- [ ] CHANGELOG.md updated for user-facing changes
- [ ] No unnecessary dependencies
- [ ] Tested on Windows if touching paths
