# Test Tagging and GitHub Checklist

This document defines the organization-wide tagging contract for pytest suites
and the mandatory preflight checklist for GitHub-facing work.

## Scope

Applies to all Python repositories using the reusable workflow:
`pipe-works/.github/.github/workflows/reusable-python-ci.yml`.

## Test Tagging Contract

Use only registered markers. Marker misuse causes silent CI drift and invalid
runtime assumptions.

### Marker definitions

- `unit`: isolated, deterministic tests with mocked dependencies.
- `integration`: tests that require real integration surfaces (filesystem, DB,
  service boundaries).
- `slow`: tests that are materially slower than normal unit/integration tests.
- `requires_model`: tests that require model files, heavyweight assets, or
  model-adjacent runtime dependencies.

Repo-specific markers (for example `api`, `db`, `auth`, `game`, `security`) are
allowed for domain grouping and should not replace the core markers above.

### Tagging rules

1. Tag tests by execution characteristics, not by preference.
2. If a test is both slow and model-bound, apply both markers.
3. If a test is intentionally excluded from fast lanes, document why in the
   test module or surrounding docs.
4. Do not remove `slow` or `requires_model` markers solely to improve CI time.
5. Keep marker registration in pytest config synchronized with actual usage.

## CI Lane Mapping

Default lane mapping in pilot repositories:

- Matrix lane: `not slow and not requires_model`
- Coverage lane: `not slow and not requires_model`
- Supplemental lane: `slow or requires_model` (non-coverage)
- Weekly full sweep: no marker filtering (runs complete assurance path)

Any lane contract changes must be reviewed as CI-policy changes, not routine
test edits.

## Weekly Full Sweep Policy

Each maintained code repository should run a scheduled weekly CI sweep that:

1. Executes full CI (code quality, matrix tests, coverage, security, gitleaks,
   docs/build where configured).
2. Is independent of pull-request content-classification shortcuts.
3. Produces auditable run evidence in GitHub Actions history.

## Mandatory GitHub Preflight Checklist

Before running `gh` commands, editing CI workflows, or changing test tags:

1. Read this document and `AGENT_FOUNDATION.md`.
2. Confirm target repository, active branch, and PR/base branch.
3. Confirm current check contract (`All Checks Passed`, `Secret Scan
   (Gitleaks)`) for the target repo.
4. Identify whether your change affects:
   - marker semantics
   - CI lane routing
   - branch protection expectations
5. Plan evidence collection before merging:
   - relevant `gh run view` / `gh pr checks` output
   - before/after runtime comparison for CI timing work
6. If changing tagging or CI routing, update repo `AGENTS.md` and docs in the
   same PR.

## Commit Tag Guidance for CI/Tagging Work

- Use `ci:` for workflow behavior changes.
- Use `test:` for marker/test-only changes.
- Use `docs:` for documentation-only contract clarifications.
- Use `feat:` only for actual new user-facing capability.

