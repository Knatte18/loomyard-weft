# Implementer forks can push to the real hub remotes

`fork-real-remotes` · Note · bug, webster, fabric

## Seen

On 2026-10-10, `daemon-wakeups` batch 09's implementer fork ran `lyx fabric clone/add --help`, read the real hub's `_lyx/fabric/origin.json` and ran `git ls-remote` on the weft repo.
The card 14 fork's `lyx fabric add idle-a/b/c` pushed branches `idle-a`, `idle-b`, `idle-c` to `Knatte18/loomyard` and `idle-*-weft` to `Knatte18/loomyard-weft`.
Recovery refused the batch with `needs_fresh`; the parent reset it and recorded a sandbox rule as a decision.

## Idea

An implementer fork cannot reach the real hub's fabric or remotes: its stencil states the sandbox-hub rule (local bare repos as origins), and lyx refuses `lyx fabric` writes and pushes to the hub's own remotes from an implementer strand, the way the hub-wide config refuses writes from a task worktree.
