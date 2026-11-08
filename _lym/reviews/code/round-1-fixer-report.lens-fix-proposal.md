```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed. Those are F2 (log and return nil) and F3 (one landing-deps test case).

**F2 (`internal/landingshed/prreview.go`, `askParentReview`)**
- Proposal: log the failed `updateRecord` with Warn and return nil, so the driver does not re-step and re-post.
- The concrete case I tried is a post that succeeds and a persistent record-write failure. The row now enters the wait with `ParentReviewAsked` unset, and every later entry (a resume or a re-step for another cause) posts the same request again. Before the fix, the error made the very next step post it again. The Warn and the updated doc comment ("the row's next entry asks again") state this, so the change is sound and I found nothing it breaks.
- I did not find a case where the nil return hides a wrong result. The wait re-reads the record itself, so a vanished record still errors there.

**Finding 1, LOW. The F2 test row is missing.**
- Changed text: the F2 fix in `prreview.go`, with no change in `prreview_session_test.go`.
- The review's `fix` key said to add a row for a record write that fails after a successful post, or to keep the error and add a row asserting it. Neither was added.
- Evidence: the two commits touch only `prreview.go`, `doc.go`, `overview.md` and `landingdeps_test.go`.
- Nothing pins the single-ask behavior, so reverting to `return p.updateRecord(...)` leaves every test green.
- This is a coverage gap the review named. If the fixer cannot make a write fail through the fixture, the fixer-report must defer it with that reason.

**Finding 2, LOW. A test case name says "hub layer" but writes the worktree layer.**
- Changed text: `TestPullRequestDeps_ParentReviewWantedReadsThePairsLayers`, case "the hub layer set to true reads true".
- `seedLoomConfigWithKeys(t, loc.AnchorPath(), ...)` writes the anchor's `_lym/config/loom.yaml` (`wiring_test.go:870`). That is the worktree layer, not the hub board directory.
- The name overstates what is pinned. A closure that read only the hub layer would still pass the local-layer case and fail this one only by accident.
- The third case's `hubValue: "false"` is redundant with the template default.

**Not found:** the local-layer case would fail if the closure read the wrong layer, so the F3 proposal pins what it was meant to. The `AskParent` closure stays untested, but the review's `fix` key asked only for `ParentReviewWanted`.

Further tool calls: none. My context held every file I needed.
