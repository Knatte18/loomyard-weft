# Batten treats a stray non-worktree directory as a gone pair, and a beam landing accepts its own beam mark

`batten-stray-dir-and-beam-mark` · Task · bug, batten, pawl · running · Darn-Write

Depends on: [`batten-remote-order`](design-batten-remote-order.md)

From [issue #21](https://github.com/Knatte18/loomyard/issues/21), the batten half of its three faults; the `fabric remove` half is task `fabric-remove-ignored-files`.

## What happened

Hub NORCE-DrillingAndWells/Models (C#), beam `loomyard-setup` after its landing.

1. Worktree-Teardown refused the beam's own pair: "the pair of loomyard-setup carries the beam mark, so an interrupted promote of an older build left it and a run would restart the ended run inside a beam; way forward: run lym batten cancel loomyard-setup --remove". The pair was a real beam landing through its own pawl run, so the mark was expected, not a leftover.
2. `lym batten cancel loomyard-setup --remove` refused: "the work of loomyard-setup reached its parent; way forward: run lym batten run loomyard-setup to finish the landing", which loops back to step 1.
3. `lym fabric remove loomyard-setup` then left the worktree directory holding git-ignored build output.
4. With that directory present but no longer a git repository, `lym batten run`, `lym batten cancel` and `lym batten cleanup` all fail with "not a git repository", so the run stays `failed` at Worktree-Teardown with no lym way forward.

## Expected

- Batten verbs treat a directory that is no longer a linked worktree as a gone pair, as for a missing directory, which task `batten-remote-order` handles; this task extends that handling once it has landed.
- Worktree-Teardown of a beam's own landing run accepts the beam mark its own Beam-Up wrote.
- No pair of refusals points at each other: a way forward never names a verb that refuses with a way forward back to the first.
