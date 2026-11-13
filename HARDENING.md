<!-- markdownlint-disable -->

# Hardening Report: AISquare-Studio--AISquare-Studio-QA/v0.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **AISquare-Studio--AISquare-Studio-QA/v0.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action pins its actions to mutable version tags instead of 40-character commit SHAs: `uses: actions/cache@v5` (twice), `uses: actions/checkout@v6`, `uses: actions/setup-python@v6` and `uses: actions/upload-artifact@v7` (twice).

Locations:

- `action.yml:114`
- `action.yml:123`
- `action.yml:132`
- `action.yml:140`
- `action.yml:211`
- `action.yml:222`

### hardcoded-credentials (severity: high)

The input `staging-password` has a hardcoded literal default: `default: 'password123'`. If a caller doesn't set the input, this known credential is passed to the action as STAGING_PASSWORD. Remove the default and require callers to pass it from a secret (e.g. `${{ secrets.STAGING_PASSWORD }}`).

Locations:

- `action.yml:31`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index bd04620..b00ea58 100644
--- a/action.yml
+++ b/action.yml
@@ -1,142 +1,130 @@
 name: 'AISquare Studio AutoQA'
 description: 'AI-powered test generation with metadata-driven organization (flow_name, tier, area) using CrewAI + Playwright'
 author: 'AISquare Studio'
-
 branding:
   icon: 'zap'
   color: 'blue'
-
 inputs:
   openai-api-key:
     description: 'OpenAI API key for test generation'
     required: true
-  
   qa-github-token:
     description: 'GitHub token with access to AutoQA repository (required for private repos)'
     required: false
     default: ${{ github.token }}
-  
   staging-url:
     description: 'Staging environment URL for testing'
     required: true
-  
   staging-email:
     description: 'Test email for staging environment login'
     required: false
-    default: 'test@example.com'
-  
   staging-password:
     description: 'Test password for staging environment login'
     required: false
-    default: 'password123'
-  
   target-repo-path:
     description: 'Path to the target repository (usually ".")'
     required: false
     default: '.'
-  
   git-user-name:
     description: 'Git user name for commits'
     required: false
     default: 'AutoQA Bot'
-  
   git-user-email:
     description: 'Git user email for commits'
     required: false
     default: 'rabia.tahirr@opengrowth.com'
-  
   pr-body:
     description: 'Pull request description to parse for AutoQA metadata block (```autoqa\nflow_name: ...\ntier: ...\n```)'
     required: false
     default: ${{ github.event.pull_request.body }}
-  
   test-directory:
     description: 'Base directory for generated tests (will create tests/autoqa/{tier}/{area}/test_{flow_name}.py structure)'
     required: false
     default: 'tests/autoqa'
-  
   create-pr:
     description: 'Create a pull request instead of pushing directly to main branch'
     required: false
     default: 'false'
-  
   execution-mode:
     description: 'Mode of operation: "generate" (new test only), "suite" (run all tests), or "all" (generate + run all)'
     required: false
     default: 'generate'
-
 outputs:
   test_generated:
     description: 'Whether a test was successfully generated'
     value: ${{ steps.autoqa.outputs.test_generated }}
-  
   test_file_path:
     description: 'Path to the generated test file (e.g., tests/autoqa/A/auth/test_user_login.py)'
     value: ${{ steps.autoqa.outputs.test_file_path }}
-  
   test_results:
     description: 'JSON string of test execution results'
     value: ${{ steps.autoqa.outputs.test_results }}
-  
   generation_metadata:
     description: 'JSON string of test generation metadata (includes flow_name, tier, area, etag, steps)'
     value: ${{ steps.autoqa.outputs.generation_metadata }}
-  
   screenshot_path:
     description: 'Path to the generated screenshot'
     value: ${{ steps.autoqa.outputs.screenshot_path }}
-  
   etag:
     description: 'ETag hash for idempotency (SHA256 of metadata + steps)'
     value: ${{ steps.autoqa.outputs.etag }}
-  
   flow_name:
     description: 'Flow name from AutoQA metadata'
     value: ${{ steps.autoqa.outputs.flow_name }}
-  
   tier:
     description: 'Test tier from AutoQA metadata (A, B, or C)'
     value: ${{ steps.autoqa.outputs.tier }}
-  
   area:
     description: 'Test area from AutoQA metadata (optional)'
     value: ${{ steps.autoqa.outputs.area }}
-  
   error:
     description: 'Error message if test generation or validation failed'
     value: ${{ steps.autoqa.outputs.error }}
-
 runs:
   using: 'composite'
   steps:
     # Cache the AutoQA action repository to avoid repeated clones
-    - name: 🗂️ Cache AutoQA Repository
       uses: actions/cache@v5
+    - name: "\U0001F5C2️ Cache AutoQA Repository"
       with:
         path: .autoqa-action
         key: ${{ runner.os }}-autoqa-repo-${{ github.sha }}-v1
         restore-keys: |
           ${{ runner.os }}-autoqa-repo-v1
-          
-    - name: 🔍 Checkout AutoQA Action Repository
       uses: actions/checkout@v6
+    - name: "\U0001F50D Checkout AutoQA Actio
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, hardcoded-credentials, prior-fix

**Notes:**

All three findings are fixed in action.yml. Nothing has been run or tested.

- **Pinned actions:** every `uses:` now points to a full commit SHA, with the tag kept as a comment: actions/cache v5 (both uses), actions/checkout v6, actions/setup-python v6 and actions/upload-artifact v7 (both uses).
- **Credential defaults removed:** `staging-password` no longer defaults to `password123`, and `staging-email` no longer defaults to `test@example.com`. This repeats what the previous harden did. Callers who relied on these defaults now get empty values, so they need to pass both inputs, with the password coming from a secret.
- **Checkout ref:** the step that checks out AISquare-Studio/AISquare-Studio-QA used `ref: ${{ github.action_ref || 'main' }}`. In this mirror, that ref is the mirror's own tag or commit, which may not exist in the upstream repo. I set it to the fixed value `v0.1.0`, the upstream tag this version was built from.

