<!-- markdownlint-disable -->

# Hardening Report: AISquare-Studio--AISquare-Studio-QA/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **AISquare-Studio--AISquare-Studio-QA/v0.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The '❌ Fail if tests failed' run: block directly interpolates ${{ steps.autoqa.outcome }} inside shell command strings (sub-rule a). Although steps.*.outcome is GitHub-controlled, any ${{ ... }} expression interpolated directly into a run: shell script is a script-injection finding — the value is substituted into the shell command before the shell parses it. Offending lines:
  if [ "${{ steps.autoqa.outcome }}" = "skipped" ]; then
  elif [ "${{ steps.autoqa.outcome }}" = "failure" ]; then
These should be moved to an env: variable and referenced as "$STEP_OUTCOME" instead.

Locations:

- `action.yml:271`
- `action.yml:273`

### unpinned-uses (severity: high)

All 7 uses: references in action.yml use mutable version tags instead of pinned 40-character SHA commit digests, making the action vulnerable to supply-chain attacks if any of these actions are compromised or their tags are moved:
- actions/cache@v5 (Cache AutoQA Repository step)
- actions/checkout@v6 (Checkout AutoQA Action Repository step)
- actions/setup-python@v6 (Setup Python Environment step)
- actions/cache@v5 (Cache Playwright Browsers step)
- actions/upload-artifact@v7 (Upload Screenshots as Artifacts step)
- actions/upload-artifact@v7 (Upload Test Reports step)
- actions/upload-artifact@v7 (Upload Dashboard Results step)
All should be pinned to their full 40-hex-character SHA, e.g. uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4

Locations:

- `action.yml:161`
- `action.yml:167`
- `action.yml:173`
- `action.yml:213`
- `action.yml:249`
- `action.yml:257`
- `action.yml:265`

### hardcoded-credentials (severity: high)

The 'staging-password' input has a hardcoded literal default value of 'password123'. This matches the pattern password: <literal-value> and embeds a credential directly in the action definition. Even as a default/example value, hardcoded passwords in action.yml are a security risk as they may be used in production without being overridden. The default should be removed or replaced with an empty string, requiring callers to always supply the value explicitly.

Locations:

- `action.yml:29`

## Iteration Notes

### Iteration 1

**Fixes applied:** hardcoded-credentials, unpinned-uses, script-injection

**Notes:**

Fixed all three findings in hardened/action/action.yml:
1. hardcoded-credentials (line 29): Replaced default: 'password123' with default: '' for the staging-password input.
2. unpinned-uses (lines 161, 167, 173, 213, 249, 257, 265): Pinned all 7 uses: references to full 40-char commit SHAs — actions/cache@v5 → caa296126883cff596d87d8935842f9db880ef25, actions/checkout@v6 → d23441a48e516b6c34aea4fa41551a30e30af803, actions/setup-python@v6 → ece7cb06caefa5fff74198d8649806c4678c61a1, actions/upload-artifact@v7 → cf430e030ddbb5b0abf93d22962f4752f3646cd9. Original tags preserved as inline comments.
3. script-injection (lines 271, 273): Moved ${{ steps.autoqa.outcome }} into an env: block as AUTOQA_OUTCOME and replaced both inline ${{ steps.autoqa.outcome }} interpolations in the run: shell script with $AUTOQA_OUTCOME.

