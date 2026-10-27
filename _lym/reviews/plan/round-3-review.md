---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lym/plan/08-dobby-worktree-gone.md:40
    summary: >-
      the after-tick read error is logged through d.failures, which the tick Recover in loop.go resets on every successful tick, so the episode flaps or double-counts
    fix: >-
      In card 8's after-tick-read paragraph, replace "the loop logs it through `d.failures` as a tick error is logged" with "the loop logs it through a `logger.Episode` of its own, a new `stateRead` field of `daemon` beside `ownerRead` (the card already edits `daemon`), as `beamOwnedElsewhere` logs an unreadable owner; a read that does not error ends that episode with `Recover`". Keep "keeps the earlier `keptTidy` and goes on" as it is.
  - id: F2
    severity: LOW
    class: scope
    location: _lym/plan/08-dobby-worktree-gone.md:41
    summary: >-
      the branch where the after-tick read finds no record or an undecodable one, so keptTidy becomes nil with a Warn, has no named input and no TestRun_WorktreeGoneExits row reaches it
    fix: >-
      In card 8's `TestRun_WorktreeGoneExits` row list, add a row after the directory-replaced-state.json row: "a record holding a `Tidy` at start whose `state.json` tick 1's hook overwrites with bytes that are not JSON while `.git` stays present, sending no watch event, and the same `Deps.Sleep` wrapper removing the worktree on its first call: the undecodable record reads as absent, `keptTidy` becomes nil, and the fake recorded no `StopStrand`". Name this row in the paragraph's sentence on a failed decode, as the directory row is named for the read error.
---

### [LOW] The after-tick read error shares the tick's failure episode

**Location:** `_lym/plan/08-dobby-worktree-gone.md:40`

**Issue:** The card has the loop log a read error with the marker still present "through `d.failures` as a tick error is logged".
`loop.go` calls `d.failures.Fail` when `tick` errors and `d.failures.Recover` right after every tick that returns nil, before the card's read runs (`internal/dobbyengine/loop.go:221-225`).
`logger.Episode.Fail` logs the first failure of an episode at Warn and later ones at Debug, and `Recover` logs at Info and resets the count (`internal/logger/episode.go:21-38`).
So a read that fails while its tick succeeded gives, on every iteration, a Warn from the read's `Fail` and an Info from the next tick's `Recover`, which is not once per episode.
A read that fails together with a failing tick, as the directory-replaced `state.json` of the last row does from the next tick on, counts two failures per iteration in one `failed_ticks`.
Round 2 proposed the `d.failures` route and the fixer applied it as proposed (it was never disputed); this finding corrects that proposal against `Episode`'s Recover semantics rather than re-raising a disputed item.
Every other failure source of `daemon` has an episode of its own (`ownerRead`, `moveStands`, `landRead`, `landPost`), and the card's `daemon` edit has room for one more.

### [LOW] No row reaches the no-record or undecodable branch of the after-tick read

**Location:** `_lym/plan/08-dobby-worktree-gone.md:41`

**Issue:** The paragraph prescribes four outcomes for the after-tick read.
Marker gone is reached by the rows that remove the worktree inside a tick: the lock's directory is gone, so `lock.AcquireReadLock` fails with a missing path and the read errors without creating anything.
A read error with the marker present is reached by the directory-replaced `state.json` row: `os.ReadFile` on a directory fails with an error that is not `IsNotExist`, which `ReadJSONNoMkdir` returns as `ErrRead`.
A read that succeeds with `Tidy` set or cleared is reached by the rows where tick 1 adds or clears a `Tidy`.
The fourth, a read that finds no record or fails to decode, which "reads as no record" with one Warn and sets `keptTidy` to nil, has no named input in any row.
It cannot arise from the tick alone, since `tick` opens with `UpdateState`, which writes the record and replaces an undecodable one; it needs the tick hook to write the bytes, as the directory row does.
The branch changes what `exitWorktreeGone` stops, so it is a prescribed behavior with its own outcome.

## Rejected

- Focus 1, `TestRun_WorktreeGoneExits` row for a worktree removed after tick 1 by `Deps.Sleep`: not a defect. With the top-of-iteration check deleted, iteration 2 still runs `tick` (the wrapper advances no clock, so no probe is due, and `tick` does not need one), and its `UpdateState` recreates `.lym/beam` under the removed root. The post-tick check then sees a record read and `.git` absent and exits after tick 2. The row's `len(h.ticks) == 1` fails (2), and so does "the worktree path does not exist after `Run` returns" (the stub is there); `CheckBinary` is called once either way, so that assertion alone would not catch it. `stopAt` 3 leaves room for the second tick, since the hook cancels only on tick 3. Whether `Watch` wakes the loop (`woke` true) or `Sleep` returns first, the next iteration is the same.
- Focus 2, branch coverage of the after-tick read: marker gone with a failed read (the in-tick removal rows) and read error with marker present (the directory row) both have a named input and a named row, and the marker-gone discard is one code path whatever the read returned. The error handling's disagreement with `loop.go` and the unreached no-record branch are F1 and F2.
- Focus 3, card 11's Intent sentence on a hub `gate.yaml` seeded from a prime copy or legacy files: not a defect. It carries the case rather than a bound: the absent-file branch runs `m.MigrateHubBase` over the `sparseSeed` bytes, `Ensure` returning after one stamp read once the stamp names the running build is the stated reason (`internal/hubreconcile/hubreconcile.go:45-47`), and `TestReconcileHubWideAt_SeedsAdoptFromATunedFinalize` has a second row for the absent file seeded from a prime copy. The gate module has `OpenMaps`, so an absent `gate.yaml` does take the `SeedPrime` arm (`internal/configsync/configsync.go:900-905`). `SeedLegacy` is unreachable for `gate`, since `legacyConfigModules` lists only `fabric`; the Intent names both seeds as the decision record does, and one call site over both arms adds nothing to maintain, so it is left.
