# Orch rules for the loomyard hub

## Production

An exception to `CLAUDE.md`'s Production rule: after your own pushed fix, on a clean `main`, you run `update-plugins.sh` yourself and say so in your report.

## Watching runs

The operator watches each run in its own terminal window, never in VS Code.
VS Code opens only when the operator asks for it, through `lyx ide` or `batten.yaml`'s `open_ide`.

## A review segment that already converged

When a Bouncer whose round already converged stops on something after the verdict, such as a failed commit, and its bounce budget runs out, never take the `goto` the error suggests: a goto gives the whole segment a fresh budget and runs a new review round over an approved artifact (`ubuntu-2404-support`, 2026-10-19).
Settle the stop with `lyx loom circling accept <slug>` and `lyx loom resume` in the task worktree, so the Bouncer settles on the verdict it has; when accept refuses, ask the operator before any goto.

## Intake only on the operator's word

This overrides every intake step in your role: never run `lyx board intake` at a wake, a landing or any other time on your own, and never start an intake fork on your own.
Run intake only when the operator asks for it.
Selfreport is for LoomYard bugs only; an issue in any other repository is filed to that repository's own issues, never through selfreport.

## No parent reviews

This overrides your role's Parent-review section: you never review a parent-review or PR-Gate request yourself or through a fork, even when the request asks for a one-shot fork; the alignment agent is the dedicated reviewer.
When the alignment agent escalates, read only the brief's "Why it did not settle the review", put that question to the operator in plain words, and submit the operator's answer.

## Only your own hub

You start runs only in this hub; a run in another hub, such as `quarry-LYXHUB`, is started by that hub's own orch.
Setting up another hub, or writing its board, happens only when the operator asks.

## No changes in the prime

Never edit, commit or push code or docs directly in the prime worktree: a change goes through a task worktree, so the prime is never dirty when a batten run starts or a run lands.
Board, config and orch-rules writes through `lyx` are not prime edits.

## Say worktree

To the operator, a fabric checkout is one worktree: say "worktree", never "pair".

## No status polling

Never check runs with `lyx batten status` on your own; runs are watched through their notices, and a command's own output says what it did.
When a status read is needed, filter it to the fields you need, never the full JSON with its history.
