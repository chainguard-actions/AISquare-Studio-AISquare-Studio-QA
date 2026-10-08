<!-- markdownlint-disable -->

# Hardening Report: AISquare-Studio--AISquare-Studio-QA/v0.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **AISquare-Studio--AISquare-Studio-QA/v0.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 6 `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA commit digests. This exposes the action to supply-chain attacks where a tag can be silently moved to point to malicious code. Failing references: `actions/cache@v5` (×2), `actions/checkout@v6`, `actions/setup-python@v6`, `actions/upload-artifact@v7` (×2).

Locations:

- `action.yml:118`
- `action.yml:126`
- `action.yml:135`
- `action.yml:143`
- `action.yml:196`
- `action.yml:207`

### hardcoded-credentials (severity: high)

action.yml contains a hardcoded literal password as the default value for the `staging-password` input: `default: 'password123'`. This is a hardcoded credential embedded in the action definition. Any workflow using this action without explicitly overriding the input will use this literal password. Additionally, a hardcoded email address is set as the default for `staging-email`: `default: 'test@example.com'`.

Locations:

- `action.yml:33`
- `action.yml:28`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, hardcoded-credentials

**Notes:**

Fixed all 6 unpinned `uses:` references by replacing mutable version tags with pinned 40-character SHA commit digests (preserving the tag in a comment for readability): actions/cache@v5 (×2) → SHA caa296126883cff596d87d8935842f9db880ef25, actions/checkout@v6 → SHA d23441a48e516b6c34aea4fa41551a30e30af803, actions/setup-python@v6 → SHA ece7cb06caefa5fff74198d8649806c4678c61a1, actions/upload-artifact@v7 (×2) → SHA cf430e030ddbb5b0abf93d22962f4752f3646cd9. Removed hardcoded credential defaults: dropped `default: 'test@example.com'` from staging-email and `default: 'password123'` from staging-password; both inputs remain optional (required: false) but no longer embed insecure credentials in the action definition.

