# Implementer forks can push to the real hub remotes

`fork-real-remotes` · Note · bug, webster, fabric

## Seen

On 2026-10-10, `daemon-wakeups` batch 09's implementer fork ran `lyx fabric clone/add --help`, read the real hub's `_lyx/fabric/origin.json` and ran `git ls-remote` on the weft repo.
The card 14 fork's `lyx fabric add idle-a/b/c` pushed branches `idle-a`, `idle-b`, `idle-c` to `Knatte18/loomyard` and `idle-*-weft` to `Knatte18/loomyard-weft`.
Recovery refused the batch with `needs_fresh`; the parent reset it and recorded a sandbox rule as a decision.

## Idea

An implementer fork cannot reach the real hub's fabric or remotes: its stencil states the sandbox-hub rule (local bare repos as origins), and lyx refuses `lyx fabric` writes and pushes to the hub's own remotes from an implementer strand, the way the hub-wide config refuses writes from a task worktree.


## From issue #494

Imported from [issue #494](https://github.com/Knatte18/loomyard/issues/494).

An agent following a "scratch hub" procedure cloned the real weft remote and ran `lyx fabric add` three times.
Each call pushed its code and weft branches to the real origin, leaving stray branches on both the code and weft repositories.
Nothing in `lyx fabric add` output or `--help` made clear that it pushes, so the scratch hub was not isolated.

Suggested fix: document the push in `lyx fabric add --help`, and offer a flag or setting for a local-only add, or a way to point a scratch hub at a local bare remote.

Provenance: task `daemon-wakeups`; note `09-orch-held-injection-log.md`.
