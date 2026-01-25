# Migration Guide: Updating Existing Repos to Organization Standards

> **Step-by-step guide for migrating existing pipe-works repositories to use organization-wide standards**

## Overview

This guide helps you update an existing repository to use:
- Reusable CI workflows
- Standard pre-commit hooks
- Consistent tool versions (especially black 26.1.0)
- Standard configurations

## Before You Start

1. **Backup your work**: Commit or stash any uncommitted changes
2. **Check current status**: Run existing tests to ensure they pass
3. **Review differences**: Compare your current config with standards

---

## Step 1: Update pyproject.toml

### 1.1 Pin Black Version

**Find**:
```toml
[project.optional-dependencies]
dev = [
    "black>=24.0.0",  # or any unpinned version
```

**Replace with**:
```toml
dev = [
    "black==26.1.0",  # PINNED org-wide
```

### 1.2 Add Standard Tool Configurations

Add/update these sections from `config-templates/pyproject-standards.toml`:

```toml
[tool.black]
line-length = 100
target-version = ["py312"]

[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP"]

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "--cov --cov-report=html --cov-report=xml --cov-report=term-missing"

[tool.mypy]
python_version = "3.12"
warn_return_any = true
warn_unused_configs = true

[tool.bandit]
exclude_dirs = ["tests", "docs", "_working"]
skips = ["B101"]
```

---

## Step 2: Update Pre-commit Configuration

### 2.1 Backup Current Config

```bash
cp .pre-commit-config.yaml .pre-commit-config.yaml.backup
```

### 2.2 Copy Standard Config

```bash
cp ../pipe-works/.github/config-templates/.pre-commit-config.yaml .pre-commit-config.yaml
```

### 2.3 Customize for Your Project

Edit `.pre-commit-config.yaml` and:
- Add project-specific mypy dependencies
- Enable/disable optional hooks (codespell, pytest-check)
- Adjust file exclusions if needed

### 2.4 Update Pre-commit Hooks

```bash
pre-commit install
pre-commit autoupdate
```

---

## Step 3: Reformat Code with Black 26.1.0

### 3.1 Install New Black Version

```bash
pip install black==26.1.0
```

### 3.2 Check What Will Change

```bash
black --check --diff src/ tests/
```

### 3.3 Apply Formatting

```bash
black src/ tests/
```

### 3.4 Commit Formatting Changes

```bash
git add -A
git commit -m "style: Reformat code with black 26.1.0

- Update to organization-standard black 26.1.0
- Align formatting with pipe-works standards

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

---

## Step 4: Update CI Workflow

### 4.1 Create New Workflow File

**File**: `.github/workflows/ci.yml`

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  ci:
    uses: pipe-works/.github/.github/workflows/reusable-python-ci.yml@main
    with:
      python-version: '3.12'
      coverage-threshold: 50  # Adjust for your project
      run-security: true
      run-docs: false  # Set to true if you have docs
    secrets: inherit
```

### 4.2 Customize Parameters

Adjust these based on your project:

- `python-version`: Usually '3.12'
- `coverage-threshold`: Your minimum coverage (50, 70, 80, etc.)
- `run-docs`: Set to `true` if you have Sphinx docs
- `test-os`: Add multi-OS testing if needed:
  ```yaml
  test-os: '["ubuntu-latest", "macos-latest", "windows-latest"]'
  ```

### 4.3 Remove Old Workflow (Optional)

If you're fully migrating to the reusable workflow, backup and remove your old CI file:

```bash
mv .github/workflows/ci.yml .github/workflows/ci.yml.old
# Then create the new one above
```

---

## Step 5: Add/Update Codecov Configuration

### 5.1 Copy Standard Config

```bash
cp ../pipe-works/.github/config-templates/codecov.yml codecov.yml
```

### 5.2 Customize Coverage Targets

Edit `codecov.yml`:

```yaml
coverage:
  status:
    project:
      default:
        target: 70%  # Your project's target
```

---

## Step 6: Test Everything Locally

### 6.1 Run Pre-commit

```bash
pre-commit run --all-files
```

Fix any issues that come up.

