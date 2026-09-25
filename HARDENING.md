<!-- markdownlint-disable -->

# Hardening Report: AISquare-Studio--AISquare-Studio-QA/v0.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **AISquare-Studio--AISquare-Studio-QA/v0.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 7 `uses:` references in action.yml are pinned to mutable version tags instead of immutable 40-character commit SHA digests. This exposes the action to supply-chain attacks if any upstream action is compromised or its tag is moved. Failing references: actions/cache@v5 (×2), actions/checkout@v6, actions/setup-python@v6, actions/upload-artifact@v7 (×3).

Locations:

- `action.yml:161`
- `action.yml:168`
- `action.yml:175`
- `action.yml:183`
- `action.yml:237`
- `action.yml:246`
- `action.yml:255`

### hardcoded-credentials (severity: high)

The `staging-password` input has a hardcoded literal default value of `'password123'`. This is a plaintext credential embedded directly in the action definition. Callers who do not override this input will silently use this hardcoded password. The value matches the pattern `(?i)password\s*[=:]\s*[A-Za-z0-9][A-Za-z0-9+/=_\-]{7,}`.

Locations:

- `action.yml:31`

### script-injection (severity: high)

Sub-rule (a): The '❌ Fail if tests failed' run: block directly interpolates `${{ steps.autoqa.outcome }}` inside shell `if` statements. Any `${{ ... }}` expression interpolated directly into a run: shell command is a script-injection risk, as the value is substituted by the template engine before the shell parses it. If the outcome value were ever attacker-influenced (e.g. via a compromised step output), it could inject arbitrary shell commands. The offending lines are: `if [ "${{ steps.autoqa.outcome }}" = "skipped" ]` and `elif [ "${{ steps.autoqa.outcome }}" = "failure" ]`. These should be replaced with an env: variable and a quoted `"$VAR"` shell reference.

Locations:

- `action.yml:268`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, hardcoded-credentials, script-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:
1. unpinned-uses: Pinned all 7 `uses:` references to full commit SHAs — actions/cache@v5 → caa296126883cff596d87d8935842f9db880ef25 (×2), actions/checkout@v6 → d23441a48e516b6c34aea4fa41551a30e30af803, actions/setup-python@v6 → ece7cb06caefa5fff74198d8649806c4678c61a1, actions/upload-artifact@v7 → 043fb46d1a93c77aae656e7c1c64a875d1fc6a0a (×3). Original tags preserved as inline comments.
2. hardcoded-credentials: Replaced the hardcoded default value 'password123' for the staging-password input with an empty string ''.
3. script-injection: Moved ${{ steps.autoqa.outcome }} out of the run: shell block into an env: block as AUTOQA_OUTCOME, and updated both shell if/elif conditions to reference $AUTOQA_OUTCOME instead.

