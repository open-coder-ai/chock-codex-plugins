---
name: block-destructive-commands
description: "Best-effort guard against destructive commands: rm -rf targeting absolute, home, or root-adjacent paths; git push --force (not --force-with-lease); git reset --hard; git clean -f; kubectl delete; terraform destroy. Known bypass classes include aliases, quoted arguments, non-standard clients, and scripts that invoke these commands indirectly. This is friction, not a security boundary."
metadata:
  chock.artifact: rule
  chock.enforcement: advise
  chock.hooks: hooks/hooks.json
---

# Block Destructive Commands

Best-effort guard against destructive commands: rm -rf targeting absolute, home, or root-adjacent paths; git push --force (not --force-with-lease); git reset --hard; git clean -f; kubectl delete; terraform destroy. Known bypass classes include aliases, quoted arguments, non-standard clients, and scripts that invoke these commands indirectly. This is friction, not a security boundary.

```
block(destructive_command): rm_-rf(/|~|.), git_push_--force, git_reset_--hard, git_checkout_., git_clean_-f, kubectl_delete, terraform_destroy
require_approval: reset_hard|rm_-rf|branch_-D; prefer: stash|soft_reset|force-with-lease|dry-run
```

This policy is enforced in Codex by the PreToolUse hook shipped with this plugin, once its one-time trust review is approved (until then it is advisory, and a plugin update voids the trust until re-approved). Subject to the fail-open conditions in the plugin description. Repo-wide enforcement across every commit and in CI still needs `chock sync`. See https://github.com/open-coder-ai/chock
