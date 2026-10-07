<!-- markdownlint-disable -->

# Hardening Report: AISquare-Studio--AISquare-Studio-QA/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **AISquare-Studio--AISquare-Studio-QA/v0.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 7 `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA digests, exposing the action to supply-chain attacks if any upstream action is compromised or a tag is moved. Failing references:
- `actions/cache@v5` (Cache AutoQA Repository step)
- `actions/checkout@v6` (Checkout AutoQA Action Repository step)
- `actions/setup-python@v6` (Setup Python Environment step)
- `actions/cache@v5` (Cache Playwright Browsers step)
- `actions/upload-artifact@v7` (Upload Screenshots as Artifacts step)
- `actions/upload-artifact@v7` (Upload Test Reports step)
- `actions/upload-artifact@v7` (Upload Dashboard Results step)

Locations:

- `action.yml:196`
- `action.yml:202`
- `action.yml:210`
- `action.yml:228`
- `action.yml:263`
- `action.yml:274`
- `action.yml:285`

### script-injection (severity: high)

Rule (a) violation: The 'Fail if tests failed' run: block directly interpolates `${{ steps.autoqa.outcome }}` inside a shell `if` statement. Any `${{ ... }}` expression interpolated directly into a `run:` shell command string is a script-injection risk — the YAML template substitution happens before the shell ever sees the value. Offending lines:
  `if [ "${{ steps.autoqa.outcome }}" = "skipped" ]; then`
  `elif [ "${{ steps.autoqa.outcome }}" = "failure" ]; then`
These should be moved to an `env:` block and referenced as `"$AUTOQA_OUTCOME"` in the shell script.

Locations:

- `action.yml:296`

### hardcoded-credentials (severity: high)

The `staging-password` input has a hardcoded literal default value `password123` — a non-expression literal string assigned to a field whose name contains `password`. Even though this is a default for a test credential, hardcoding it in the action definition is a security anti-pattern: it may be inadvertently used against real staging environments and is visible to anyone reading the action source. The value should be left empty (no default) and required to be supplied explicitly by the caller via a secret. Additionally, `staging-email` has a hardcoded default `test@example.com`.

Locations:

- `action.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, hardcoded-credentials

**Notes:**

Fixed all three findings in hardened/action/action.yml:

1. **unpinned-uses**: Pinned all 7 `uses:` references to full 40-character SHA digests:
   - `actions/cache@v5` → `actions/cache@caa296126883cff596d87d8935842f9db880ef25 # v5` (2 occurrences)
   - `actions/checkout@v6` → `actions/checkout@d23441a48e516b6c34aea4fa41551a30e30af803 # v6`
   - `actions/setup-python@v6` → `actions/setup-python@ece7cb06caefa5fff74198d8649806c4678c61a1 # v6`
   - `actions/upload-artifact@v7` → `actions/upload-artifact@cf430e030ddbb5b0abf93d22962f4752f3646cd9 # v7` (3 occurrences)

2. **script-injection**: Moved `${{ steps.autoqa.outcome }}` out of the `run:` shell script into an `env:` block as `AUTOQA_OUTCOME`, and updated the shell conditionals to reference `$AUTOQA_OUTCOME` instead.

3. **hardcoded-credentials**: Removed the hardcoded default values `'password123'` for `staging-password` and `'test@example.com'` for `staging-email`. Both inputs are now optional with no default, requiring callers to supply credentials explicitly.

