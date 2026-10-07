<!-- markdownlint-disable -->

# Hardening Report: AISquare-Studio--AISquare-Studio-QA/v0.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **AISquare-Studio--AISquare-Studio-QA/v0.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 7 `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA commit digests, making the action vulnerable to supply-chain attacks if any upstream action is compromised or a tag is moved. Failing references: `actions/cache@v5` (×2), `actions/checkout@v6`, `actions/setup-python@v6`, `actions/upload-artifact@v7` (×3). Each should be replaced with the full SHA, e.g. `actions/cache@1bd1e32a3bdc45362d1e726936510720a7c6158d # v4`.

Locations:

- `action.yml:161`
- `action.yml:167`
- `action.yml:172`
- `action.yml:175`
- `action.yml:215`
- `action.yml:222`
- `action.yml:230`

### hardcoded-credentials (severity: high)

The `staging-password` input has a hardcoded literal default value of `'password123'` (matching pattern: `password: <alphanumeric-value>`). Even though this is a default for a test credential, hardcoding a password in the action definition is a security anti-pattern — it may be committed to version control and used in real environments. It should be replaced with a reference to a secret (e.g. `${{ secrets.STAGING_PASSWORD }}`) or left empty with `required: true`.

Locations:

- `action.yml:28`

### script-injection (severity: high)

Sub-rule (a) violation: The '❌ Fail if tests failed' step directly interpolates `${{ steps.autoqa.outcome }}` inside a `run:` shell command string. The expression is embedded in shell `if [ "${{ steps.autoqa.outcome }}" = "skipped" ]` and `elif [ "${{ steps.autoqa.outcome }}" = "failure" ]`. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted into the shell script before the shell parses it. The fix is to use the env: block to pass the value as an environment variable and reference it as `"$AUTOQA_OUTCOME"` in the shell.

Locations:

- `action.yml:237`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, hardcoded-credentials, script-injection

**Notes:**

Fixed all 3 findings in hardened/action/action.yml: (1) Pinned all 7 unpinned `uses:` references to full SHA digests: actions/cache@v5 → caa296126883cff596d87d8935842f9db880ef25 (×2), actions/checkout@v6 → d23441a48e516b6c34aea4fa41551a30e30af803, actions/setup-python@v6 → ece7cb06caefa5fff74198d8649806c4678c61a1, actions/upload-artifact@v7 → cf430e030ddbb5b0abf93d22962f4752f3646cd9 (×3). (2) Replaced hardcoded `default: 'password123'` for `staging-password` input with `default: ''`. (3) Moved `${{ steps.autoqa.outcome }}` out of the `run:` shell string into an `env:` block as `AUTOQA_OUTCOME`, referencing it as `$AUTOQA_OUTCOME` in the shell script.

