Lens fix-proposal.

1. LOW: F6's `hubStopWorktrees` dropped a pair whose `_lym` junction is not wired, with no log and no failure; the stop could leave such a pair's reed session running and report success.
   The review's fix text did not say to skip unwired pairs.
   F1, F2, F3, F4, F5, F7, F8 and F9 held up when challenged.

Tool calls: one grep of `ResolveWorktree`, `resolveCore` and `resolveBeamTarget`, to see that `AnchorRel` is populated for a located prime.
