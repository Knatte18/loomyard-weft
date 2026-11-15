```yaml
lens: code-tests
outcome: reported
```

Task: lens code-tests. For each behaviour the diff adds or changes, I name the test row that pins it, or none.

**Per-behaviour list**

| Behaviour | Pinning test | Status |
|---|---|---|
| `clearDoneRuns` skips a run that is not done | `TestClearDoneRuns/RunningRunIsNotCleared` | pinned |
| Skip when `DriverLockHeld` | `.../DoneRunWithItsDriverLockHeldIsNotCleared` | pinned |
| Skip when a pass is in flight (`d.flights`) | `.../DoneRunWithAPassInFlightIsNotCleared` | pinned |
| Skip while a landing notice is remembered (`current.LandNotices`) | `.../DoneRunWithALandingNoticeRememberedIsNotCleared` | pinned |
| Skip when the run is in `d.landed` and `CountedLandings` lacks it | `.../DoneRunWithAnUncountedLandingIsNotCleared` | pinned |
| A done run with no landing is cleared, the seam is called and the log line is written | `.../DoneRunIsCleared` | pinned |
| A cleared run's `CountedLandings` entry is deleted and other slugs' entries stay | `.../DoneRunWithACountedLandingIsClearedAndLeavesTheCount`, plus the `"other"` assertion in every row | pinned |
| The seam returns false: no log line, count kept | `.../DoneRunTheClearLeavesAloneKeepsItsCount` | pinned |
| The seam errors: count kept, no log line | `.../DoneRunWhoseClearFailsKeepsItsCount` | pinned |
| `battencli.clearDoneRun` with `admitsAnyRun`: a done darn run is removed, committed and pushed | `TestClearDoneRun_RemovesADoneRunTheRecipeFilterAdmits/AnyRecipe_DoneDarnRunIsRemovedCommittedAndPushed` | pinned |
| `admitsAnyRun`: a done run with no child-recipe record is removed | `.../AnyRecipe_DoneRunWithNoChildRecipeRecordIsRemoved` | pinned |
| `restoreDoneRunRecords` with an empty recipe: the status is written back and no child-recipe file is created | `.../AnyRecipe_RecordsCommitFailureReturnsTheErrorAndKeepsTheRun`, plus the new `ReadChildRecipe` assertion | pinned |
| `restoreDoneRunRecords` with a recipe and the filing mark | `TestClearDoneRun_FinishesTheRemovalOfARunWhoseRecordsCommitFailed`, plus the existing commit-failure rows | pinned |
| `clearDoneRun` leaves a running run alone | `.../AnyRecipe_RunningDarnRunStays` | pinned |
| `clearDoneRun` leaves a run alone when its run lock is held | `.../AnyRecipe_DoneDarnRunWhoseRunLockIsHeldStays` | pinned |
| `admitsMotifRun` still filters (non-motif stays) | the existing motif rows (the first rows of the table, outside this diff) | pinned, from the diff's context |

**Findings**

1. **LOW / scope** — `internal/dobbyengine/loop.go:697`. The tick wiring has no test.
   - The call `d.clearDoneRuns(now)` sits inside `if scheduling`, after `countLandings`.
   - The new test calls `clearDoneRuns` directly. No row fails if the call is deleted, or runs while an anchor change or the adopt hold stands.
   - The `doc.go` claim "Each tick with no anchor change standing" is therefore unpinned.

2. **LOW / scope** — `internal/dobbyengine/tidy.go:117`. The assignment `d.landed = landed` has no test.
   - `TestClearDoneRuns` sets `d.landed` by hand.
   - If the assignment were removed, the uncounted-landing skip would stop working in production, and every row would still pass.
   - A test that runs `countLandings` and then `clearDoneRuns` over a fake `Deps.Landings` would pin the link.

3. **LOW / scope** — `internal/dobbyengine/donerun.go:28`. The nil `Deps.ClearDoneRun` early return has no row.
   - The doc says "A nil ClearDoneRun clears nothing".
   - Other tests that build a `daemon` without the seam may cover it indirectly, but none of the diff's tests do.

4. **LOW / scope** — `internal/dobbyengine/donerun.go:29-33`. The unreadable state record returns early, and no row covers it.

5. **LOW / scope** — `internal/dobbyengine/donerun.go:57-67`. A failed `CountedLandings` update, which only warns, has no row.
   - The focus directive names this behaviour.
   - It is hard to provoke with a temp state path, so the finding is LOW.

6. **LOW / scope** — `internal/dobbyengine/donerun.go:47-48`. The "logged once per episode" claim for a failing clear is not asserted.
   - `doneRunClears.Fail` and `.Recover` go through the logger, and the test checks neither.
   - The "never fails the tick" claim is also not asserted. The function returns nothing, so the claim holds by signature.
   - The error row checks only the call, the log lines and the count.

7. **LOW / scope** — `internal/battencli/cleanup.go:197-201`. The commit-message branch for a run with no child recipe has no assertion.
   - The message is `batten: remove the done run of <slug>` for an empty recipe, and `... <recipe> run ...` otherwise.
   - The new rows assert commit and spawn counts through `wantCommits` and `wantSpawns`, as far as the diff shows. They do not assert the message text.
   - A regression to `"done  run"` would pass.

8. **LOW / scope** — `internal/battencli/cleanup.go:135-141`. The exported `ClearDoneRun` has no test.
   - Its `ResolveBattenPrime` refusal and prime resolution are untested. Only the unexported `clearDoneRun` is driven.
   - `ClearMotifRun` follows the same pattern, so this is minor.

9. **LOW / scope** — `internal/beamcli/dobbydeps.go:177`. The `ClearDoneRun` seam wiring has no test.
   - The directive asks whether any `Deps` construction is left with a nil seam it needed. The wiring is thin, and nothing asserts it is filled.

Findings 1 and 2 are the strongest coverage gaps: if either call were removed, the whole suite would stay green.

**Tool calls:** none. The inherited diff and files in my context were enough.