### 6.2 Run Tests

```bash
pytest -v --cov
```

Ensure all tests pass and coverage meets threshold.

### 6.3 Run Type Checking

```bash
mypy src/
```

### 6.4 Run Security Scan

```bash
bandit -r src/ -c pyproject.toml
```

---

## Step 7: Update Documentation

### 7.1 Update README.md

Add a badge section:

```markdown
[![CI](https://github.com/pipe-works/your-repo/workflows/CI/badge.svg)](https://github.com/pipe-works/your-repo/actions)
[![codecov](https://codecov.io/gh/pipe-works/your-repo/branch/main/graph/badge.svg)](https://codecov.io/gh/pipe-works/your-repo)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
```

### 7.2 Update Development Instructions

Reference organization standards:

```markdown
## Development

This project follows [pipe-works organization standards](https://github.com/pipe-works/.github/blob/main/docs/DEVELOPMENT_STANDARDS.md).

### Setup

```bash
pip install -e ".[dev]"
pre-commit install
```

### Code Quality

```bash
black src/ tests/        # Format code
ruff check src/ --fix    # Lint and auto-fix
mypy src/                # Type check
pytest                   # Run tests
```
```

---

## Step 8: Push and Verify CI

### 8.1 Commit All Changes

```bash
git add -A
git commit -m "chore: Migrate to pipe-works organization standards

- Pin black to 26.1.0
- Add standard pre-commit hooks
- Update CI to use reusable workflow
- Add codecov configuration
- Update pyproject.toml with standard tool configs

Aligns with pipe-works/.github standards.

Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>"
```

### 8.2 Push to GitHub

```bash
git push origin main  # or your branch
```

### 8.3 Verify CI Passes

1. Go to `https://github.com/pipe-works/your-repo/actions`
2. Check that the CI workflow runs
3. Verify all jobs pass (code-quality, test, security, build)
4. Check codecov report is uploaded

---

## Troubleshooting

### Black Reformatting Failures

**Issue**: Pre-commit or CI fails with "would reformat"

**Solution**:
```bash
# Ensure you have correct version
pip install black==26.1.0

# Reformat
black src/ tests/

# Commit
git add -A
git commit -m "style: Format with black 26.1.0"
```

### Mypy Import Errors

**Issue**: `ModuleNotFoundError` in mypy

**Solution**: Add missing types to `.pre-commit-config.yaml`:
```yaml
- repo: https://github.com/pre-commit/mirrors-mypy
  hooks:
  - id: mypy
    additional_dependencies: [gradio, pydantic, pytest]
```

### Coverage Threshold Failures

**Issue**: Tests fail with "coverage below threshold"

**Solution**:
- Lower threshold temporarily in CI workflow
- Add tests to increase coverage
- Adjust threshold in `pytest.ini` and CI

### Pre-commit Hook Too Slow

**Issue**: pytest hook runs slowly

**Solution**: Either:
1. Disable pytest hook in pre-commit (rely on CI)
2. Run only fast tests:
   ```yaml
   args: [tests/unit/, --tb=short, --no-cov, -q]
   ```

---

## Migration Checklist

Use this checklist to track your progress:

- [ ] Backup current configuration
- [ ] Pin black to 26.1.0 in pyproject.toml
- [ ] Add standard tool configs to pyproject.toml
- [ ] Copy and customize .pre-commit-config.yaml
- [ ] Reformat code with black 26.1.0
- [ ] Create new CI workflow using reusable workflow
- [ ] Add codecov.yml configuration
- [ ] Test pre-commit hooks locally
- [ ] Test pytest locally
- [ ] Update README.md with badges and dev instructions
- [ ] Commit all changes
- [ ] Push and verify CI passes
- [ ] Update ReadTheDocs (if applicable)

---

## Getting Help

- **Questions**: Open an issue in [pipe-works/.github](https://github.com/pipe-works/.github/issues)
- **Standards**: See [DEVELOPMENT_STANDARDS.md](./DEVELOPMENT_STANDARDS.md)
- **CI Issues**: Check reusable workflow at `.github/workflows/reusable-python-ci.yml`
