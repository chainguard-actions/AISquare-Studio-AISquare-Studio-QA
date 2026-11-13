<!-- markdownlint-disable -->

# Hardening Report: AISquare-Studio--AISquare-Studio-QA/v0.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **AISquare-Studio--AISquare-Studio-QA/v0.2.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action steps use version tags instead of full 40-character commit SHAs: actions/cache@v5 (lines 164 and 189), actions/checkout@v6 (line 172), actions/setup-python@v6 (line 181), actions/upload-artifact@v7 (lines 268, 279 and 291).

Locations:

- `action.yml:164`
- `action.yml:172`
- `action.yml:181`
- `action.yml:189`
- `action.yml:268`
- `action.yml:279`
- `action.yml:291`

### script-injection (severity: high)

Rule (a): the 'Fail if tests failed' step puts a GitHub Actions expression straight into its run: script: `if [ "${{ steps.autoqa.outcome }}" = "skipped" ]; then` and `elif [ "${{ steps.autoqa.outcome }}" = "failure" ]; then`. Pass it through an env: variable (e.g. AUTOQA_OUTCOME) and use "$AUTOQA_OUTCOME" in the script instead.

Locations:

- `action.yml:304`
- `action.yml:306`

### hardcoded-credentials (severity: high)

The input 'staging-password' has the literal default value 'password123'. A password should not have a literal default. Remove it and require the caller to pass the value from a secret.

Locations:

- `action.yml:31`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index b510649..58b84d1 100644
--- a/action.yml
+++ b/action.yml
@@ -1,192 +1,170 @@
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
     default: 'test@example.com'
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
     description: 'Mode of operation: "generate" (new test only), "suite" (run all tests), "all" (generate + run all), "auto-criteria" (AI generates test criteria from PR diff for developer review), "gap-driven" (uses memory coverage gaps to generate test criteria for uncovered modules), or "gap-analysis" (scans for present/missing test workflows and persists results to a SQLite database)'
     required: false
     default: 'generate'
-  
   action-ref:
     description: 'Git ref (branch, tag, or SHA) to checkout for the AutoQA action repository. Defaults to main when not provided.'
     required: false
     default: 'main'
-
   auto-criteria-fallback:
     description: 'When true, if no autoqa block is found in the PR description, automatically generate test criteria from the PR diff (Proposal 16)'
     required: false
     default: 'false'
-
   auto-criteria-mode:
     description: 'Auto-criteria mode: "suggest" (post for review) or "auto" (proceed if high confidence)'
     required: false
     default: 'suggest'
-
   auto-criteria-threshold:
     description: 'Confidence score threshold (0-100) for auto-proceed mode'
     required: false
     default: '85'
-
   auto-criteria-approval:
-    description: 'Approval mechanism: "reaction" (👍), "comment" (/autoqa approve), or "label" (autoqa:approved)'
+    description: "Approval mechanism: \"reaction\" (\U0001F44D), \"comment\" (/autoqa approve), or \"label\" (autoqa:approved)"
     required: false
     default: 'reaction'
-
   gap-analysis-scope:
     description: 'Scope for gap-analysis mode: "auto" (detect from event), "pr" (PR changed files only), or "full" (entire repo)'
     required: false
     default: 'auto'
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
     value: ${{ steps.autoqa.outp
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, hardcoded-credentials, prior-fix

**Notes:**

In action.yml: pinned actions/cache@v5 (2 places), actions/checkout@v6, actions/setup-python@v6 and actions/upload-artifact@v7 (3 places) to full commit SHAs, with the tag kept in a trailing comment. In the 'Fail if tests failed' step, steps.autoqa.outcome now comes in through env (AUTOQA_OUTCOME) and the script reads "$AUTOQA_OUTCOME". Removed the 'password123' default from staging-password. Callers that relied on that default now pass an empty STAGING_PASSWORD unless they supply the value from a secret. These edits re-apply what the earlier harden did (prior-fix). That patch also changed whitespace and the emoji escaping, which made no security difference, so I left those alone. The checkout of the AutoQA repo still uses the action-ref input (default 'main'), as upstream does. It doesn't depend on github.action_ref, so I left it unchanged.

