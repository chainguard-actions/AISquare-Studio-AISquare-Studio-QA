<!-- markdownlint-disable -->

# Hardening Report: AISquare-Studio--AISquare-Studio-QA/v0.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **AISquare-Studio--AISquare-Studio-QA/v0.2.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 7 `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if any upstream action is compromised or its tag is moved. Failing references: `actions/cache@v5` (×2), `actions/checkout@v6`, `actions/setup-python@v6`, `actions/upload-artifact@v7` (×3).

Locations:

- `action.yml:168`
- `action.yml:176`
- `action.yml:184`
- `action.yml:192`
- `action.yml:232`
- `action.yml:243`
- `action.yml:255`

### script-injection (severity: high)

Sub-rule (a): The 'Fail if tests failed' run: block directly interpolates `${{ steps.autoqa.outcome }}` inside shell command strings. Any `${{ ... }}` expression interpolated directly into a run: script is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, bypassing shell quoting. Offending lines: `if [ "${{ steps.autoqa.outcome }}" = "skipped" ]` and `elif [ "${{ steps.autoqa.outcome }}" = "failure" ]`.

Locations:

- `action.yml:268`

### hardcoded-credentials (severity: high)

The `staging-password` input has a hardcoded literal default value `password123` — a non-expression literal string assigned to a field whose name contains 'password'. This embeds a credential directly in the action definition rather than requiring callers to supply it via a secret expression.

Locations:

- `action.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, hardcoded-credentials

**Notes:**

Fixed all 3 findings in action.yml: (1) Pinned all 7 unpinned `uses:` references to full 40-char SHAs with tag comments preserved — actions/cache@v5→27d5ce7f (×2), actions/checkout@v6→df4cb1c0, actions/setup-python@v6→a309ff8b, actions/upload-artifact@v7→043fb46d (×3). (2) Fixed script-injection in 'Fail if tests failed' step by moving `${{ steps.autoqa.outcome }}` into an `env:` block as `AUTOQA_OUTCOME` and referencing it as `$AUTOQA_OUTCOME` in the shell script. (3) Removed hardcoded `password123` default from `staging-password` input to eliminate the embedded credential.

