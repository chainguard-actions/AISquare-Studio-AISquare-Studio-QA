<!-- markdownlint-disable -->

# Hardening Report: AISquare-Studio--AISquare-Studio-QA/v0.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **AISquare-Studio--AISquare-Studio-QA/v0.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All six `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA commit hashes. This exposes the action to supply-chain attacks if any of the referenced actions are compromised or their tags are moved. Failing references: `actions/cache@v5` (×2), `actions/checkout@v6`, `actions/setup-python@v6`, `actions/upload-artifact@v7` (×2).

Locations:

- `action.yml:97`
- `action.yml:103`
- `action.yml:112`
- `action.yml:118`
- `action.yml:161`
- `action.yml:169`

### hardcoded-credentials (severity: high)

The `staging-password` input in action.yml has a hardcoded literal default value of `password123`. This is a plaintext credential embedded in the action definition. Even as a default, shipping a real-looking password in source code is a security risk and violates the hardcoded-credentials check. The literal value matches the pattern `password\s*:\s*[A-Za-z0-9...]{7,}`.

Locations:

- `action.yml:30`

### github-env-injection (severity: high)

The `_set_outputs` method in `src/autoqa/action_runner.py` writes values directly to `$GITHUB_OUTPUT` without sanitizing newlines. For single-line values it uses `f"{key}={value}\n"`, and for multiline values it uses a heredoc with a fixed `EOF` delimiter: `f"{key}<<EOF\n{value}\nEOF\n"`. The `pr_body` output value is sourced from the `PR_BODY` environment variable, which is set to `${{ inputs.pr-body }}` (defaulting to `${{ github.event.pull_request.body }}`). An attacker controlling the PR body can embed a line containing only `EOF` to terminate the heredoc early, then inject arbitrary `KEY=VALUE` pairs into `$GITHUB_OUTPUT`, poisoning downstream step outputs. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before any write.

Locations:

- `src/autoqa/action_runner.py:367`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, hardcoded-credentials, github-env-injection

**Notes:**

1. Pinned all 6 `uses:` references in action.yml to full 40-char SHAs: actions/cache@27d5ce7f107fe9357f9df03efb73ab90386fccae (×2), actions/checkout@df4cb1c069e1874edd31b4311f1884172cec0e10, actions/setup-python@a309ff8b426b58ec0e2a45f0f869d46889d02405, actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a (×2). 2. Removed hardcoded 'password123' default from staging-password input in action.yml and from all three fallback defaults in action_runner.py (_load_config, _generate_test_code, _execute_test), replacing with empty strings. 3. Fixed _set_outputs in action_runner.py: multiline values now use a cryptographically random unique delimiter (secrets.token_hex(16)) to prevent heredoc injection; single-line values have newlines/carriage-returns stripped before writing to $GITHUB_OUTPUT.

