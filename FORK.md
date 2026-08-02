# Fork notes — `crs2007/ha-mcp`

This is a fork of [`homeassistant-ai/ha-mcp`](https://github.com/homeassistant-ai/ha-mcp).

Unlike a plain mirror, **`master` here carries local commits**: it is upstream `master`
plus the features below, merged in so this fork can be run directly as a personal
distribution. Read the resync section before pulling upstream changes.

## What is baked into `master`

Each feature also lives on its own branch, which is the head of an upstream pull request.
Keep those branches alive and rebase them independently — they are how the work gets
merged upstream, and `master` is only where it is assembled to run locally.

| Feature | Branch | Upstream PR |
|---|---|---|
| `ha_intent_process` — routes a sentence through HA's Assist conversation pipeline (`POST /api/conversation/process`) | `feat/ha-intent-process-v2` | [#2108](https://github.com/homeassistant-ai/ha-mcp/pull/2108) |
| Structured summary mode for `ha_get_logs(source='error_log')` — deduplicated, component-grouped error parsing | `fix/ha-get-logs-structured-errors` | [#2107](https://github.com/homeassistant-ai/ha-mcp/pull/2107) |
| MCP Prompts layer — 12 prompts across safety / troubleshooting / automation / status / security | `feat/mcp-prompts-layer-v2` | [#2112](https://github.com/homeassistant-ai/ha-mcp/pull/2112) |

When an upstream PR merges, its feature arrives through the normal upstream sync and the
corresponding merge commit on `master` becomes redundant — resolve that in favour of
upstream and drop the branch.

## Resyncing with upstream

> [!WARNING]
> **Do not use GitHub's "Sync fork" button.** It offers a *Discard commits* option that
> resets `master` to upstream, which would delete every feature listed above. The button's
> fast-forward path also no longer applies, because `master` has diverged.

Sync locally instead:

```bash
git fetch upstream
git checkout master
git merge upstream/master      # a real merge, not a fast-forward
git push origin master
```

Then rebase each feature branch that has not yet merged upstream:

```bash
git checkout <branch>
git rebase upstream/master
git push --force-with-lease origin <branch>
```

Rebase the branches onto `upstream/master`, never onto this fork's `master` — otherwise the
upstream PR diff picks up the other features.

## Branch protection

`master` is protected with force-pushes and deletion blocked, and **no** required pull
request or status checks, so the local merge-and-push resync flow above works without a
bypass. Admins are not enforced, so protection can be lifted if a history rewrite is ever
genuinely needed.

## Local development notes

These are environment quirks on the maintainer's Windows machine, not repository problems —
all of the following pass on CI (Linux):

- Run pytest with `PYTHONUTF8=1`; the default Windows locale codec (cp1255) fails to decode
  UTF-8 skill files during collection.
- Pass `--timeout-method=thread`; `pytest-timeout`'s default signal method calls
  `signal.setitimer`, which does not exist on Windows.
- `mypy src/` reports 4 `fcntl` attribute errors in `utils/config_write_lock.py`, and
  `tests/src/unit/test_custom_component_filesystem.py` has 3 path-semantics failures. Both
  are pre-existing on clean upstream `master` and are Windows-only.
