# Orch rules for the loomyard hub

## Production

An exception to `CLAUDE.md`'s Production rule: after your own pushed fix, on a clean `main`, you run `update-plugins.sh` yourself and say so in your report.

## Watching runs

The operator watches each run in its own terminal window, never in VS Code.
VS Code opens only when the operator asks for it, through `lyx ide` or `batten.yaml`'s `open_ide`.

## A review segment that already converged

When a Bouncer whose round already converged stops on something after the verdict, such as a failed commit, and its bounce budget runs out, never take the `goto` the error suggests: a goto gives the whole segment a fresh budget and runs a new review round over an approved artifact (`ubuntu-2404-support`, 2026-10-19).
Settle the stop with `lyx loom circling accept <slug>` and `lyx loom resume` in the task worktree, so the Bouncer settles on the verdict it has; when accept refuses, ask the operator before any goto.
