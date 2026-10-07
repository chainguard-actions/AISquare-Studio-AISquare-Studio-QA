<!-- markdownlint-disable -->

# Hardening Report: AISquare-Studio--AISquare-Studio-QA/v0.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **AISquare-Studio--AISquare-Studio-QA/v0.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 6 `uses:` references in action.yml use mutable version tags instead of pinned 40-character commit SHA hashes. This exposes the action to supply-chain attacks where a tag could be silently moved to point to malicious code. Failing references: `actions/cache@v5` (×2), `actions/checkout@v6`, `actions/setup-python@v6`, `actions/upload-artifact@v7` (×2). Each should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:88`
- `action.yml:94`
- `action.yml:101`
- `action.yml:108`
- `action.yml:163`
- `action.yml:173`

### hardcoded-credentials (severity: high)

The `staging-password` input has a hardcoded literal default value of `password123`. This is a plaintext password embedded directly in the action definition. Even as a default/example value, hardcoded passwords are a security risk — they may be used in real environments and are visible to anyone who reads the action source. The value should be removed or replaced with a reference to a secret (e.g. `${{ secrets.STAGING_PASSWORD }}`).

Locations:

- `action.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, hardcoded-credentials

**Notes:**

Fixed all 6 unpinned `uses:` references by pinning them to their full 40-character commit SHAs (with version tag comments for readability): actions/cache@v5→caa2961..., actions/checkout@v6→d23441a..., actions/setup-python@v6→ece7cb0..., actions/upload-artifact@v7→cf430e0.... Removed the hardcoded default password 'password123' from the staging-password input definition.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all four findings in examples/fe-react-autoqa-workflow.yml:

1. missing-permissions: Added `permissions: {}` at top level; `contents: read, pull-requests: read` on `autoqa` job; `pull-requests: read` on `validate-autoqa-format` job.

2. unpinned-uses: Pinned all three actions to full commit SHAs — actions/checkout@11d5960a326750d5838078e36cf38b85af677262 (v4), AISquare-Studio/AISquare-Studio-QA@71db4eea23e14f684b5c91c2dbbc9d5145510b48 (main), actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 (v4).

3. script-injection: Moved all ${{ }} expressions from run: shell blocks into env: blocks. In 'Get PR Body': EVENT_NAME, REPOSITORY, PR_NUMBER, PR_BODY_RAW env vars replace inline expressions. In 'Check AutoQA Format': PR_BODY env var replaces inline expression.

4. github-env-injection: PR_BODY is now sanitized with `printf '%s' "$PR_BODY" | tr -d '\r'` before being written to GITHUB_OUTPUT, preventing newline injection attacks.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed the _set_outputs method in src/autoqa/action_runner.py to sanitize values before writing to $GITHUB_OUTPUT. The original code wrote attacker-controlled values (parsed from PR_BODY, which comes from github.event.pull_request.body) directly to GITHUB_OUTPUT without stripping newlines, allowing injection of arbitrary key=value pairs. The fix converts each value to a string, then strips all \r and \n characters before writing, preventing newline injection. The multiline heredoc path was also removed since all values are now sanitized to be single-line safe.

