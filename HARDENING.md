<!-- markdownlint-disable -->

# Hardening Report: AISquare-Studio--AISquare-Studio-QA/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **AISquare-Studio--AISquare-Studio-QA/v0.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml has composite steps that point to mutable tags instead of 40-char commit SHAs: actions/cache@v5 (line 169), actions/checkout@v6 (line 178), actions/setup-python@v6 (line 187), actions/cache@v5 (line 217), actions/upload-artifact@v7 (lines 293, 304, 316).

Locations:

- `action.yml:169`
- `action.yml:178`
- `action.yml:187`
- `action.yml:217`
- `action.yml:293`
- `action.yml:304`
- `action.yml:316`

### script-injection (severity: high)

Rule (a) is broken: an expression is placed directly in the run block of step '❌ Fail if tests failed': `if [ "${{ steps.autoqa.outcome }}" = "skipped" ]; then` and `elif [ "${{ steps.autoqa.outcome }}" = "failure" ]; then`. To fix it, pass the value in through env (e.g. AUTOQA_OUTCOME) and use "$AUTOQA_OUTCOME".

Locations:

- `action.yml:329`

### hardcoded-credentials (severity: high)

The input 'staging-password' has a hardcoded literal default value: `default: 'password123'`. Remove the default and make callers supply it through a secret.

Locations:

- `action.yml:36`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 76d089b..22ef81c 100644
--- a/action.yml
+++ b/action.yml
@@ -1,194 +1,170 @@
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
   openai-model:
     description: 'OpenAI model to use for test generation (e.g., openai/gpt-4.1, openai/gpt-4o, openai/gpt-3.5-turbo)'
     required: false
     default: 'openai/gpt-4.1'
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
     description: 'Path to the generated test file (e.g., tests/autoqa/A/auth
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, hardcoded-credentials, prior-fix

**Notes:**

- Pinned the action references to full commit SHAs from lookup_action_sha, keeping the tag as a comment: actions/cache v5 (2 uses), actions/checkout v6, actions/setup-python v6, actions/upload-artifact v7 (3 uses).
- Script injection: the '❌ Fail if tests failed' step no longer puts steps.autoqa.outcome straight into the shell script. It now reads the value from an env variable, AUTOQA_OUTCOME, quoted as "$AUTOQA_OUTCOME".
- Hardcoded credentials / prior fix: removed the default 'password123' from staging-password. Callers now have to pass it through a secret. As the previous harden did, I also removed the 'test@example.com' default from staging-email.
- I did not copy the previous harden's blank-line and emoji-escaping changes, since they only affect formatting.
- I left the checkout of AISquare-Studio/AISquare-Studio-QA at inputs.action-ref (default 'main') as upstream had it, because it doesn't use the mirror's github.action_ref.

