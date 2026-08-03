<!-- markdownlint-disable -->

# Hardening Report: AISquare-Studio--AISquare-Studio-QA/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **AISquare-Studio--AISquare-Studio-QA/v0.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

The `staging-password` input in action.yml has a hardcoded literal default value of `password123`. This is a hardcoded credential matching the pattern `password: <literal-value>`. Even as a default, embedding a real-looking password in the action definition is a security risk.

Locations:

- `action.yml:30`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside `run:` shell command strings in multiple places:

1. action.yml — "Fail if tests failed" step: `${{ steps.autoqa.outcome }}` is interpolated directly inside a bash `run:` block (e.g., `if [ "${{ steps.autoqa.outcome }}" = "skipped" ]`). A malicious step output could inject shell commands.

2. release.yml — "Extract changelog for this version" step: `${{ steps.version.outputs.version }}` and `${{ steps.version.outputs.tag }}` are interpolated directly inside a `run:` block (e.g., `VERSION="${{ steps.version.outputs.version }}"`; `NOTES="Release ${{ steps.version.outputs.tag }}"`). Although these come from a prior step that parses a git tag, any expression in a run: block is a script-injection risk.

3. release.yml — "Update major version tag" step: `${{ steps.version.outputs.major }}` and `${{ steps.version.outputs.tag }}` are interpolated directly inside a `run:` block (e.g., `MAJOR="v${{ steps.version.outputs.major }}"`; `git tag -fa "$MAJOR" -m "Update $MAJOR tag to ${{ steps.version.outputs.tag }}"`). These should be passed via env vars and referenced as shell variables.

Locations:

- `action.yml:261`
- `.github/workflows/release.yml:107`
- `.github/workflows/release.yml:122`
- `.github/workflows/release.yml:140`

### unpinned-uses (severity: high)

Multiple `uses:` references across action.yml and workflow files are pinned to mutable version tags rather than immutable 40-character commit SHAs. This exposes the action to supply-chain attacks if any upstream action is compromised or its tag is moved.

Failing references:
- action.yml: `actions/cache@v5`, `actions/checkout@v6`, `actions/setup-python@v6`, `actions/upload-artifact@v7` (×3)
- .github/workflows/lint.yml: `actions/checkout@v6`, `actions/setup-python@v6`, `stefanzweifel/git-auto-commit-action@v7`
- .github/workflows/release.yml: `actions/checkout@v6` (×4), `actions/setup-python@v6` (×2), `softprops/action-gh-release@v3`
- .github/workflows/test-action.yml: `actions/checkout@v6` (×3), `actions/setup-python@v6`, `actions/cache@v5` (×2)

Locations:

- `action.yml:175`
- `action.yml:183`
- `action.yml:196`
- `action.yml:218`
- `action.yml:247`
- `action.yml:251`
- `action.yml:255`
- `.github/workflows/lint.yml:20`
- `.github/workflows/lint.yml:23`
- `.github/workflows/lint.yml:44`
- `.github/workflows/release.yml:15`
- `.github/workflows/release.yml:40`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:68`
- `.github/workflows/release.yml:83`
- `.github/workflows/release.yml:96`
- `.github/workflows/release.yml:128`
- `.github/workflows/test-action.yml:15`
- `.github/workflows/test-action.yml:20`
- `.github/workflows/test-action.yml:27`
- `.github/workflows/test-action.yml:113`
- `.github/workflows/test-action.yml:120`
- `.github/workflows/test-action.yml:152`

### missing-permissions (severity: medium)

The workflow file `.github/workflows/test-action.yml` has no top-level `permissions:` key, and none of its three jobs (`test-action-components`, `test-self-autoqa`, `validate-action-yml`) define a job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents, packages, etc.).

Locations:

- `.github/workflows/test-action.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** hardcoded-credentials, script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings: (1) Removed hardcoded 'password123' default from staging-password input in action.yml. (2) Fixed script injection in action.yml 'Fail if tests failed' step and release.yml 'Extract changelog' and 'Update major version tag' steps by moving ${{ }} expressions into env: blocks and referencing them as shell variables. (3) Pinned all unpinned action references to full commit SHAs across action.yml, .github/workflows/lint.yml, .github/workflows/release.yml, and .github/workflows/test-action.yml. (4) Added top-level 'permissions: contents: read' block to .github/workflows/test-action.yml.

