# Card 16 — batten-start-guards

**Create:**
- `internal/battencli#startGuard`
- `internal/battencli#hubStartGuard`
- `internal/battencli#startGuard.refuse`
- `internal/battencli#battenCLI.refuseCancelMarker`
- `internal/battencli#TestStartGuard_Refuse`
- `internal/battencli#TestArmSeed_CancelMarkerRefusesTheWrite`
- `internal/battencli#TestArmAt_StartGuardRefusesBeforeAnyWrite`

**Edit:**
- `internal/battencli#battenCLI`
- `internal/battencli#battenCLI.armAt`
- `internal/battencli#battenCLI.armSeed`
- `internal/battencli#battenCLI.refuseStartOwnedElsewhere`
- `internal/battencli#TestRefuseStartOwnedElsewhere`

**Uses:**
- `internal/battencli#readCancelMarker`
- `internal/battencli#cancelMarkerRefusal`
- `internal/battencli#fakeOwners`
- `internal/battencli#takeCancelMarker`
- `internal/fabricengine#SourceDirtyPaths`
- `internal/fabricengine#Topology.PairParts`

**Intent:**
Batten's pre-run refuses, before it writes anything, a run a cancel is ending (D3), a dirty source (D10) and a start over a left local part of a kept pair (D5).

`startGuard` carries three seams: `Marker func() (cancelMarkerReading, error)`, `SourceDirty func() ([]string, error)` and `PairParts func() (fabricengine.PairParts, error)`; a nil seam checks nothing, which only a hand-populated receiver has.
`hubStartGuard` wires them to `readCancelMarker`, `fabricengine.SourceDirtyPaths(location)` and a topology over the hub's fabric config's `PairParts(location, slug)`, all local reads with no network call.
`battenCLI` gains the field `guard startGuard`, which `armAt` fills with `hubStartGuard` when its `Marker` is nil, as it fills the owner seams.

`startGuard.refuse`, in order:

1. For `run`, `step` and `goto`, a present marker refuses with `cancelMarkerRefusal`; every other verb skips it.
2. For `run` and `step` with no status file (`found` false), or with a status whose current producer is Worktree-Create while the task worktree is absent, tracked dirty paths refuse: `the source worktree has uncommitted tracked changes: <paths>; commit or revert them in the source worktree, then run "lyx batten <verb> <slug>" again` (PATTERN-no-denied-recovery); untracked files are never listed, so they never refuse.
   A resume past Worktree-Create, or at it with the task worktree present, runs no dirty check.
3. For `run` and `step` with no status file, a left task worktree, records worktree or local records branch refuses: `<slug> left a pair from an earlier run (<parts>); run "lyx batten cancel <slug> --remove" from the prime, which removes it, then "lyx batten <verb> <slug>"`.
   A task branch left alone does not refuse here: the add refuses it, and Worktree-Create halts on it with the precondition reason (card 10).
   Bound: a pair made by hand with `lyx fabric add` before the first `lyx batten run` is refused the same way.

A probe error refuses naming the probe and the verb's re-run.
`armAt`, for `run`, `step` and `goto`, reads the status file (`shedrun.StatusFile`, under its lock, creating the lock's directory as `checkRunStatus` does) and, for a found status at Worktree-Create, `taskWorktreePresent`, then calls `c.guard.refuse` right after setting the receiver's fields and before the start owner check, so before the board pull, the prime claim, `armSeed` and `wire`.
The `--window` path passes `armAt` in the persistent pre-run before its `RunE`, so it refuses before any window opens and the caller reads the refusal from its own call.

The Added-after-Discussion re-read: `refuseCancelMarker` reads `c.guard.Marker` and returns its refusal when present; `refuseStartOwnedElsewhere` calls it right after its successful `Pull` and before `claimPrimeOwner`, and `armSeed` calls it right before `WriteSeed`, so a start whose pull spanned a cancel writes neither the prime claim nor the seed.

`TestStartGuard_Refuse` is a table over fake seams: a present marker with its writer alive or gone refuses `run`, `step` and `goto` and passes `status` and `pause`; a dirty start refuses naming each path, `commit or revert` and the verb; a start with no dirty path passes; a resume at Worktree-Create with no task worktree refuses when dirty, and one with it present, or past the row, passes while dirty; each left part (task worktree, records worktree, records branch) refuses a start naming `lyx batten cancel some-slug --remove`, a task branch alone passes, and a resume with left parts passes; a seam error refuses.
`TestArmSeed_CancelMarkerRefusesTheWrite` drives `armSeed` with a `Marker` seam reading present on a temporary location: it refuses, writes no seed and registers no process; with the seam reading absent it seeds as before.
`TestArmAt_StartGuardRefusesBeforeAnyWrite` is a tier-1 integration test in package `battencli` over a hubforge hub, pinning `armAt`'s order and the criterion that `run`, plain and in a window, refuses with no seed, no claim and no window: with a tracked file of the prime modified, `armAt` for `run` on a receiver whose owner seams are a `fakeOwners` refuses naming the path and `commit or revert`, and the fake records no pull and no write, and no seed or status file exists; with the source clean and a marker taken by `takeCancelMarker`, `armAt` for `step` refuses with the cancel refusal and the same nothing-written assertions; and `RunCLIIn` from the prime with `run <slug> --window` on the dirty source refuses on the envelope naming the path, with no seed written, so no window was opened.
`TestRefuseStartOwnedElsewhere` gains a row where the marker reads absent before the pull and present after it: the start refuses with the cancel refusal and `fakeOwners` records the pull and no prime write.

**ImpactSummary:** `armAt` serves batten's own verbs and `ArmAt`/`ResumeInWindow`; the guard adds refusals only on a cancel marker, a dirty source at a start and left pair parts at a start, and hand-populated receivers keep a nil guard.
