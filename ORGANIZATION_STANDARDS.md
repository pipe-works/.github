# Pipe-Works Organization Standards

> **Central repository for pipe-works organization-wide standards, configurations, and reusable workflows**

This repository contains the shared development standards, CI/CD workflows, and configuration templates for all pipe-works projects.

## Contents

### 📋 Documentation

- **[Development Standards](./docs/DEVELOPMENT_STANDARDS.md)** - Required tools, versions, and practices
- **[Migration Guide](./docs/MIGRATION_GUIDE.md)** - Step-by-step guide for updating existing repos

### 🔄 Reusable Workflows

Located in `.github/workflows/`:

- **`reusable-python-ci.yml`** - Comprehensive CI workflow with code quality, testing, security, and docs

### 📦 Configuration Templates

Located in `config-templates/`:

- **`.pre-commit-config.yaml`** - Standard pre-commit hooks (black 26.1.0 pinned!)
- **`pyproject-standards.toml`** - Standard pyproject.toml sections
- **`codecov.yml`** - Codecov configuration

### 🏛️ Profile

- **`profile/README.md`** - Public organization profile (displayed on GitHub)

---

## Quick Start

### For New Repositories

1. **Create your repository** on GitHub under the pipe-works organization

2. **Copy standard configurations**:
   ```bash
   # From the pipe-works/.github repo
   cp config-templates/.pre-commit-config.yaml your-repo/
   cp config-templates/codecov.yml your-repo/
   ```

3. **Add standard tool configs** to `pyproject.toml`:
   - Copy relevant sections from `config-templates/pyproject-standards.toml`
   - Minimum: `[tool.black]`, `[tool.ruff]`, `[tool.pytest.ini_options]`

4. **Set up CI workflow** in `your-repo/.github/workflows/ci.yml`:
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
         run-docs: false
       secrets: inherit
   ```

5. **Install pre-commit**:
   ```bash
   pip install pre-commit
   pre-commit install
   ```

### For Existing Repositories

Follow the **[Migration Guide](./docs/MIGRATION_GUIDE.md)** for detailed step-by-step instructions.

**Quick version**:
1. Pin black to 26.1.0 in pyproject.toml
2. Copy `.pre-commit-config.yaml` template
3. Reformat code with black 26.1.0
4. Update CI to use reusable workflow
5. Test locally, commit, push

---

## Key Standards

### Tool Versions (PINNED)

- **Black**: `26.1.0` - **MUST be pinned** to prevent formatting drift
- **Python**: `3.12+` minimum
- **Ruff**: `>=0.14.0`
- **Mypy**: `>=1.19.0`

### Why Pin Black?

Black's formatting rules can change between versions. Pinning ensures:
- ✅ All developers use the same formatting
- ✅ CI and local environments match
- ✅ No spurious "would reformat" errors
- ✅ No git diffs from formatting changes

### Code Quality Checklist

Every pipe-works repository should have:

- [x] Black formatted code (100 char line length)
- [x] Ruff linting with standard rules
- [x] Mypy type checking (lenient initially)
- [x] Pytest with >=50% coverage
- [x] Pre-commit hooks installed
- [x] Codecov integration
- [x] GPL-3.0 license

---

## Reusable CI Workflow

### Features

The `reusable-python-ci.yml` workflow provides:

✅ **Code Quality**: Ruff, Black, Mypy
✅ **Testing**: Pytest with coverage reporting
✅ **Security**: Bandit + Trivy vulnerability scanning
✅ **Documentation**: Sphinx build validation (optional)
✅ **Package Build**: Validates package can be built and published
✅ **Multi-OS**: Support for Ubuntu, macOS, Windows (optional)
✅ **Multi-Python**: Test against multiple Python versions (optional)

### Parameters

```yaml
with:
  python-version: '3.12'              # Primary Python version
  additional-python-versions: '[]'    # e.g. '["3.13"]'
  coverage-threshold: 50              # Minimum coverage %
  run-security: true                  # Run Bandit + Trivy
  run-docs: false                     # Build Sphinx docs
  test-os: '["ubuntu-latest"]'        # OS matrix
