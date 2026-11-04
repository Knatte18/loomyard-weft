---
verdict: APPROVED
---

No findings.
Each focus question is settled below, read against the diff `263b2ca1..HEAD` and the amended board entry.

## Focus verdicts

- **`answerReopen` on a merged pull request (`internal/landingshed/prreview.go`).**
  Not a defect.
  A `StateMerged` status takes no `SetDraft`, `Comment` or `Close` call, because only the `StateOpen` cases make them.
  The record is then removed by the `req.Close || Closed || Merged` branch, and the row reroutes with the reason "pull request #N is already merged".
  The next landing finds no record and opens a new pull request.
  `TestPRReview_Reopen/APullRequestMergedOnGitHubLosesItsRecordSoTheReopenedWorkGetsANewOne` pins `wantCalls: ["Status 7"]` and `wantNoRecord`.
- **`TestPRReview_Reopen` and the pr-reviewer's stop.**
  Not a defect.
  Rows with a recorded strand mark it live and assert `wantStops: 1`.
  The failing-stop row sets `stopErr`, expects an error, expects no GitHub call and expects `wantRecord: reviewed` unchanged.
  Its `wantStops: 2` counts the row's own stop attempt plus `endSession`'s attempt on the error path.
- **`RemoveReady` and the `Unready` closure.**
  Not a defect.
  - Slug check: `TestRemoveReady/RecordOfAnotherBeamIsLeftStanding`.
  - Absent record: `AbsentRecordIsNotRemovedAndNothingIsCreated`.
  - Commit of the removal and a repeated call with no record left: `TestHubLandDeps_UnreadyCommitsTheRemovalOfTheReadyRecordAndSucceedsAgain`.
  - Empty pathspec: `Unready` probes `fabricengine.AnchoredPathHasContent` first and returns without committing when the landing directory holds nothing git accepts as a pathspec.
    The probe has its own test in `internal/fabricengine/anchoredpathhascontent_test.go`.
- **`readReopenRequest` (`internal/loomcli/landingdeps.go`).**
  Not a defect.
  `TestReadReopenRequest` has one row each for no request file, a request naming another run, a missing seed, a matching request, and a matching request carrying `Close`.
- **Beam-Await's `answerReopen` (`internal/pawlengine/beamawait.go`).**
  Not a defect.
  When `RemoveReopenRequest` fails for a request naming this run, `enter` returns false and `entered` stays false.
  The next wake retries the status write and the removal, and nothing starts the dobby.
  One notice is posted, guarded by `reopenStuckNoticed`.
  A request of another run whose removal fails does not hold the entry.
  Both are covered by the two "CannotBeRemoved" rows in `TestBeamAwait`.

## Other checks

- Scope against the amended entry: the verb refuses outside Parent-Align, Beam-Describe and PR-Review, keeps the pull request as a draft with a comment or closes it under `--close`, removes the ready record, and sends the run back to Beam-Await.
  Part 2 items 1 and 2 are present in the earlier commits of the range.
- The pawl recipe's new `on_reroute: Beam-Await` edge is asserted in `TestWire_BeamRecipe`.
- `docs/overview.md` and the package docs carry the new verb.
- No commit in the range carries a `Pattern-Departure:` trailer, and I found no `PATTERN` entry the diff violates.
