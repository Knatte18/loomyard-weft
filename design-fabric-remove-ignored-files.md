# fabric remove leaves ignored files behind, and the half-removed directory then blocks every batten verb

`fabric-remove-ignored-files` · Task · bug, fabric, batten, pawl · draft

Depends on: [`batten-remote-order`](design-batten-remote-order.md)

Imported from [issue #21](https://github.com/Knatte18/loomyard/issues/21).

## What happened

Hub NORCE-DrillingAndWells/Models (C#), beam `loomyard-setup` after its landing.

1. Worktree-Teardown refused the beam's own pair: "the pair of loomyard-setup carries the beam mark, so an interrupted promote of an older build left it and a run would restart the ended run inside a beam; way forward: run lym batten cancel loomyard-setup --remove". The pair was a real beam landing through its own pawl run, so the mark was expected, not a leftover.
2. `lym batten cancel loomyard-setup --remove` refused: "the work of loomyard-setup reached its parent; way forward: run lym batten run loomyard-setup to finish the landing", which loops back to step 1.
3. `lym fabric remove loomyard-setup` succeeded (worktrees, branches, launchers, session), but left the worktree directory in place holding git-ignored build output: `src/csharp/NORCE.Models/<project>/obj/` under three projects.
4. With that directory present but no longer a git repository, `lym batten run`, `lym batten cancel` and `lym batten cleanup` all fail with "not a git repository", so the run stays `failed` at Worktree-Teardown with no lym way forward. The operator's rule is that worktrees are removed only through lym, so there is no sanctioned fix.

## Expected

- `fabric remove` deletes the whole worktree directory, ignored files (obj/, bin/, caches) included, once git has removed the worktree.
- batten verbs treat a directory that is no longer a linked worktree as a gone pair, as for a missing directory (see #9), not as an error.
- Worktree-Teardown of a beam's own landing run accepts the beam mark its own Beam-Up wrote.