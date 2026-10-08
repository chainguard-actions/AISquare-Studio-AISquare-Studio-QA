<!-- markdownlint-disable -->

# Hardening Report: AISquare-Studio--AISquare-Studio-QA/v0.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **AISquare-Studio--AISquare-Studio-QA/v0.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

The `staging-password` input has a hardcoded literal default value of `password123`. This is a real credential embedded in the action definition, not a secrets expression. Any workflow that uses this action without explicitly overriding `staging-password` will use this hardcoded password. The value matches the pattern `password: <alphanumeric-8+>` and is not a `${{ secrets.* }}` reference.

Locations:

- `action.yml:30`

### unpinned-uses (severity: high)

All 7 `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if any of these upstream actions are compromised or their tags are moved. Failing references:
- `uses: actions/cache@v5` (Cache AutoQA Repository step)
- `uses: actions/checkout@v6` (Checkout AutoQA Action Repository step)
- `uses: actions/setup-python@v6` (Setup Python Environment step)
- `uses: actions/cache@v5` (Cache Playwright Browsers step)
- `uses: actions/upload-artifact@v7` (Upload Screenshots step)
- `uses: actions/upload-artifact@v7` (Upload Test Reports step)
- `uses: actions/upload-artifact@v7` (Upload Dashboard Results step)

Locations:

- `action.yml:155`
- `action.yml:161`
- `action.yml:169`
- `action.yml:175`
- `action.yml:220`
- `action.yml:231`
- `action.yml:242`

### script-injection (severity: high)

Sub-rule (a) violation: The '❌ Fail if tests failed' step's `run:` block directly interpolates `${{ steps.autoqa.outcome }}` inside shell `if` comparisons. Any `${{ ... }}` expression interpolated directly into a `run:` shell script is a script-injection risk because the value is substituted into the shell command string before the shell parses it. Offending lines:
  `if [ "${{ steps.autoqa.outcome }}" = "skipped" ]; then`
  `elif [ "${{ steps.autoqa.outcome }}" = "failure" ]; then`
These should be replaced with an env-var reference, e.g. set `AUTOQA_OUTCOME: ${{ steps.autoqa.outcome }}` in an `env:` block and use `"$AUTOQA_OUTCOME"` in the shell script.

Locations:

- `action.yml:255`
- `action.yml:257`

## Iteration Notes

### Iteration 1

**Fixes applied:** hardcoded-credentials, unpinned-uses, script-injection

**Notes:**

1. hardcoded-credentials: Removed the hardcoded 'password123' default value from the staging-password input (line 30). The input is now optional with no default, requiring callers to explicitly provide a password via secrets.
2. unpinned-uses: Pinned all 7 action references to full 40-character SHA hashes: actions/cache@v5 → caa296126883cff596d87d8935842f9db880ef25 (used twice), actions/checkout@v6 → d23441a48e516b6c34aea4fa41551a30e30af803, actions/setup-python@v6 → ece7cb06caefa5fff74198d8649806c4678c61a1, actions/upload-artifact@v7 → cf430e030ddbb5b0abf93d22962f4752f3646cd9 (used three times). Version tags preserved as comments.
3. script-injection: Added an env block with AUTOQA_OUTCOME: ${{ steps.autoqa.outcome }} to the 'Fail if tests failed' step, and replaced both inline ${{ steps.autoqa.outcome }} interpolations in the shell script with $AUTOQA_OUTCOME references.

