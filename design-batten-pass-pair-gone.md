# batten pass fails forever when a run's task worktree is gone

`batten-pass-pair-gone` · Note · bug, batten, fabric

Imported from [issue #9](https://github.com/Knatte18/loomyard/issues/9).

## Seen

In the NORCE Models hub, the beam project run `loomyard-setup` stands at batten row 6, `Run-Shed`. Worktree-Create finished on 2026-10-30.
Its task worktree, `…/Models-LYMHUB/loomyard-setup`, no longer exists, and `lym fabric pairs` lists only the prime.
Every dobby pass on the run now fails the same way, three times in a row so far:

```
battencli: origin reconcile of "loomyard-setup" failed: fabricengine: origin reconcile of run loomyard-setup: no pair worktree /home/…/Models-LYMHUB/loomyard-setup
```

The run's status shows `state: running` with an empty `error` and `hand_back: false`. Dobby only reports the repeated failure to the orch.

## Expected

A missing task worktree should not make the pass fail without end.
Batten (or the origin reconcile) should recreate or adopt the pair from its branches when they still exist, the way `lym fabric add` adopts an existing weft branch.
If it cannot, it should hand the run back with a stop that names the recovery, not leave it running and retry.

## Workaround

The orch recreates the worktree by hand with `lym fabric add loomyard-setup`, then runs `lym batten run loomyard-setup`.
