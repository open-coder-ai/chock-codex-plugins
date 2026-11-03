---
name: block-persistence-shapes
description: "Best-effort guard against shell commands that publish or keep access after the session: npm/pnpm/yarn/twine/ poetry/uv/cargo/gem publish, docker/podman push and login, npm token and registry edits, repos made public; user services, launch agents, cron/at/schtasks, Run keys, authorized_keys, runner registration, sudoers, setuid bits, detached downloads. Asks on gh release create, git remote add, git push to a URL. Misses: scripts, aliases, system units, shell rc files, ssh-run commands."
metadata:
  chock.artifact: rule
  chock.enforcement: advise
  chock.hooks: hooks/hooks.json
---

# Block Persistence and Publishing Commands

Best-effort guard against shell commands that publish or keep access after the session: npm/pnpm/yarn/twine/ poetry/uv/cargo/gem publish, docker/podman push and login, npm token and registry edits, repos made public; user services, launch agents, cron/at/schtasks, Run keys, authorized_keys, runner registration, sudoers, setuid bits, detached downloads. Asks on gh release create, git remote add, git push to a URL. Misses: scripts, aliases, system units, shell rc files, ssh-run commands.

```
never(run): publish(npm|pnpm|yarn|twine|poetry|uv|cargo|gem|docker|podman), registry_login_or_token, repo_public, persist(user_service|launch_agent|cron|at|schtasks|run_key|authorized_keys|runner|sudoers|setuid|detached_download)
ask(person): release_create, remote_add, push_to_url; allow: dry_run, pack, build, status, list  # human decisions
```

This policy is enforced in Codex by the PreToolUse hook shipped with this plugin, once its one-time trust review is approved (until then it is advisory, and a plugin update voids the trust until re-approved). Subject to the fail conditions in the plugin description. Repo-wide enforcement across every commit and in CI still needs `chock sync`. See https://github.com/open-coder-ai/chock
