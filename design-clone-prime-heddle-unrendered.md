# fabric clone leaves the prime without heddle's rendered files

`clone-prime-heddle-unrendered` · Task · bug, fabric, heddle · done

## Seen

The operator (2026-11-07): after `lym fabric clone`, heddle's files are not rendered in the prime worktree, while a worktree that fabric adds gets them.

In code the clone does tell heddle: `internal/fabriccli/clone.go` calls `fabricengine.NotifyWorktreeMade(l)` for the prime right after `WireJunctionsWith`, and `heddleengine.Seam.Made` renders every target.
Two things differ from the add path, which calls it last in `Topology.add`:

- The clone fires it before `configsync.ReconcileAll` writes the per-worktree module configs and before their commit, so a render that reads config may find none yet.
- The clone discards the warning text `NotifyWorktreeMade` returns, so a failed render shows only in the log at Warn and the operator is never told; the add path's handling is not checked.

Which of the two, or what else, leaves the prime unrendered is not confirmed; the clone's log holds the seam warning if the render failed.

## Wanted

A `lym fabric clone` leaves the prime with heddle's files rendered, as a worktree fabric adds has them.
A failed render on clone is printed with its way forward, `lym heddle render` in the prime, not only logged.
