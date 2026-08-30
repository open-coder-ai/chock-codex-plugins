# chock-codex-plugins

[![Generated-only](https://github.com/open-coder-ai/chock-codex-plugins/actions/workflows/generated-only.yml/badge.svg)](https://github.com/open-coder-ai/chock-codex-plugins/actions/workflows/generated-only.yml)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Contribute upstream](https://img.shields.io/badge/contribute-chock--catalog-8957e5)](https://github.com/open-coder-ai/chock-catalog)

Chock policies packaged as installable plugins for **OpenAI Codex**. Guard policies ship a
real `PreToolUse` hook, so a matched destructive command is **denied before it runs** —
witnessed blocking on a real Codex Desktop install (Windows, 2026-08-24).

**One thing Codex makes you do first, and honestly stated here because the plugin
description says "session-enforced": Codex installs every hook UNTRUSTED.** Until you open
the plugin's page and approve its hook ("needs review before it can run" → Trust), the
plugin is advisory text only. That trust is bound to a hash of the hook command, so **a
plugin update silently voids it** — re-approve after updating. Any hook failure (missing
`python3`, timeout) **fails OPEN**: Codex allows the command.

<img src="docs/assets/hero.svg" alt="Animated replay: an agent runs helm uninstall and the chock guard denies it before it runs (Codex, permissionDecision deny)" width="720">

**This repository is generated.** Every file is compiled from policy sources in
[chock-catalog](https://github.com/open-coder-ai/chock-catalog) by
[chock](https://github.com/open-coder-ai/chock). Pull requests here are closed with a
pointer to the catalog.

## Install

Codex reads this repository as a git marketplace (the same mechanism as its Plugins UI):

```toml
# ~/.codex/config.toml
[marketplaces.chock-codex]
source_type = "git"
source = "https://github.com/open-coder-ai/chock-codex-plugins.git"
```

Then install plugins from the `chock-codex` marketplace in the Plugins UI, and **approve
each guard's hook trust review**.

Using a different agent? Sibling repos built from the same catalog:
[chock-claude-plugins](https://github.com/open-coder-ai/chock-claude-plugins) (Claude
Code), [chock-copilot-plugins](https://github.com/open-coder-ai/chock-copilot-plugins)
(Copilot CLI / VS Code),
[chock-cursor-plugins](https://github.com/open-coder-ai/chock-cursor-plugins) (Cursor).

## Layout

```
codex/<policy-id>/                 Codex plugin packages (.codex-plugin/plugin.json;
                                   hooks/hooks.json where the policy has a guard)
.claude-plugin/marketplace.json    the index Codex reads from git marketplaces
```

See **[PLUGINS.md](PLUGINS.md)** for every policy, its version and its posture.

**A plugin is not the same as adopting Chock.** Repo-wide enforcement — git hooks and a CI
gate a session cannot skip, with no trust toggle to forget — comes from installing Chock in
the repository:

```bash
pip install chock
chock init && chock sync --ci
```

## Trust

- **Generated only:** CI regenerates from the pinned catalog and fails on any difference.
- **Byte-identical guards:** guard scripts and the hook adapter are verbatim copies of
  their framework sources.
- **Best-effort, not a boundary:** guards are pattern-based filters. See
  [SECURITY.md](https://github.com/open-coder-ai/chock/blob/main/SECURITY.md).

## License

Apache-2.0, same as the framework and the catalog.
