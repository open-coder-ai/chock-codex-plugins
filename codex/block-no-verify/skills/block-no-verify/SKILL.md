---
name: block-no-verify
description: "Best-effort guard against bypassing git hooks via git commit/push --no-verify, commit's short -n form, or -c core.hooksPath overrides. On git push, -n means --dry-run and stays allowed. Known bypass classes include aliases, wrapper scripts, and non-standard clients. Fix the underlying hook failure instead of skipping validation."
metadata:
  chock.artifact: rule
  chock.enforcement: advise
  chock.hooks: hooks/hooks.json
---

# Block No-Verify

Best-effort guard against bypassing git hooks via git commit/push --no-verify, commit's short -n form, or -c core.hooksPath overrides. On git push, -n means --dry-run and stays allowed. Known bypass classes include aliases, wrapper scripts, and non-standard clients. Fix the underlying hook failure instead of skipping validation.

```
never(commit): --no-verify|-n; never(push): --no-verify
if(hook_fails): fix_issue; never(skip_hook)
```

This policy is enforced in Codex by the PreToolUse hook shipped with this plugin, once its one-time trust review is approved (until then it is advisory, and a plugin update voids the trust until re-approved). Subject to the fail-open conditions in the plugin description. Repo-wide enforcement across every commit and in CI still needs `chock sync`. See https://github.com/open-coder-ai/chock
