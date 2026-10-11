# Batten notices: no attach clause, pair-relative paths, waits not errors

A batten notice lands in the orch session, which re-reads its context every turn, so each notice is trimmed to what the orch acts on.

- **No attach clause.**
  `renderNotice` no longer ends a notice with `; cd <pair> && lyx reed attach`: the orch knows how to attach to a pair.
- **Pair-relative paths.**
  Every path under the task pair, in the condition, the child's reason and the stop report clause, is rendered relative to the pair, since the slug already names it, for example `report _lyx/shed/<slug>/drive-reports/drive-report-….md`.
  The `InnerRunDeps.AttachDir` seam, now used only for this, is renamed `PairDir`; the wiring in `battencli` and its tests follow the rename.
- **A wait is not an error.**
  The child's `Status.Error` is labelled `error:` only for a failed or blocked child, and `waiting:` for any other state, so an awaiting PR-Gate reads as `waiting: the pull request awaits review`.
- **The condition says what happened.**
  A state-changed notice names the child's state and the row it stopped at, for example `[batten notice] board-config-per-worktree: awaiting at PR-Gate; waiting: the pull request awaits review; since …; history 7; report …`, replacing `child left running; child state awaiting, error: …`.
  The driver-busy notice leads with the same state text, since its child is halted too.

The `battenshed` package doc describes the new notice shape, and `TestRenderNotice` pins the labels, the condition text, the relative paths and the absent attach clause.