```

### Example: Multi-OS, Multi-Python

```yaml
jobs:
  ci:
    uses: pipe-works/.github/.github/workflows/reusable-python-ci.yml@main
    with:
      python-version: '3.12'
      additional-python-versions: '["3.13"]'
      test-os: '["ubuntu-latest", "macos-latest", "windows-latest"]'
      coverage-threshold: 70
      run-docs: true
    secrets: inherit
```

---

## Configuration Templates

### .pre-commit-config.yaml

**Features**:
- Black 26.1.0 (pinned)
- Ruff linting
- File checks (EOF, trailing whitespace, YAML, TOML, JSON)
- Python checks (AST, builtins, docstrings, debug statements)
- Mypy type checking
- Bandit security scanning
- Markdownlint
- Codespell spell checking
- Optional pytest hook

**Copy from**: `config-templates/.pre-commit-config.yaml`

### pyproject-standards.toml

**Includes**:
- Development dependencies (pytest, black, ruff, mypy, bandit)
- Black configuration (100 char line length)
- Ruff configuration (standard rules)
- Pytest configuration (coverage, async support)
- Mypy configuration (lenient initial setup)
- Bandit security configuration

**Copy relevant sections to your** `pyproject.toml`

### codecov.yml

**Features**:
- Project coverage target (50% default)
- Patch coverage target (70% for new code)
- PR comments with diff
- Ignore paths (tests, docs, examples)

**Copy from**: `config-templates/codecov.yml`

---

## Organization Secrets

The following secrets are configured at the organization level and automatically available to all repos:

- **`CODECOV_TOKEN`** - For uploading coverage reports to Codecov

No need to configure these per-repository!

---

## Development Workflow

### Setting Up a New Project

```bash
# 1. Clone your new repo
git clone git@github.com:pipe-works/new-repo.git
cd new-repo

# 2. Copy configs from .github
cp ../pipe-works/.github/config-templates/.pre-commit-config.yaml .
cp ../pipe-works/.github/config-templates/codecov.yml .

# 3. Create pyproject.toml with standard configs
# (copy sections from config-templates/pyproject-standards.toml)

# 4. Install dev dependencies
pip install -e ".[dev]"
pre-commit install

# 5. Start developing!
```

### Before Committing

```bash
# Format code
black src/ tests/

# Lint code
ruff check src/ --fix

# Type check
mypy src/

# Run tests
pytest

# Or let pre-commit do it all:
pre-commit run --all-files
```

---

## Updating Standards

### How to Update Organization Standards

1. **Update this repository** (`.github`):
   - Modify reusable workflows
   - Update configuration templates
   - Update documentation

2. **Test changes** on a single repo first

3. **Announce changes** in organization discussions

4. **Update repositories** gradually using migration guide

### Version Bumping Process

When updating pinned versions (e.g., black):

1. Test new version on pipeworks_name_generation (our reference repo)
2. Update `.github/config-templates/.pre-commit-config.yaml`
3. Update `.github/config-templates/pyproject-standards.toml`
4. Update `DEVELOPMENT_STANDARDS.md`
5. Create migration PR for each active repo

---

## Contributing

### Proposing Standard Changes

1. Open an issue in this repository
2. Discuss rationale and impact
3. Create PR with proposed changes
4. Test on at least one repo
5. Document migration path if breaking change

### Guidelines

- **Breaking changes require migration guide updates**
- **Test on multiple repos before merging**
- **Update documentation alongside code changes**
- **Version bumps need clear changelog**

---

## Repository Status

### Repos Using These Standards

- ✅ **pipeworks_name_generation** - Reference implementation (complete)
- ✅ **pipeworks-image-generator** - Migrated (complete)
- ✅ **pipeworks_entity_state_generation** - Migrated (complete)
- ✅ **pipeworks_mud_server** - Migrated (complete)
- ⏳ **pipe-works-org** - N/A (static HTML/CSS, no Python)

---

## Support

- **Questions**: Open an issue in this repository
- **Standards discussion**: Use GitHub Discussions
- **Urgent issues**: Tag `@aa-parky` in issue

---

## License

This repository and all templates are licensed under GPL-3.0-or-later, consistent with all pipe-works projects.
