```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the F8 fix, option (a), which I carried out as proposed. I looked for concrete cases where it goes wrong.

Proposals challenged: F8 (a), "every lym verb that loads the loom or tapestry config runs `hubreconcile.EnsureOrWarn` before that load".

1. MEDIUM. D5's added line ends "once the stamp names the running build, the call costs one stamp read".
   Summary: while the stamp stays stale, every verb that loads the loom or tapestry config reruns the full hub walk on each call, not only the start verbs.
   Evidence: `ensure` writes the stamp only when the walk did not skip (`internal/hubreconcile/walk.go:77-85`). A walk skips when any worktree is mid-merge (`walk.go:323-326`) or the board pull skips the hub-wide write (`walk.go:268-270`). Each rerun takes the hub lock, pulls the board in `PullThenCommitWritten` (`walk.go:264`) and reconciles every worktree. One worktree sitting in a conflict resolution therefore makes every such verb on the hub pay a board pull and a full walk until the merge ends. Under (a) that covers loom's per-step verbs and webster's and shuttle's wiring (`internal/webstercli/wiring.go`, `internal/shuttleparity/seats.go:178-182`), where today only start verbs pay it. The changed text does not mention this cost.

2. MEDIUM. The new D5 acceptance criterion and the rationale's "lets in-flight runs carry over with no operator step".
   Summary: with several in-flight runs, the F8 halt can still happen through the lock timeout that the new Bound line admits.
   Evidence: `DefaultLockWait = 2 * time.Minute` (`internal/hubreconcile/hubreconcile.go:16`). On a new build, the first verbs of several runs race for the hub lock (`walk.go:56-62`). The first one runs the walk, which includes a board pull and every worktree. The others wait, and if the walk takes longer than two minutes they get `LockTimeoutError`. `EnsureOrWarn` only warns (`hubreconcile.go:67-71`), so their load refuses under D4 and the run halts. That is the review's scenario of several running tasks on one hub. The criterion pins one verb only, so it passes while this case still halts. A waiter also blocks for up to two minutes where it took seconds before.

3. LOW. "Every lym verb that loads the loom or tapestry config".
   Summary: the set of verbs is broader than the decision suggests.
   Evidence: `configreg`'s module specs load loom and tapestry through `CheckLoad` (`internal/configreg/configreg.go:164,174`), which `configsync` uses. So the `lym config` verbs, including the `lym config reconcile --apply` that D4 names as a way forward, would each first run the full walk. Shuttle's manifest and CLI also reach the loader through `shuttleparity` (`internal/shuttlecli/manifest.go`, `internal/shuttlecli/cli.go`). I did not check whether those shuttle paths run on a hook's fast path, where a walk with a two-minute lock wait would be costly.

Tool calls beyond context:
- Grep for `DefaultLockWait` and the packages that call the loom and tapestry loaders: needed the lock wait value and the set of verbs (a) reaches.
- Grep in `configreg` and `shuttleparity` for the loader calls and their callers: needed to know which verbs load the config indirectly.
