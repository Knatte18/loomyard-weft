---
verdict: APPROVED
prior:
  - id: F8
    status: resolved
    answer: "D5 now has every lym verb outside lym config that loads the loom or tapestry config run hubreconcile.EnsureOrWarn before the load; its rationale and Bound lines state the real walk timing, and an acceptance criterion pins the stale-stamp in-flight case."
---

## Round 3 review

No new finding.
F8 is resolved.
The reasons follow, each tied to the code read.

### D5 closes the window

- D5 chose option (a) of F8's fix: "Every lym verb outside `lym config` that loads the loom or tapestry config runs `hubreconcile.EnsureOrWarn` before that load".
  Today `internal/loomcli/arm.go` runs `EnsureOrWarn` for `start` and `resume` alone, so a non-start verb of an in-flight run loads its config with no walk.
  Under D5 the first such verb on a new build runs the walk, which moves the hub base, the tracked worktree files and the Seed-Child-written local files before the load.
  The Notes for the plan writer name the `arm` widening and the loaders `webstercli` and `shuttleparity` reach, which matches the loader sites found in the code.
- The cost premise matches `ensure` in `internal/hubreconcile/walk.go`.
  A stamp that names the running build returns after the stamp read and `repoDefaultAbsent`'s stat of each per-worktree module's repo default, with no lock and no git.
  D5's "the stamp read and board file checks" says the same.
- `ReconcileGeometry` reports false for a repository with no board-level lym dir, so a standalone repository never walks.
  That equals today's behavior for the start verbs, and D4's refusal names `lym config reconcile --apply` there.

### D5's Bound lines cover files the walk has not reached

The third Bound states that a file the walk has not moved keeps its old key until the next walk or the layer's reconcile command, and that the load refuses it under D4 meanwhile.
It names each case the code shows:

- a file gaining an old key after the walk;
- a worktree the walk skipped as mid-merge, which `reconcileWorktree` reports as skipped;
- an incomplete pair, which the walk passes over without a skip;
- `EnsureOrWarn` failing or waiting out `DefaultLockWait`, which `ensure` returns as `*LockTimeoutError`.

The fourth Bound states that a skipped worktree or a skipped hub-wide write leaves the stamp stale, so every such verb reruns the walk.
`ensure` does that: a skipped walk returns before `writeStamp`.

### D4's load refusal and D5's rationale match the call sites

- The rationale's premise is the true one: the walk runs before the first load on a new build, not before any command.
- `Ensure` callers outside `EnsureOrWarn` (`fabriccli` clone, `beamcli`, `adoptcli`, `battencli` wire) return their error and never go on; D5 names only `EnsureOrWarn` for the new verbs, which go on with the config as it is and meet D4's refusal if the walk failed.
- The re-run way forward changes `reconcileWayForward`, which the Notes name.

### The acceptance criterion pins the in-flight case

The D5 criterion "after a rebuild that leaves the build stamp stale, a non-start loom verb of an in-flight run whose hub base and local `loom.yaml` hold `code_max_bounces: 2` resolves `loom.code.max_bounces` to 2 and does not halt" pins the stale-stamp, in-flight case F8 named.
The failure paths that end in a D4 refusal are bounds, not criteria, which fits: the Goal's "no operator step" holds when the walk succeeds.
