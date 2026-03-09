# Agent Foundation Rules

This is the canonical cross-repository foundation for agent behavior in the
pipe-works organization.

All repository-level `AGENTS.md` files must begin by referencing this document
before project-specific instructions.

## Mandatory Foundation Rules

1. Preserve CI integrity: optimize runtime without reducing assurance.
2. Keep required checks stable:
   - `All Checks Passed`
   - `Secret Scan (Gitleaks)`
3. Treat secret scanning as a hard gate in CI and local pre-commit.
4. Follow the test-tagging contract exactly.
5. Use explicit evidence (run IDs, durations, check statuses) for CI changes.

## Required Companion Document

Before GitHub/CI/test-tag work, read:

- `TEST_TAGGING_AND_GITHUB_CHECKLIST.md`

Canonical links:

- https://github.com/pipe-works/.github/blob/main/.github/docs/AGENT_FOUNDATION.md
- https://github.com/pipe-works/.github/blob/main/.github/docs/TEST_TAGGING_AND_GITHUB_CHECKLIST.md

## Standard Repository AGENTS Structure

Repository `AGENTS.md` files should follow this order:

1. `Foundation Must-Dos (Org-Wide)` with links to the canonical docs.
2. Repository-specific architecture and operating constraints.
3. Repository-specific commands, testing, and delivery expectations.

## Weekly CI Sweep Baseline

Maintained code repositories should include a weekly scheduled CI run in
`.github/workflows/ci.yml` to validate full assurance paths outside PR traffic.

Recommended trigger:

```yaml
on:
  schedule:
    - cron: '17 5 * * 1'  # Weekly Monday full sweep (UTC)
```

## AGENTS Standardization Rollout Tracker

Status snapshot as of 2026-03-09.

| Repository | Standardized Foundation Header | Notes |
|---|---|---|
| `pipeworks_mud_server` | yes | pilot updated |
| `pipeworks_entity_state_generation` | yes | pilot updated |
| `pipeworks_axis_descriptor_lab` | yes | pilot updated |
| `pipeworks_name_generation` | no | pending rollout |
| `pipeworks-image-generator` | no | pending rollout |
| `pipeworks_ipc` | no | pending rollout |
| `pipeworks_mud_mapper` | no | pending rollout |
| `pipeworks-dev-notes` | no | pending rollout |
| `pipe-works-org` | no | pending rollout (non-Python repo) |
| `pipe-works` | no | pending rollout (meta/docs repo) |
