<p align="center">
  <img src="https://raw.githubusercontent.com/open-coder-ai/chock/main/docs/assets/logo.svg" alt="chock logo" width="110">
</p>

<h1 align="center">chock-codex-plugins</h1>

<p align="center"><strong>Chock policies as Codex plugins — a real <code>PreToolUse</code> deny hook, held untrusted until you approve it.</strong></p>

<p align="center">

[![Generated-only](https://github.com/open-coder-ai/chock-codex-plugins/actions/workflows/generated-only.yml/badge.svg)](https://github.com/open-coder-ai/chock-codex-plugins/actions/workflows/generated-only.yml)
[![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)
[![Contribute upstream](https://img.shields.io/badge/contribute-chock--catalog-8957e5)](https://github.com/open-coder-ai/chock-catalog)

</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/open-coder-ai/chock/main/docs/assets/demo.gif" alt="Chock's demo: an agent runs a destructive command and a guard plugin denies it before it executes" width="760">
</p>

An agent you're running can already touch your shell, your git history, and your CI config.
You want it to move fast without being the reason a stray `helm uninstall` actually happens.
Telling it to be careful in a prompt is not a guarantee; a plugin that can refuse the command
is closer to one — a matched destructive command is denied before it runs, witnessed on a
real Codex Desktop install (Windows, 2026-08-24).

**Codex makes you approve this first.** Codex installs every hook UNTRUSTED. Until you open a
plugin's page and approve its hook ("needs review before it can run" → Trust), the plugin is
advisory text only — the trust is bound to a hash of the hook command, so an update silently
voids it. Any hook failure (missing `python3`, timeout) fails **open**: Codex allows it.

## Install

Codex reads this repository as a git marketplace, the same mechanism as its Plugins UI:

```toml
# ~/.codex/config.toml
[marketplaces.chock-codex]
source_type = "git"
source = "https://github.com/open-coder-ai/chock-codex-plugins.git"
```

Then install plugins from the `chock-codex` marketplace in the Plugins UI, and approve each
guard's hook trust review.

## What you get

See **[PLUGINS.md](PLUGINS.md)** for the full list: each policy, its version, and whether it
enforces or advises in this client. A plugin governs one session; it doesn't run in CI or
travel with a clone. For enforcement that follows the repository, with no trust toggle to
forget, install Chock directly: `pip install chock && chock init && chock sync --ci`.

## Generated from chock-catalog

Every file here is compiled from policy sources in
[chock-catalog](https://github.com/open-coder-ai/chock-catalog) by
[chock](https://github.com/open-coder-ai/chock). Pull requests against this repository are
closed automatically — open them against the catalog instead.

- **Generated only:** CI regenerates from the pinned catalog and fails on any difference.
- **Byte-identical guards:** guard scripts and the hook adapter are verbatim copies of their
  framework sources.
- **Best-effort, not a boundary:** guards are pattern-based filters; see
  [SECURITY.md](https://github.com/open-coder-ai/chock/blob/main/SECURITY.md).
- This README is the exception: the one hand-written file here, outside the guarantee.

## Part of open-coder-ai

| | |
|---|---|
| [agentseam](https://github.com/open-coder-ai/agentseam) | the primitives — one handler API and a verified capability matrix across 16 agents |
| [chock](https://github.com/open-coder-ai/chock) | the compiler — one policy into git hooks, CI gates and native pre-tool hooks |
| [chock-catalog](https://github.com/open-coder-ai/chock-catalog) | the policies — 39, each labelled enforced or advisory, with replayed evals |
| [context-report](https://github.com/open-coder-ai/context-report) | the evidence — a signed report of whether an agent artifact actually works |
| [chock-threat-intel](https://github.com/open-coder-ai/chock-threat-intel) | the threat ledger the catalog's policies answer to |
| chock-{claude,cursor,copilot,codex}-plugins | the catalog, packaged for each agent's plugin format (generated) |
| chock-quickstart · chock-example | template repos: what `chock init` leaves behind, and a full adoption |

## License

Apache-2.0, same as the framework and the catalog.
