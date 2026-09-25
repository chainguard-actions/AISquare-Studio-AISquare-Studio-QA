<!-- markdownlint-disable -->

# Hardening Report: AISquare-Studio--AISquare-Studio-QA/v0.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **AISquare-Studio--AISquare-Studio-QA/v0.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

The `staging-password` input has a hardcoded literal default value of `'password123'` (matching the pattern `password: <alphanumeric-value>`). Even though this is a default for a test credential, embedding a literal password in the action definition is a hardcoded-credentials violation. Any caller that does not override this input will silently use this plaintext password.

Locations:

- `action.yml:30`

### unpinned-uses (severity: high)

All six `uses:` references in action.yml use mutable version tags instead of pinned 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or overwritten. Failing references:
- `uses: actions/cache@v5` (Cache AutoQA Repository step, line ~108)
- `uses: actions/checkout@v6` (Checkout AutoQA Action Repository step, line ~113)
- `uses: actions/setup-python@v6` (Setup Python Environment step, line ~120)
- `uses: actions/cache@v5` (Cache Playwright Browsers step, line ~126)
- `uses: actions/upload-artifact@v7` (Upload Screenshots step, line ~170)
- `uses: actions/upload-artifact@v7` (Upload Test Reports step, line ~180)

Locations:

- `action.yml:108`
- `action.yml:113`
- `action.yml:120`
- `action.yml:126`
- `action.yml:170`
- `action.yml:180`

## Iteration Notes

### Iteration 1

**Fixes applied:** hardcoded-credentials, unpinned-uses

**Notes:**

Fixed two findings in hardened/action/action.yml:
1. hardcoded-credentials: Removed the hardcoded default value 'password123' from the staging-password input. The input now has no default, so callers must explicitly supply a password rather than silently inheriting a plaintext credential.
2. unpinned-uses: Pinned all 6 uses: references to full 40-character commit SHAs (resolved via lookup_action_sha): actions/cache@v5→caa296126883cff596d87d8935842f9db880ef25, actions/checkout@v6→d23441a48e516b6c34aea4fa41551a30e30af803, actions/setup-python@v6→ece7cb06caefa5fff74198d8649806c4678c61a1, actions/upload-artifact@v7→043fb46d1a93c77aae656e7c1c64a875d1fc6a0a. Version tags are preserved as inline comments.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the _set_outputs() method in src/autoqa/action_runner.py (line 530). The original code wrote values to $GITHUB_OUTPUT without sanitization — single-line values used `key=value\n` and multiline values used a heredoc `key<<EOF\nvalue\nEOF\n`. Both paths were vulnerable to injection via attacker-controlled PR body content (flow_name, tier, area, etag, error, test_file_path). The fix: (1) removes the heredoc path entirely (it was inherently unsafe since an attacker could inject 'EOF' to escape the delimiter), (2) converts all values to strings with None-safety, and (3) strips all \r and \n characters from every value before writing to GITHUB_OUTPUT. This prevents an attacker who controls the PR body from injecting newlines to poison subsequent steps that consume these outputs via ${{ steps.autoqa.outputs.* }}.

