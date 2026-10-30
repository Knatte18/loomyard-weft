```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run fix-code-correctness over the F1–F6 change set, using only the context already held.

Findings:

1. **LOW** — `closeRecordedPullRequest` in `internal/landingshed/prreview.go` (the added `stopLiveSession` block before `RemoveRecord`).
   - When `stopLiveSession(rec)` fails, for example `StopStrand` errors, the code logs a Warn and then removes the record anyway.
   - A session that was just found live is then left running with no record to find it by.
   - Neither `endSession` nor a re-entry can stop it later, so this recreates F2's premise on the failure path.
   - Evidence: `endSession` stops through `stopRecordedSession`, which returns `false, nil` when `ReadRecord` finds nothing.
   - Returning the error and keeping the record, as `endSession`'s caller contract allows, would let the re-entry retry. The review's fix asked only for a logged failure, so the LOW is optional.

I found no defects in the rest of the change set. Each step was traced from the code and tests already in context:

- **F1, `wake` reorder.** I traced the new `merged` skip with `record seed("h1")`, `head h2` and status `mergedAt("h2")`.
  - The pending entry status is returned by `pollStatus` because it is not yet due.
  - `merged` is true, so the gate is skipped.
  - `judge` finds `HeadSHA h2 != GatedHead h1` and returns Stuck naming `lym loom goto Finalize`.
  - The new row would fail with the defect in place: gating first would record `h2` as gated and end Done. It also asserts `verifyCalls == 0`.
  - The shadowed `outcome, pointer, ended, err :=` inside the gate block is scoped correctly, and the later `=` assignment at `judge` still compiles.
  - A closed pull request with the continue flag and an ungated clean head still gates first, so `ContinueOnAClosedPullRequestWithAnUngatedCleanHeadGatesItFirst` holds.
  - The edited mark row (approved at `h2`, record gated `h1`) runs `poll → gate → push → record gated h2`, and `sessionStopWanted` then reads the post-gate record, so it ends Done with the same marks.
- **F2 test.** With `mode off`, a record naming a live `g-live` and a merged status, `closeRecordedPullRequest` stops the strand and removes the record, then `endSession` finds nothing. The result is `stops=["g-live"]`, and the test would show 0 stops without the fix.
- **F3.** The `RegisterSession` override removes the record, so `updateRecord` returns `ErrNoRecord`, `StopStrand` runs once, and `endSession` finds no record. `starts=1, stops=1` holds, and the row would show 0 stops without the fix.
- **F4.** `LoadConfig(ConfigDirs(t.prime))` matches the call at `teardown.go:548`, and `BranchPrefix+slug` matches `add.go:277`. The only callers of `closeOpenPullRequest` are the two in `pullrequest.go` and the test, all updated. The gone-path Warn is now skipped when the config load fails, but no test calls `closePairPullRequest` directly. The row's `branch=warp/slug` and `may stay open` substrings match the new message.
- **F5** is a doc-only change and matches the code.
- **F6.** The row with `Continue` set and `MailedHead=h2`, so no gate runs, reaches `judge` with `!gated` and keeps waiting. Dropping the `!gated` half would add a `Close` call and end Done, which the row would catch.

Tool calls made: none; the context already held every changed file and test.
