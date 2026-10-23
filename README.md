<p align="center">
  <img src="https://raw.githubusercontent.com/open-coder-ai/chock/main/docs/assets/logo.svg" alt="chock logo" width="110">
</p>

<h1 align="center">chock-codex-plugins</h1>

<p align="center"><strong>Blocks your Codex agent's wrong moves before they land.</strong></p>
<p align="center">Chock policies as Codex plugins — a real <code>PreToolUse</code> deny hook for guards, a <code>Stop</code> gate for writes, both held untrusted until you approve them.</p>

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
real Codex Desktop install (Windows, 2026-08-24). Gate policies work differently here: Codex
records no file-writing tool vocabulary, so they cannot judge the write itself and are wired at
`Stop` instead, re-reading what the turn left on disk.

**Codex makes you approve this first.** Codex installs every hook UNTRUSTED. Until you open a
plugin's page and approve its hook ("needs review before it can run" → Trust), the plugin is
advisory text only — the trust is bound to a hash of the hook command, so an update silently
voids it. A hook that cannot run at all (missing `python3`, timeout) fails **open**: Codex allows it. A
guard that runs and crashes is a **deny** here, because Codex rejects the confirmation prompt
other clients show, and a gate that cannot reach a decision refuses rather than allowing
something it never judged.

**How a deny reaches Codex.** Each guard plugin wires one `PreToolUse` hook, and the hook
answers on stdout with a JSON decision, not an exit code:
`{"hookSpecificOutput": {"hookEventName": "PreToolUse", "permissionDecision": "deny", "permissionDecisionReason": "..."}}`
with exit status 0. Gate plugins wire the same `PreToolUse` hook plus a `Stop` hook. The one
exit-code answer is exit 2, used only when no working Python 3.11+ is found.

## Two ways to get these plugins

**This marketplace: no chock on the machine.** Each policy is its own plugin, republished with
every chock release. You need Python 3.11+ and git, and nothing else. Codex reads this
repository as a git marketplace, the same mechanism as its Plugins UI:

```toml
# ~/.codex/config.toml
[marketplaces.chock-codex]
source_type = "git"
source = "https://github.com/open-coder-ai/chock-codex-plugins.git"
```

Then install plugins from the `chock-codex` marketplace in the Plugins UI, and approve each
guard's hook trust review.

