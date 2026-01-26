# CI/CD Guide for pipe-works Organization

This guide explains the CI/CD workflows available to all pipe-works projects.

## Table of Contents

- [Overview](#overview)
- [Reusable Workflows](#reusable-workflows)
  - [Python CI](#python-ci)
  - [Release Please](#release-please)
  - [Dependency Update](#dependency-update)
- [Configuration](#configuration)
- [Conventional Commits](#conventional-commits)
- [Troubleshooting](#troubleshooting)

---

## Overview

The pipe-works organization uses centralized reusable workflows to maintain consistency across all projects. These workflows are defined in the [pipe-works/.github](https://github.com/pipe-works/.github) repository and referenced by individual project workflows.

### Benefits

- **Consistency**: All projects follow the same CI/CD patterns
- **Maintainability**: Fix once, apply everywhere
- **Best Practices**: Incorporates lessons from pipeworks_name_generation
- **Automation**: Semantic versioning, changelogs, and releases

---

## Reusable Workflows

### Python CI

**File**: `reusable-python-ci.yml`

Comprehensive CI pipeline with code quality checks, testing, security scanning, documentation builds, and package validation.

#### Features

- **Code Quality**: Ruff linting, Black formatting, mypy type checking
- **Testing**: pytest with coverage reporting to Codecov
- **Multi-Version**: Test across Python versions (e.g., 3.12, 3.13)
- **Multi-OS**: Test on ubuntu, macos, windows (configurable)
- **Security**: Bandit static analysis + Trivy vulnerability scanning
- **Documentation**: Sphinx builds with warning thresholds
- **Package Build**: Validates package with twine
- **ML Support**: Disk cleanup and offline mode for model testing
- **All-Checks-Passed**: Single status check for branch protection

#### Usage

```yaml
name: CI

on:
  push:
    branches: [main, develop, 'release-please--*']
  pull_request:
    branches: [main, develop]
  workflow_dispatch:

permissions:
  contents: read
  security-events: write

jobs:
  ci:
    uses: pipe-works/.github/.github/workflows/reusable-python-ci.yml@main
    with:
      python-version: '3.12'
      python-versions: '["3.12", "3.13"]'
      coverage-threshold: 50
      run-security: true
      run-docs: true
      docs-warning-threshold: 0
      test-os: '["ubuntu-latest"]'
      requires-models: false
    secrets: inherit
```

#### Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `python-version` | string | `'3.12'` | Primary Python version |
| `python-versions` | string | `''` | JSON array of versions to test |
| `coverage-threshold` | number | `50` | Minimum coverage percentage |
| `run-security` | boolean | `true` | Run security scans |
| `run-docs` | boolean | `false` | Build documentation |
| `docs-warning-threshold` | number | `0` | Max doc warnings (0 = strict) |
| `test-os` | string | `'["ubuntu-latest"]'` | Operating systems to test |
| `requires-models` | boolean | `false` | Enable ML model support |
| `pytest-markers` | string | `''` | Pytest marker expression |
| `pytest-args` | string | `''` | Additional pytest arguments |

#### Examples

**Standard Python Project**:
```yaml
with:
  python-version: '3.12'
  coverage-threshold: 80
  run-security: true
  run-docs: true
```

**ML/Model Project**:
```yaml
with:
  python-version: '3.12'
  python-versions: '["3.12", "3.13"]'
  coverage-threshold: 50
  requires-models: true
  pytest-markers: 'not requires_model and not slow'
```

**Multi-OS Project**:
```yaml
with:
  python-version: '3.12'
  test-os: '["ubuntu-latest", "macos-latest", "windows-latest"]'
  coverage-threshold: 70
```

**Project with Sphinx Warnings**:
```yaml
with:
  python-version: '3.12'
  run-docs: true
  docs-warning-threshold: 60  # Allow up to 60 warnings
```

---

### Release Please

**File**: `reusable-release-please.yml`

Automated semantic versioning and changelog generation using conventional commits.

#### Features

- **Semantic Versioning**: Auto-bump versions based on commit types
- **Changelog Generation**: Automatic from conventional commits
- **Pre-1.0 Mode**: Breaking changes bump minor (0.x.0)
- **CI Integration**: Triggers CI on release-please branches
- **Branch Protection**: Reports status for merge requirements
- **Release Creation**: Automated GitHub releases

#### Usage

**Step 1**: Create configuration files in repo root

`release-please-config.json`:
```json
{
  "$schema": "https://raw.githubusercontent.com/googleapis/release-please/main/schemas/config.json",
  "packages": {
    ".": {
      "release-type": "python",
      "package-name": "your-package-name",
      "bump-minor-pre-major": true,
      "bump-patch-for-minor-pre-major": true,
      "changelog-sections": [
        {"type": "feat", "section": "Features", "hidden": false},
        {"type": "fix", "section": "Fixes", "hidden": false},
        {"type": "docs", "section": "Documentation", "hidden": false},
        {"type": "refactor", "section": "Internal Changes", "hidden": false},
        {"type": "chore", "section": "Maintenance", "hidden": true}
      ]
    }
  }
}
```

`.release-please-manifest.json`:
```json
{
  ".": "0.1.0"
}
```

**Step 2**: Create workflow

`.github/workflows/release-please.yml`:
```yaml
name: Release Please

on:
  push:
    branches: [main]

jobs:
  release:
    uses: pipe-works/.github/.github/workflows/reusable-release-please.yml@main
    with:
      release-type: 'python'
      package-name: 'your-package-name'
      bump-minor-pre-major: true
    secrets: inherit
```

#### Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `release-type` | string | `'python'` | Release type (python, node, etc.) |
| `package-name` | string | **required** | Package name for releases |
| `bump-minor-pre-major` | boolean | `true` | Bump minor for breaking pre-1.0 |
| `bump-patch-for-minor-pre-major` | boolean | `true` | Bump patch for minor pre-1.0 |

#### How It Works

1. **Push to main**: Workflow checks for conventional commits
2. **PR Creation**: Creates/updates release PR with changelog
3. **CI Trigger**: Automatically triggers CI on release PR branch
4. **Status Report**: Reports CI status back to PR
5. **Merge PR**: Merging creates GitHub release with assets
6. **Version Bump**: Updates version in `pyproject.toml` and manifest

---

### Dependency Update

**File**: `reusable-dependency-update.yml`

Automated weekly checks for outdated dependencies with GitHub issue creation.

#### Features

- **Scheduled Checks**: Weekly cron job
- **Issue Management**: Creates/updates single tracking issue
- **Dependency Table**: Lists current vs. latest versions
- **Update Commands**: Provides copy-paste upgrade commands
- **Configurable**: Custom schedule and labels

#### Usage

```yaml
name: Dependency Update

on:
  schedule:
    - cron: '0 9 * * 1'  # Monday 9 AM UTC
  workflow_dispatch:

jobs:
  update:
    uses: pipe-works/.github/.github/workflows/reusable-dependency-update.yml@main
    with:
      labels: 'dependencies,automated'
      python-version: '3.12'
    secrets: inherit
```

#### Inputs

| Input | Type | Default | Description |
|-------|------|---------|-------------|
| `labels` | string | `'dependencies,automated'` | Labels for issues |
| `python-version` | string | `'3.12'` | Python version to check |

#### Issue Format

```markdown
## Outdated Dependencies

Found 5 package(s) with newer versions available:

| Package | Current | Latest | Type |
|---------|---------|--------|------|
| pytest | 7.4.0 | 8.0.0 | wheel |
| black | 23.0.0 | 24.0.0 | wheel |

### Action Required

Review these updates and consider:
1. Checking changelogs for breaking changes
2. Updating version constraints in `pyproject.toml`
3. Running tests after updates
4. Creating a PR with the updates

### Update Commands

\`\`\`bash
# Update specific package
pip install --upgrade pytest

# Update all packages (use with caution)
pip list --outdated --format=freeze | grep -v "^\\-e" | cut -d = -f 1 | xargs -n1 pip install -U
\`\`\`
```

---

## Configuration

### Branch Protection

Enable branch protection on `main` with:

1. **Required Status Check**: `All Checks Passed`
   - This single check covers all CI jobs
   - Simplifies branch protection rules

2. **Require up-to-date branches**: ✅
3. **Require conversation resolution**: ✅
4. **Include administrators**: ❌ (allow emergency fixes)

### Codecov

Add `CODECOV_TOKEN` secret to repository:

1. Go to [codecov.io](https://codecov.io)
2. Add repository
3. Copy upload token
4. Add as repository secret: `CODECOV_TOKEN`

### ReadTheDocs

1. Copy `.readthedocs.yaml` to repo root
2. Go to [readthedocs.org](https://readthedocs.org)
3. Import repository
4. Configure project settings
5. Add badge to README.md

---

## Conventional Commits

Release-please requires conventional commit format for automated versioning.

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

| Type | Version Bump | Description |
|------|--------------|-------------|
| `feat` | minor (0.x.0) | New feature |
| `fix` | patch (0.0.x) | Bug fix |
| `docs` | none | Documentation only |
| `refactor` | none | Code restructuring |
| `test` | none | Adding tests |
| `chore` | none | Build/tooling changes |
| `BREAKING CHANGE` | major (x.0.0) or minor if pre-1.0 | Breaking change |

### Examples

**New Feature**:
```
feat(name-generator): add phonetic pattern matching

Implement pattern-based syllable matching for more consistent
name generation across different input corpora.

Closes #123
```

**Bug Fix**:
```
fix(ci): correct hook execution order

Pre-commit hooks now run in correct order:
file checks → formatting → linting

Fixes #456
```

**Breaking Change**:
```
feat(api)!: remove deprecated endpoints

BREAKING CHANGE: Removed /v1/old-endpoint. Use /v2/new-endpoint instead.

Migration guide: https://docs.example.com/migration
```

**Documentation**:
```
docs: update CI/CD guide with release-please workflow

Add section explaining automated semantic versioning and
conventional commit requirements.
```

### Scope

Scope is optional but recommended:
- `feat(name-generator)`: Feature in name generator module
- `fix(ci)`: Fix in CI configuration
- `docs(api)`: API documentation update
- `refactor(core)`: Core module refactoring

### Subject

- Use imperative mood: "add" not "added" or "adds"
- Don't capitalize first letter
- No period at the end
- Keep under 50 characters

### Body

- Explain **why** not **what** (code shows what)
- Wrap at 72 characters
- Use bullet points for multiple items

### Footer

- Reference issues: `Closes #123`, `Fixes #456`
- Breaking changes: `BREAKING CHANGE: description`
- Co-authors: `Co-Authored-By: Name <email>`

---

## Troubleshooting

### CI Fails on Release-Please Branch

**Problem**: CI doesn't trigger on release-please PRs

**Solution**: Ensure ci.yml includes release-please branches:
```yaml
on:
  push:
    branches: [main, develop, 'release-please--*']
```

### Documentation Build Warnings

**Problem**: Too many Sphinx warnings fail build

**Solution**: Adjust `docs-warning-threshold`:
```yaml
with:
  docs-warning-threshold: 60  # Allow up to 60 warnings
```

### Coverage Drops Below Threshold

**Problem**: CI fails due to low coverage

**Options**:
1. Add tests to increase coverage
2. Exclude UI/glue code in `pyproject.toml`:
   ```toml
   [tool.coverage.run]
   omit = ["*/ui/*", "*/api/server.py"]
   ```
3. Lower threshold temporarily:
   ```yaml
   with:
     coverage-threshold: 40
   ```

### Release-Please Not Creating PR

**Problem**: No PR after merging to main

**Checklist**:
1. ✅ Using conventional commits?
2. ✅ `release-please-config.json` exists?
3. ✅ `.release-please-manifest.json` exists?
4. ✅ Workflow has correct permissions?
5. ✅ Package name matches `pyproject.toml`?

### Multi-OS Tests Fail on Windows

**Problem**: Tests pass on ubuntu/macos but fail on windows

**Common Issues**:
- Path separators (`/` vs `\\`)
- Line endings (LF vs CRLF)
- Case-sensitive file systems
- Missing dependencies

**Solution**: Use `os.path` or `pathlib` for paths, configure git:
```bash
git config core.autocrlf true
```

### Security Scan Permissions Error

**Problem**: "Resource not accessible by integration"

**Solution**: Add permissions to workflow:
```yaml
permissions:
  contents: read
  security-events: write
```

### Dependency Issue Spam

**Problem**: Too many dependency update issues

**Solution**:
1. Adjust schedule to monthly:
   ```yaml
   on:
     schedule:
       - cron: '0 9 1 * *'  # First day of month
   ```
2. Use Dependabot instead (disable workflow)

---

## Best Practices

### 1. Start Strict, Relax as Needed

Begin with:
- `coverage-threshold: 80`
- `docs-warning-threshold: 0`
- Multi-OS testing if public library

Relax requirements if they become blockers.

### 2. Use Pre-Commit Hooks

Install pre-commit hooks locally:
```bash
pip install pre-commit
pre-commit install
```

Catches issues before CI runs.

### 3. Write Good Commit Messages

Good:
```
feat(parser): add support for nested structures

Implement recursive parsing for nested data structures,
enabling more complex configuration files.

Closes #234
```

Bad:
```
update stuff
```

### 4. Monitor CI Performance

If CI takes > 15 minutes:
- Use pytest markers to skip slow tests in CI
- Cache dependencies
- Run expensive tests only on main branch

### 5. Keep Dependencies Updated

Weekly dependency checks help:
- Catch security vulnerabilities early
- Avoid large upgrade jumps
- Stay current with ecosystem

---

## Additional Resources

- [pipe-works/.github repository](https://github.com/pipe-works/.github)
- [Release Please Documentation](https://github.com/googleapis/release-please)
- [Conventional Commits Specification](https://www.conventionalcommits.org/)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Codecov Documentation](https://docs.codecov.com/)
- [ReadTheDocs Documentation](https://docs.readthedocs.io/)

---

**Last Updated**: 2026-01-26
**Version**: 1.0
**Maintainer**: pipe-works organization
