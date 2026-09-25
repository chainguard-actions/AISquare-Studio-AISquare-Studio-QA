<!-- markdownlint-disable -->

# Hardening Report: AISquare-Studio--AISquare-Studio-QA/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **AISquare-Studio--AISquare-Studio-QA/v0.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 7 `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA commit digests, creating a supply-chain attack risk. Failing references: `actions/cache@v5` (two occurrences), `actions/checkout@v6`, `actions/setup-python@v6`, `actions/upload-artifact@v7` (three occurrences). Each should be replaced with the full SHA, e.g. `actions/checkout@<40-hex-sha> # v6`.

Locations:

- `action.yml:163`
- `action.yml:170`
- `action.yml:177`
- `action.yml:208`
- `action.yml:252`
- `action.yml:261`
- `action.yml:271`

### hardcoded-credentials (severity: high)

The `staging-password` input has a hardcoded literal default value of `password123`. This is a hardcoded credential (matches pattern: `password: password123`). It should be removed or replaced with a reference to a secret expression such as `${{ secrets.STAGING_PASSWORD }}`.

Locations:

- `action.yml:36`

### script-injection (severity: high)

Sub-rule (a): The '❌ Fail if tests failed' `run:` block directly interpolates `${{ steps.autoqa.outcome }}` inside shell `if` comparisons. Any `${{ ... }}` expression interpolated directly into a `run:` shell command string is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it. Offending lines: `if [ "${{ steps.autoqa.outcome }}" = "skipped" ]` and `elif [ "${{ steps.autoqa.outcome }}" = "failure" ]`. Fix: use the environment variable `$GITHUB_ACTION_OUTCOME` or set an `env:` variable and reference it as a shell variable.

Locations:

- `action.yml:281`
- `action.yml:283`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, hardcoded-credentials, script-injection

**Notes:**

Fixed all 3 findings in hardened/action/action.yml:
1. unpinned-uses: Pinned all 7 `uses:` references to full SHA digests — actions/cache@caa296126883cff596d87d8935842f9db880ef25 (×2), actions/checkout@d23441a48e516b6c34aea4fa41551a30e30af803, actions/setup-python@ece7cb06caefa5fff74198d8649806c4678c61a1, actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a (×3). Version tags preserved as inline comments.
2. hardcoded-credentials: Removed the hardcoded `default: 'password123'` from the staging-password input definition.
3. script-injection: Moved `${{ steps.autoqa.outcome }}` into an `env:` block as `AUTOQA_OUTCOME` in the '❌ Fail if tests failed' step, and updated both shell `if` comparisons to reference `$AUTOQA_OUTCOME` instead.

