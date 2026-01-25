# Pipe-Works Development Standards

> **Organization-wide standards for all pipe-works repositories**

This document defines the development standards, tooling, and best practices for all projects in the pipe-works organization.

## Table of Contents

- [Python Version](#python-version)
- [Code Quality Tools](#code-quality-tools)
- [Testing Standards](#testing-standards)
  - [ML/Model Testing](#mlmodel-testing)
- [CI/CD](#cicd)
- [Pre-commit Hooks](#pre-commit-hooks)
- [Documentation](#documentation)
- [Versioning](#versioning)
- [License](#license)

---

## Python Version

**Required**: Python 3.12+

All pipe-works projects target Python 3.12 as the minimum version. This allows us to use:
- Modern type hints (`|` union syntax, `type` statement)
- Performance improvements
- Latest standard library features

---

## Code Quality Tools

### Black (Code Formatter)

**Version**: `26.1.0` (PINNED)

**Why pinned?** Black formatting can change between versions. Pinning ensures all developers and CI use the same formatting rules, preventing spurious diffs.

**Configuration**:
```toml
[tool.black]
line-length = 100
target-version = ["py312"]
```

**Usage**:
```bash
black src/ tests/
```

### Ruff (Linter)

**Version**: `>=0.14.0`

Ruff combines the functionality of flake8, isort, pylint, and more into one fast tool.

**Configuration**:
```toml
[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "UP"]
```

**Usage**:
```bash
ruff check src/ tests/
ruff check src/ --fix  # Auto-fix issues
```

### Mypy (Type Checker)

**Version**: `>=1.19.0`

**Configuration**:
```toml
[tool.mypy]
python_version = "3.12"
warn_return_any = true
warn_unused_configs = true
disallow_untyped_defs = false  # Start lenient
```

**Usage**:
```bash
mypy src/
```

### Bandit (Security Scanner)

**Version**: `>=1.7.0`

Scans code for common security issues.

**Configuration**:
```toml
[tool.bandit]
exclude_dirs = ["tests", "docs", "_working"]
skips = ["B101"]
```

---

## Testing Standards

### Pytest

**Version**: `>=8.0.0`

**Required plugins**:
- `pytest-cov>=4.1.0` - Coverage reporting
- `pytest-asyncio>=0.23.0` - If using async code

**Configuration**:
```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = """
    --cov
    --cov-report=html
    --cov-report=xml
    --cov-report=term-missing
    --cov-fail-under=50
"""
```

**Coverage Targets**:
- **Minimum**: 50% for overall project
- **Goal**: 70%+ for new code
- **Core logic**: 90%+ recommended

### Test Structure

```
tests/
├── unit/              # Fast, isolated tests
├── integration/       # Tests with dependencies
├── conftest.py        # Shared fixtures
└── README.md          # Test documentation
```

### ML/Model Testing

For projects using machine learning models (PyTorch, Diffusers, Transformers), special considerations apply:

#### Test Markers

Use pytest markers to categorize tests by resource requirements:

```toml
[tool.pytest.ini_options]
markers = [
    "slow: Tests that take >5 seconds (deselect with '-m \"not slow\"')",
    "requires_model: Tests requiring ML model downloads (deselect with '-m \"not requires_model\"')",
    "integration: Integration tests requiring external resources",
    "unit: Fast unit tests (default)",
]
```

**Usage**:

```python
import pytest

@pytest.mark.requires_model
def test_model_inference():
    # Test that requires downloading a model
    pass

@pytest.mark.slow
def test_large_batch_processing():
    # Test that takes >5 seconds
    pass

# Fast tests (no marker needed)
def test_validation_logic():
    # Quick unit test
    pass
```

#### CI Configuration for ML Projects

Use the reusable workflow with ML-specific parameters:

```yaml
jobs:
  ci:
    uses: pipe-works/.github/.github/workflows/reusable-python-ci.yml@main
    with:
      python-version: '3.12'
      coverage-threshold: 50
      requires-models: true                           # Enable disk cleanup
      pytest-markers: 'not requires_model and not slow'  # Skip expensive tests
    secrets: inherit
```

**What `requires-models: true` does**:

- Frees up ~10GB disk space (removes dotnet, ghc, boost, agent tools)
- Uses `pip install --no-cache-dir` to save space
- Sets `HF_HUB_OFFLINE=1` to prevent accidental model downloads
- Sets `PIPEWORKS_MODELS_DIR=/tmp/models` for isolation

#### Local Development

**Install test dependencies**:
```bash
pip install -e ".[dev]"
```

**Run fast tests only** (pre-commit):
```bash
pytest -m "not requires_model and not slow"
```

**Run all tests** (before PR):
```bash
pytest  # Runs everything including model tests
```

**Run specific marker**:
```bash
pytest -m requires_model  # Only model tests
pytest -m slow            # Only slow tests
```

#### Best Practices

1. **Keep model downloads optional**: Tests should work without internet access
2. **Use small test models**: If testing model code, use tiny/test model variants
3. **Mock model outputs**: For testing logic around models, mock the inference
4. **Separate concerns**:
   - `unit/` - Logic tests (fast, no models)
   - `integration/` - Model integration tests (slow, requires downloads)

**Example test organization**:

```
tests/
├── unit/
│   ├── test_validation.py      # No marker (fast)
│   ├── test_config.py          # No marker (fast)
│   └── test_prompts.py         # No marker (fast)
├── integration/
│   ├── test_model_loading.py   # @pytest.mark.requires_model
│   └── test_inference.py       # @pytest.mark.requires_model @pytest.mark.slow
└── conftest.py
```

#### GitHub Actions Disk Space

ML dependencies (PyTorch, Diffusers) are large. Standard GitHub runners have ~14GB free space. After removing unnecessary software, you gain ~10GB more.

**If tests still fail due to disk space**:

1. Check if all model downloads are marked with `@pytest.mark.requires_model`
2. Verify `HF_HUB_OFFLINE=1` is preventing downloads
3. Consider using `ubuntu-latest-large` runners (requires paid plan)
4. Use `--no-cache-dir` for pip installs in CI

---

## CI/CD

### Reusable Workflow

All repos should use the organization's reusable CI workflow:

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
      coverage-threshold: 50
      run-security: true
      run-docs: true  # Set to true if you have docs
    secrets: inherit
```

### CI Jobs

The reusable workflow includes:

1. **Code Quality** - Ruff, Black, Mypy
2. **Tests** - Pytest with coverage
3. **Security** - Bandit, Trivy
4. **Documentation** - Sphinx build (optional)
5. **Package Build** - Validate package can be built

### Required Secrets

- `CODECOV_TOKEN` - Organization-level secret for coverage reporting

---

## Pre-commit Hooks

### Installation

```bash
pip install pre-commit
pre-commit install
```

### Standard Configuration

Copy from: `pipe-works/.github/config-templates/.pre-commit-config.yaml`

**Included hooks**:
- Ruff (linting)
- Black (formatting) - **Version 26.1.0 pinned**
- File checks (trailing whitespace, EOF, YAML, etc.)
- Mypy (type checking)
- Bandit (security)
- Markdownlint (markdown formatting)
- Codespell (spell checking)

### Running Manually

```bash
pre-commit run --all-files
```

---

## Documentation

### Sphinx + ReadTheDocs

**Recommended setup**:
- Sphinx for API docs
- sphinx-rtd-theme for styling
- MyST parser for Markdown support
- Auto-deployment via ReadTheDocs

**Configuration**: See individual repo's `docs/` directory

### Docstring Style

Use **Google-style** docstrings:

```python
def example_function(param1: str, param2: int) -> bool:
    """Short description of the function.

    Longer description with more details about what the function does,
    edge cases, and usage examples.

    Args:
        param1: Description of param1
        param2: Description of param2

    Returns:
        Description of return value

    Raises:
        ValueError: When param2 is negative
    """
    pass
```

---

## Versioning

### Semantic Versioning

All projects follow [Semantic Versioning 2.0.0](https://semver.org/):

- **MAJOR**: Breaking changes
- **MINOR**: New features (backward compatible)
- **PATCH**: Bug fixes (backward compatible)

### Release Please

Projects using automated releases should use `release-please`:

```yaml
# .github/workflows/release-please.yml
name: Release Please

on:
  push:
    branches: [main]

jobs:
  release-please:
    uses: google-github-actions/release-please-action@v4
    with:
      release-type: python
```

### Conventional Commits

**Required** for release-please:

```
feat: Add new feature
fix: Fix bug in component
docs: Update README
chore: Update dependencies
refactor: Restructure module
test: Add tests for feature
```

---

## License

**Required**: GPL-3.0-or-later

All pipe-works projects are licensed under GPL-3.0-or-later.

**File**: `LICENSE` (copy from organization template)

**Headers**: Add to all source files:

```python
# Copyright (C) 2025 pipe-works
# This file is part of [project-name].
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
```

---

## Quick Start Checklist

Setting up a new pipe-works repository:

- [ ] Python 3.12+ in `requires-python`
- [ ] Copy `.pre-commit-config.yaml` from config-templates
- [ ] Copy relevant sections from `pyproject-standards.toml`
- [ ] Copy `codecov.yml` from config-templates
- [ ] Set up CI workflow using reusable workflow
- [ ] Add `CODECOV_TOKEN` secret (already set at org level)
- [ ] Create `tests/` directory with pytest
- [ ] Add GPL-3.0 LICENSE file
- [ ] Add CONTRIBUTING.md
- [ ] Configure ReadTheDocs (optional)

---

## Questions?

- **Standards issues**: Open an issue in [pipe-works/.github](https://github.com/pipe-works/.github/issues)
- **Tool-specific help**: Check individual tool documentation
- **Migration help**: See [MIGRATION_GUIDE.md](./MIGRATION_GUIDE.md)
