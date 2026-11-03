```yaml
lens: coverage
outcome: reported
```

Coverage review of the plan in `_lym/plan/` (overview and cards 1–9) against the decision record's decisions, acceptance criteria and the branches the cards add.

- **MEDIUM — D6 criterion, Card 7:** the accept path and the failing-store-then-resume path have no row that puts the record on the entry.
  - The D6 criterion names a judged round, an accept and a zero-round settle that each leave the record on the entry, plus a failing store that halts and a resume that stores. Card 7 argues this from existing shedadapters tests listed under `Uses`.
  - Those tests drive a fake `Approve` and assert only call order. `TestSequence_FullRunBlocksAtMergeIn` gets one counter assertion over a single path.
  - No planned row drives an accept settle or a failing `ApproveDiscussion` followed by a resume. No row reaches the real store either.
  - Evidence: Card 7 Intent ("its tests in `Uses` pin that order"), `internal/shedadapters#TestBouncer_Skip_FailingApproveSkipsCommit`.
- **MEDIUM — D6 wiring, Card 7:** nothing tests the `ApproveDiscussion` closure in `loomCLI.wire`.
  - The closure reads `loomengine.DiscussionDecisionRecord(location)` and stores for `shedrun.ResolveRunID(location, c.runID)`.
  - `TestStoreDecisionRecord_Real` calls `storeDecisionRecord` directly, and `TestWire_EverySeamFilled` checks only that the seam is non-nil.
  - So a closure that read the wrong file or used the wrong slug passes every row. Its read-error and store-error returns are reached by none.
  - Evidence: Card 7 Wiring paragraph and `Tests` list.
- **MEDIUM — D8, Card 8:** the darn and beam rows cannot fail if `preRun` wires `pastApproval` wrongly.
  - `decisionFake` takes `pastApproval bool` directly, so these two rows are the same as the "inside Discussion-Review" row, just with a fake false.
  - The real link in `decisionState.preRun` (`PastDiscussionApproval(recipe, currentRow)` returning `past`) is reached by no test.
  - Evidence: Card 8 Tests ("`decisionFake` gains … `pastApproval bool`").
- **MEDIUM — Card 8 branch with no stated behaviour:** `copyRecordToBoard` does not say what it does when `pastApproval` returns an error.
  - The Intent covers only false → "" and true → copy. "`pastApproval`'s error leaves the refusal as it is" covers only the failed-commit refusal, not the copy.
  - No row reaches either case.
  - Evidence: Card 8 "The copy" and "Where it runs".
- **MEDIUM — Card 8 branches with no row:**
  - a committed record HEAD does not hold (`deps.committed` found false) → error plus `decisionWayStore`;
  - a `deps.committed` read error.
  - Evidence: Card 8 "The copy" bullet two, against its Tests list.
- **MEDIUM — Card 9 branches with no row:**
  - the parent-cannot-be-resolved refusal;
  - the committed-record-missing refusal ("run `lym fabric commit` … then re-run");
  - the status-read and recipe-read error refusals.
  - Evidence: Card 9 Callers, Refusals and The copy, against its Tests list.
- **LOW — Card 6 branch with no row:** "a DiscussionWrite row that hands to a row that is not a Bouncer answers false" is not reached. Every planned row uses an embedded recipe, and none has this shape.
  - Evidence: Card 6 Intent, paragraph 1, and `TestPastDiscussionApproval` rows.
- **LOW — Card 1 branch with no row:** `DecisionRecord` returning a read error other than not-exist has no row.
  - Also untested: a store onto an unflagged entry whose file already holds the same bytes. This must still set the flag, since the no-write branch requires `Discussed`.
  - Evidence: Card 1 `DecisionRecord` bullets and `TestStoreDecisionRecord` rows.
- **LOW — Card 2 branch with no row:** renaming a flagged entry that has no record file has no row. "An absent old file is no error" is the case of a flag without a file, left by an older binary's rename or a failed move. Existing rename tests reach absent files only on unflagged entries.
  - Evidence: Card 2 Intent, first step bullet.
- **LOW — operand consistency across cards 7 and 8–9:** the slugs come from different expressions.
  - Card 7 stores for `shedrun.ResolveRunID(location, c.runID)`, which D6 names as the slug `SetRunStatus` uses.
  - Cards 8 and 9 store for `seedSlug(target.WorktreeName)`.
  - The two agree only for `SelfRunID`. The overview's Shared Decisions says all three use "the run's worktree slug", which is not the operand D6 names.
  - Evidence: Card 7 Wiring, Card 8 "New deps", `00-overview.md` Shared Decisions bullet 1, `internal/shedrun/resolve.go:20`.

Tool calls: none; everything I needed was already in my context.
