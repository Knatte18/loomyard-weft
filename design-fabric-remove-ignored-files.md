# fabric remove deletes the whole worktree directory, ignored files included

`fabric-remove-ignored-files` · Task · bug, fabric · running · Darn-Write

Depends on: [`batten-remote-order`](design-batten-remote-order.md)

From [issue #21](https://github.com/Knatte18/loomyard/issues/21), first of its three faults; the batten half is task `batten-stray-dir-and-beam-mark`.

## What happened

Hub NORCE-DrillingAndWells/Models (C#), beam `loomyard-setup` after its landing.
`lym fabric remove loomyard-setup` succeeded (worktrees, branches, launchers, session), but left the worktree directory in place holding git-ignored build output: `src/csharp/NORCE.Models/<project>/obj/` under three projects.
With that directory present but no longer a git repository, every batten verb on the run then failed with "not a git repository".

## Expected

`fabric remove` deletes the whole worktree directory, ignored files (`obj/`, `bin/`, caches) included, once git has removed the worktree; the same holds for the weft sibling.
