```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Behaviours and the tests that pin them:**

1. **The review row roots its overlay snapshot at `Env.AnchorPath`** (`entries_burler.go:59`).
   - Pinned by `TestBurlerReviewEntry_ProfileMapping/OverlayTargetRecordsTheRoundSnapshot` (`entries_burlerhalves_test.go:245-285`).
   - The fixture's anchor and worktree are separate directories (`fixture_test.go:273-275`), so on the old code the id hashes empty input and the test fails.
2. **The Bouncer's post-fix id reads through the same root.** `entries_bouncer.go:312` uses the shared `reviewTargetSnapshot`.
   - Covered by the same row through the shared helper. No Bouncer-specific row exists, and none is needed.
3. **An overlay id over no existing file is an error from `ID` and `Take`** (`targetsnapshot.go:114-116`).
   - Pinned by `TestTargetSnapshot/an overlay target with no file is an error` (`targetsnapshot_test.go:87-97`), which fails on the old code.
4. **A missing path beside an existing file is still skipped.**
   - Pinned by the existing overlay rows of `TestTargetSnapshot` (not re-read; the helper's paths include missing ones).
5. **A round whose start id and post-fix id are both empty counts as changed.** This is the path the diff makes reachable: the commit and the summary claim "a missing target can never produce a stalemate", and that claim rests on the `record.TargetID == ""` guard in `bouncerstalemate.go:55` and `bouncerverify.go:185`.
   - **None.** `bouncer_stalemate_test.go:169-170` has `RoundWithNoRecordCountsAsChanged` and `RoundWithNoPostFixIdCountsAsChanged`, but the second's target is `T0`.
   - No row has both ids empty, and no row has a snapshot whose id read fails.
6. **The review row records no target id when `Take` fails** (`burlerreview.go:230-232`). Round 2 then writes no changes file (`roundcontext.go:298`).
   - **None.** `roundsnapshot_test.go:49-51` covers an accepted id, a kept first id and a nil snapshot, but not a snapshot whose `Take` errors.

**Findings:**

- **MEDIUM**
  - Class: `design` (test economy: uncovered behaviour).
  - Location: `internal/shedadapters/bouncer_stalemate_test.go:169-170`.
  - Summary: no row pins that a round with no target id and no post-fix id counts as changed. With this diff, that is the only thing keeping a missing overlay target from reading as a stalemate.
  - Evidence: deleting the `record.TargetID == ""` guard at `bouncerstalemate.go:55` makes two empty ids equal, so the round reads as unchanged, and no existing row fails.
  - The same gap holds for `fixLeftTargetUnchanged` at `bouncerverify.go:185`. Its unchanged-target row only uses `tree-1`/`tree-1` (`bouncer_verify_test.go:173`).
  - Fix: add a stalemate row with `earlier: []stalemateRound{{}}` (both ids empty) that wants `shedengine.Stuck`. Better still, drive a fake snapshot whose `SourceTreeID` returns an error.
- **LOW**
  - Class: `design` (test economy: uncovered behaviour).
  - Location: `internal/shedadapters/roundsnapshot_test.go:49-51`.
  - Summary: no row pins that a failed `Take` at round start leaves the record with an empty `target_id`, while the round goes on and still writes its record.
  - Evidence: `burlerreview.go:230-232`; the table rows at `roundsnapshot_test.go:49-51`.
  - Fix: add a row whose overlay snapshot paths match no file, wanting a written record with `TargetID == ""`.

Tool calls:
1. Read `roundsnapshot_test.go:40-110` and grepped `bouncer_verify_test.go` for target-id rows. My context held neither table, and I needed both to check rows 5 and 6.
