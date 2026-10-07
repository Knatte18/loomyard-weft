# Run lifecycle waits: orch compact reload, a step watchdog, a pair create that retries

`lifecycle-waits` · Task · bug, enhancement, orch, shed, fabric

Three places where a run or the orch waits far longer than it needs to, or hangs unnoticed.

## Orch compact cycle times out before reload

A compact cycle never sees its own compaction finish, so it always waits out the handoff timeout and aborts.
Seen 2026-10-07 on the hub orch:

- 12:05:39 the watcher enters `compacting` and types `/compact`.
- 12:06:41 the compaction boundary is written to the transcript.
- 12:15:41 the phase ends with `last_abort_reason: "compaction timed out"`.
- The next idle turn end finds the boundary through the auto-compaction path (`CompactedSince`) and only then sends the reload, about ten minutes late.

Cause: `tickCompacting` (`internal/orchengine/watcher.go`) decides the compaction landed from the context reading of `st.ReadingTurnEnd`.
That field is written only in the idle phase (the `st.Phase == PhaseIdle && isTurnEnd(ev)` branch of the tick's event read), never while the phase is `compacting`.
So it stays on the turn end from before `/compact`, whose reading can never show `Compacted`, and the turn ends after the compaction are not read.

Side effects:

- The cycle counts as aborted: `cycle_count` does not move and `last_deferral` is set, which can push back the next automatic cycle.
- A `distill` request queued during the cycle is removed as stale (`orch: stale cycle request removed`) instead of being acted on.

Fix direction: in `compacting`, take each turn end read after the phase entry as the reading candidate, or ask the transcript for a boundary after `PhaseEnteredAt` directly, as the auto path does.
A test drives a compact cycle through a turn end that follows the boundary and asserts the reload starts without the timeout.

## A step that never returns

From run-recovery PR-Gate (2026-10-07): run-recovery gave `verifytree` a 60-minute timeout and kills the process group on Unix.
Still open from #420: a step-level watchdog, so a `lyx shed step` that never returns is noticed without the hub reading panes; and on Windows the timeout kills only the shell, not the process tree it started.


## From issue #433

Imported from [issue #433](https://github.com/Knatte18/loomyard/issues/433).

Seen 2026-10-07 when the hub started three batten runs at once (`cache-ttl-per-role`, `batcher-v2`, `burler-cost`).

- `cache-ttl-per-role`'s Worktree-Create pushed its weft branch at the same moment as the other two pairs and GitHub answered `! [remote rejected] cache-ttl-per-role-weft -> cache-ttl-per-role-weft (failed)`, with no hook message; the other two pushes went through.
- `rollbackAdd` then could not delete the warp branch: "warp-branch deletion was refused by the destructive gate (check=ownership); the branch is left behind". The branch was already pushed to origin.
- Resuming the run blocked on "branch already exists, left behind by an earlier pair", so the hub had to delete the branch locally and on the remote by hand; the next resume went through.

Wanted:

- Retry a weft push that GitHub rejects with a bare `(failed)` a few times with backoff before giving up; a bare `(failed)` reads as a transient server-side refusal.
- `rollbackAdd` removes the warp branch it created itself in the same call, locally and on the remote, instead of the ownership check refusing it.

🤖 Generated with [Claude Code](https://claude.com/claude-code)