**The builder: one plugin, needs chock.** Pick the policies you want at
**[chock.sh](https://chock.sh/coding/builder/)**; it writes one `chock install` command that
builds a single plugin on your machine from
[chock-catalog](https://github.com/open-coder-ai/chock-catalog). That command needs
`pip install chock`:

```bash
pip install chock
chock install --selection '<code>' --client codex --apply
```

Codex runs a plugin's hooks only after you trust them in `/hooks`; approve the new plugin's
hooks there, or it stays advisory text.

**Use one route for a given policy, never both.** Every copy of a plugin runs its own hook, so
a policy installed from both places runs twice. To switch a policy from one route to the other,
remove it from the first route before installing it from the second: remove each plugin you
installed from the `chock-codex` marketplace in the Codex Plugins UI, and to drop the
marketplace too, delete the `[marketplaces.chock-codex]` table from `~/.codex/config.toml`.

### Builder links, one policy at a time

Each link opens the builder with that one policy preselected for Codex, pinned to chock-catalog
commit `e885f918a335e20dfddb4fad0331d4d5b4c5eb43` (the catalog's `main` on 2026-10-21). The pin is
the catalog's current version of each policy, which can be newer than the version listed in
[PLUGINS.md](PLUGINS.md).

| policy | in Codex | builder |
| :--- | :--- | :--- |
| `agent-discipline` | advisory | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiYWdlbnQtZGlzY2lwbGluZSIsInZlcnNpb24iOiIwLjAuMyIsInNoYTI1NiI6IjAzOGIxOWQ3MjZlZmMxMjVlZWFmMTMyMWFjNGZlYjcwYzdmZDFhMjA3ZGE4ZTg5NTNiZWQ2NWQ0YWNkZDg0NzIifV19) |
| `block-curl-pipe-sh` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiYmxvY2stY3VybC1waXBlLXNoIiwidmVyc2lvbiI6IjAuMS40Iiwic2hhMjU2IjoiMzRiOTYxYzQ4MjU4OGQwYjUxYzhkYzc2NTVlOTE1ZWE5OGM1YzQ3MDI3ZGNhMTE0ZjhhN2QxMjdmMzc2NjJmMiJ9XX0) |
| `block-destructive-commands` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiYmxvY2stZGVzdHJ1Y3RpdmUtY29tbWFuZHMiLCJ2ZXJzaW9uIjoiMC4zLjAiLCJzaGEyNTYiOiI1NDFiNGQzMjc3ODg0MjQ4ZDMyMTYwMWQwNGNlMDE4MjUzN2MzMGUyNTk2ZGM3ODBjMDAxYWU1MzJiMWI5ZDUwIn1dfQ) |
| `block-invisible-unicode` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiYmxvY2staW52aXNpYmxlLXVuaWNvZGUiLCJ2ZXJzaW9uIjoiMC4xLjEiLCJzaGEyNTYiOiJiMTA4NDMxNTcwNmRjMjZlMzA5N2RkMWMxYTg5M2M3ODA0MTVmMTE2OGFiNWE5OTVjMTlhODk1ZjkzNjg4YzVhIn1dfQ) |
| `block-no-verify` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiYmxvY2stbm8tdmVyaWZ5IiwidmVyc2lvbiI6IjAuNS4wIiwic2hhMjU2IjoiYTQwNmQ1MGU0Y2YzMGQ1NjQzMDg0MDgwYWQ0MjU5ZDM0MmYzZWRkMDcwMzY4ODhjNDdhODIxNGY2NjA5OTRjYyJ9XX0) |
| `block-test-skips` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiYmxvY2stdGVzdC1za2lwcyIsInZlcnNpb24iOiIwLjEuMCIsInNoYTI1NiI6Ijg1MzVjNTJlZDA1NWIwODRiM2Q4NzBiMzM5NDQ4NzQxZTE4MzRmNmZkOTFhYjJkMzJlYWQxZDI2M2Y0NzU1NzQifV19) |
| `block-unapproved-egress` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiYmxvY2stdW5hcHByb3ZlZC1lZ3Jlc3MiLCJ2ZXJzaW9uIjoiMC4yLjQiLCJzaGEyNTYiOiJlNDg2ZmUzN2IxMjFjMjA1NDFmNzFjZWY2Y2I4NTYwNWZlMGEwNmVlZWI2ZTU2MzgzYzk4ZmZiMDBjMDA3ZTEwIn1dfQ) |
| `block-unguarded-agent-spawn` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiYmxvY2stdW5ndWFyZGVkLWFnZW50LXNwYXduIiwidmVyc2lvbiI6IjAuMi4wIiwic2hhMjU2IjoiZTQ1YzIxNTUxZjFmMWEyNDIyNzU2ODNhZmU5YWUwMDQ2Mjc3ZDhkNTdmNTM3MzQ0YWM3MTNmZmJmYmE0NjEzNyJ9XX0) |
| `block-wildcard-agent-permissions` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiYmxvY2std2lsZGNhcmQtYWdlbnQtcGVybWlzc2lvbnMiLCJ2ZXJzaW9uIjoiMC4wLjEwIiwic2hhMjU2IjoiNmIzMTlkZjY4ZjAzYzkzNjI2YzY5NWQyOTg3MGMxNzk0NTZlZmJkZjRlZmIzMzAxMmJlOWZkMGVkOGEzOGVlZCJ9XX0) |
| `chock-mise` | advisory | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiY2hvY2stbWlzZSIsInZlcnNpb24iOiIwLjAuMyIsInNoYTI1NiI6IjFiMDcyYmQ1MTc0YWM4YTc1YmQ0NDRlZDEwMGQ0MWNkYjc2Y2MzOWYxZDY2Yzg2NmE3YzJkOWE3M2VlY2Y1MWYifV19) |
| `code-safety` | advisory | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiY29kZS1zYWZldHkiLCJ2ZXJzaW9uIjoiMC4wLjQiLCJzaGEyNTYiOiI1MTA2Nzk1MTIzNWE5YWRkMzBmMmFmYTIwNzYyYzUzZmQzOTJiYTJkNTczODQ1OTcwYzI5YmQzMDI5NTFlYWUyIn1dfQ) |
| `context-hygiene` | advisory | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiY29udGV4dC1oeWdpZW5lIiwidmVyc2lvbiI6IjAuMC4yIiwic2hhMjU2IjoiNDg5MmQyMTRjNjI4OTdiYjI4M2VmNTJmZjZlY2M4MDU2ZDY3MjEzNjMwZjYxN2MyMTIwNmRmNWQ1YjVlN2Q3NCJ9XX0) |
| `firecrawl-fallback-only` | advisory | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiZmlyZWNyYXdsLWZhbGxiYWNrLW9ubHkiLCJ2ZXJzaW9uIjoiMC4xLjEiLCJzaGEyNTYiOiJjMWZiM2Y0Y2QyNGRmYjA3MGJjMWRjZmEwZDU3ZDdlOTAxNzQ4MTkwOGJiNTQ4NTBjNTZkNTZkZmNjMDc5MDY0In1dfQ) |
| `git-safety` | advisory | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiZ2l0LXNhZmV0eSIsInZlcnNpb24iOiIwLjAuNCIsInNoYTI1NiI6Ijc4NGMzYTE4OGM1YzQyYTg4NjA3MDMwMTk5ZGU0YjVhY2E5ZjNjMmY0NjI3OThlYjk2NjkxYzcxMzA3Y2E4NTgifV19) |
| `guard-memory-writes` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiZ3VhcmQtbWVtb3J5LXdyaXRlcyIsInZlcnNpb24iOiIwLjAuOSIsInNoYTI1NiI6ImU4OGZkNDZlYmJiN2E1YTY2Zjc3ZTY0YjdhZTY1ZTQwOTQ1YTRkMmE1N2RjMjVkNTNjMzFmNzNlNTc4MGNkN2IifV19) |
| `injection-defense` | advisory | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiaW5qZWN0aW9uLWRlZmVuc2UiLCJ2ZXJzaW9uIjoiMC4wLjQiLCJzaGEyNTYiOiI2NDM3Y2UyYTM2YTdjYjRhNjBmYjc4YzM5NzhhMzM2MmFhMzVjYzljZGZiOTNmODY3ZTU4NTg1MjdlNzYwOWMwIn1dfQ) |
| `java-security` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoiamF2YS1zZWN1cml0eSIsInZlcnNpb24iOiIwLjcuMCIsInNoYTI1NiI6IjJiMWE4NTg4MTk0NmUxYmI2NTE1NzE4MzJjNGJlZDQ3ZjNmMjJlNjI1NTc1OWVkMjFkM2ExNzBkZWM5NzAwNTgifV19) |
| `limit-diff-size` | advisory | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoibGltaXQtZGlmZi1zaXplIiwidmVyc2lvbiI6IjAuMC4zIiwic2hhMjU2IjoiYTdlNWFkZDY2YTU3MDZhMDg4OTg0NzY2OTVmZTJjMzBjMDVlMDZlNGJmODBjMGI0YzY3MjJjZDI4OWU1OGFmNCJ9XX0) |
| `memory-discipline` | advisory | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoibWVtb3J5LWRpc2NpcGxpbmUiLCJ2ZXJzaW9uIjoiMC4wLjUiLCJzaGEyNTYiOiIwZjdhNjY3YTA3ZThmNWI5Yzk2ZWFkNmY5NmRmN2EyZTllOTg5YzNjMGVjNzc2YWJmODNhMDdmZGRjNjk1NWY4In1dfQ) |
| `no-a11y-regression` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoibm8tYTExeS1yZWdyZXNzaW9uIiwidmVyc2lvbiI6IjEuMS4yIiwic2hhMjU2IjoiMjFjNDdjZmFlYThkOWY3ZDYxZTlmNGQ5ZDliMTI3YjczNzE1MzU2MTNhNTg4ZTFjNWU3NDQzNjAxM2QwYjFkYiJ9XX0) |
| `pin-github-actions` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoicGluLWdpdGh1Yi1hY3Rpb25zIiwidmVyc2lvbiI6IjAuMC43Iiwic2hhMjU2IjoiMTY4ZDZlOTQ2MzI0OTMwOWFiZDQwMjJhZGMxNzZiOGQ5N2FiMWZkYjE2ZjRjYTU5NjdmNWM0Y2E4NTg1NjllYyJ9XX0) |
| `protect-agent-config` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoicHJvdGVjdC1hZ2VudC1jb25maWciLCJ2ZXJzaW9uIjoiMC43LjEiLCJzaGEyNTYiOiIxOTY0NDU3NTM2NWYyNTVhNjEyMzQ1NDJjNjc2YjNiZGYwMjhiZmQyMGE3YzIwNTczYWMyM2QzMGI1OTA4N2ExIn1dfQ) |
| `protect-ci-workflows` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoicHJvdGVjdC1jaS13b3JrZmxvd3MiLCJ2ZXJzaW9uIjoiMC4yLjYiLCJzaGEyNTYiOiIwZTUwY2U5NzhmYmE3OTlmM2RlNmViYTUyNGZkMjBiMWNlODZhZGQ5YzljMWY0YmQ2MTljYzk0ZmQ4YjNmMDM3In1dfQ) |
| `protect-commit-privacy` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoicHJvdGVjdC1jb21taXQtcHJpdmFjeSIsInZlcnNpb24iOiIwLjIuNSIsInNoYTI1NiI6IjMzMTQyOTViYWUxYTRiNzVkNTAzZDk3N2M4NDQ5ZTg5OGE4MTU5MWM1Y2ExZTk4OTY4YzlmNzg5OTAyZmM3MDEifV19) |
| `protect-main-branch` | advisory | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoicHJvdGVjdC1tYWluLWJyYW5jaCIsInZlcnNpb24iOiIwLjAuMyIsInNoYTI1NiI6ImViYTE2NGQwOWMyMmI0NmY1YzJkMjY1NjYwN2Q3MzU5NmQzYjBjZTRjNzc4MTg3MzdlZmM1MjcyZDc4NzE5ZWUifV19) |
| `protect-test-integrity` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoicHJvdGVjdC10ZXN0LWludGVncml0eSIsInZlcnNpb24iOiIwLjAuMyIsInNoYTI1NiI6ImU1MzYwZDZlOTMyMDU0Y2I2ZTM3ZTc3ZWRhM2JlNmU4MWVkOTAwMGZmNTQxYjBiZGQzZjJlZTkyNTVkMmU5MmYifV19) |
| `rtk-dangerous-actions-blocker` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoicnRrLWRhbmdlcm91cy1hY3Rpb25zLWJsb2NrZXIiLCJ2ZXJzaW9uIjoiMC4zLjAiLCJzaGEyNTYiOiIxNDkwOTc5NjZkMmM4YmRiMmUxNTdhNDRiM2Y4N2JhOTRlYjk1YjRhYzdmNzI0MjQwZTUyNTJkZDU2ZjQ4ZWIxIn1dfQ) |
| `scan-secrets` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoic2Nhbi1zZWNyZXRzIiwidmVyc2lvbiI6IjAuMS4yIiwic2hhMjU2IjoiODBjMjdiNmNlMmYxNDU3MWIxYzJhMWZhMjNlN2Y3MWUzMDVjYTNlMjFmNzYwYjM1NTc1MDlkZmQ0MzA1NjBkMCJ9XX0) |
| `token-efficiency` | advisory | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoidG9rZW4tZWZmaWNpZW5jeSIsInZlcnNpb24iOiIwLjEuMSIsInNoYTI1NiI6IjE5MDAzZmFmYWEyYTQ0ODM2ZGE1NDg2ZjlmNTI4YTY3YTc1YzQ0YjJiNDkyMmUyNjQ0NTJlNTkyOWI1YTFjNzYifV19) |
| `verify-dependency-exists` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoidmVyaWZ5LWRlcGVuZGVuY3ktZXhpc3RzIiwidmVyc2lvbiI6IjAuMi4wIiwic2hhMjU2IjoiYTk4NjNmZjQ4ZTI0OWYwN2E2MGE1NmYyYjRjZWJhY2YyNGI3YzU0MTE3M2M5MDliZTg0ZjZmNjk5MTc4ODk1NSJ9XX0) |
| `verify-mcp-allowlist` | enforces | [open in the builder](https://chock.sh/coding/builder/#s=eyJzY2hlbWEiOjIsImNsaWVudCI6ImNvZGV4IiwiYnVuZGxlIjp7InZlcnNpb24iOiIxLjAuMCJ9LCJjYXRhbG9nIjp7InNvdXJjZSI6Imh0dHBzOi8vZ2l0aHViLmNvbS9vcGVuLWNvZGVyLWFpL2Nob2NrLWNhdGFsb2ciLCJyZWYiOiJlODg1ZjkxOGEzMzVlMjBkZmRkYjRmYWQwMzMxZDRkNWI0YzVlYjQzIn0sInBvbGljaWVzIjpbeyJmcm9tIjoiY2F0YWxvZyIsImlkIjoidmVyaWZ5LW1jcC1hbGxvd2xpc3QiLCJ2ZXJzaW9uIjoiMC41LjMiLCJzaGEyNTYiOiIwNDIxNjg4OWE2MDEyYjJlNTgyMmRhYjgxZGY1ODYzNmEwN2I0ODdjMjBkMWZmYjgwMjNiNzBlMThiNGNlMWE0In1dfQ) |

## Generated from chock-catalog

Every file here is compiled from policy sources in
[chock-catalog](https://github.com/open-coder-ai/chock-catalog) by
[chock](https://github.com/open-coder-ai/chock). Pull requests against this repository are
closed automatically — open them against the catalog instead.

- **Generated only:** CI regenerates from the pinned catalog and fails on any difference.
- **Byte-identical guards:** each guard script is a verbatim copy of its policy's source in the
  catalog, and the hook adapter a verbatim copy of its framework source.
- **Best-effort, not a boundary:** guards are pattern-based filters; see
  [SECURITY.md](https://github.com/open-coder-ai/chock/blob/main/SECURITY.md).
- **Tested upstream, and gated:** every policy ships an eval suite
  (`base/<policy>/evals/suite.yaml`) in the catalog, and the publish workflow runs
  `chock check` and `chock check --only evals` before packaging anything — a policy whose
  evals fail cannot reach this repository. The tests live in the catalog because the policy
  source does; this repository is compiled output.
- This README is hand-written, as are `SECURITY.md` and the workflows under `.github/`, so they
  sit outside the generated-only guarantee.

### Verify it yourself

Nothing above asks for trust that cannot be checked. This rebuilds the published tree from
source and compares it with what is committed here:

```bash
git clone https://github.com/open-coder-ai/chock-codex-plugins dist
git clone https://github.com/open-coder-ai/chock-catalog catalog
git clone --branch "$(tr -d '[:space:]' < catalog/.framework-ref)" \
  https://github.com/open-coder-ai/chock framework
pip install ./framework
chock plugin build --repo catalog --policies-dir base --format codex --out-dir dist
chock marketplace build --dist dist --tree codex
git -C dist diff --exit-code && git -C dist status --porcelain
```

Silence from both `git` commands means this repository is byte-identical to a fresh build
from the catalog. The framework ref comes from the catalog's own `.framework-ref`, which is
what the publish and Generated-only workflows read, so this recipe cannot drift from the
release a tree was actually built with.
`chock-market.lock` records a sha256 per published plugin directory, so one package can be
checked without rebuilding the rest.

**If you are listing these plugins in a marketplace,** pin both a tag and the full commit
SHA. The tag names the release; the SHA is what holds the reviewed bytes still.

## Contributing

| You want to | Go to |
| :--- | :--- |
| Fix or add a policy | [chock-catalog](https://github.com/open-coder-ai/chock-catalog/blob/main/CONTRIBUTING.md) — it reaches every client from there, including this one |
| Report that a guard did or did not block on your Codex version | an issue on [chock](https://github.com/open-coder-ai/chock/issues/new/choose), which records the witnessed-blocking claims these packages carry; "it fails open where you say it fails closed" is the most useful result you can send |
| Report a bug in how packages are generated | [chock](https://github.com/open-coder-ai/chock/issues/new/choose), where the emitter lives |
| Fix this README | here — it is hand-written, not generated |

## Part of open-coder-ai

| | |
|---|---|
| [agentseam](https://github.com/open-coder-ai/agentseam) | the primitives — one handler API and a verified capability matrix across 16 agents |
| [chock](https://github.com/open-coder-ai/chock) | the compiler — one policy into git hooks, CI gates and native pre-tool hooks |
| [chock-catalog](https://github.com/open-coder-ai/chock-catalog) | the policies — 42, each labelled enforced or advisory, with replayed evals |
| [context-report](https://github.com/open-coder-ai/context-report) | the evidence — a signed report of whether an agent artifact actually works |
| [chock-threat-intel](https://github.com/open-coder-ai/chock-threat-intel) | the threat ledger the catalog's policies answer to |
| chock-{claude,cursor,copilot,codex}-plugins | the catalog, packaged for each agent's plugin format (generated) |
| chock-quickstart · chock-example | template repos: what `chock init` leaves behind, and a full adoption |

## License

Apache-2.0, same as the framework and the catalog.
