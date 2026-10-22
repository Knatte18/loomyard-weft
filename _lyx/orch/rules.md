# Orch rules for the loomyard hub

## Production

An exception to `CLAUDE.md`'s Production rule: after your own pushed fix, on a clean `main`, you run `update-plugins.sh` yourself and say so in your report.

## Watching runs

The operator watches each run in its own terminal window, never in VS Code.
VS Code opens only when the operator asks for it, through `lyx ide` or `batten.yaml`'s `open_ide`.

## A review segment that already converged

When a Bouncer whose round already converged stops on something after the verdict, such as a failed commit, and its bounce budget runs out, never take the `goto` the error suggests: a goto gives the whole segment a fresh budget and runs a new review round over an approved artifact (`ubuntu-2404-support`, 2026-10-19).
Settle the stop with `lyx loom circling accept <slug>` and `lyx loom resume` in the task worktree, so the Bouncer settles on the verdict it has; when accept refuses, ask the operator before any goto.

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

## The orch note only at a cycle

Write the orch note only when a message asks for it at a context cycle; never update it between cycles as a running log (operator, 2026-10-22).

## Dead strands after a reboot (#799)

After a machine restart, the strands from before it stay in `lyx reed list` as dead and hold their names, so a resumed row can fail on a name clash.
When resuming runs after a restart, remove each dead strand of the run with `lyx reed remove --name <strand>` in its task worktree, then `lyx loom resume` a row that failed on the clash.
Remove this section once #799 is fixed.

## Loom only for bundles

Never propose one note alone as a loom task: a loom run has a fixed cost of about half its spend whatever its size (`docs/research/token-audit-2026-10-22`).
Fold related notes into one loom task, or propose darn for a small change, and let the operator name darn (operator, 2026-10-22).
