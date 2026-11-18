```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed code under the fix-code-correctness lens.

I ran each changed branch by hand against concrete inputs and found no BLOCKING or MEDIUM defects. Two LOW findings follow.

**LOW — `judgesnapshot_test.go`, row `DecisionRecordRolledBackToAReplaceBeforeTheSnapshotIsNamed`**
- The row passes with the F1 defect in place.
- The fake seam itself returns `found` false when `replacedBeforeSnapshot` is set, so it plays the part of the filter that lives in `replacedDecisionRecord`.
- The engine never filters, so reverting the engine change would not fail the row.
- Only the `replacedSince` assertion, which expects `records-1`, pins what the engine does: it must pass the snapshot head.
- The real pin is `TestReplacedDecisionRecord_…` in `internal/loomcli/recordhistory_integration_test.go`.
  - With the after-check removed, the rows `AReplaceAtTheSinceHeadIsNotAfterIt` and `NoReplaceIsAfterTheTip` would return `historyY` and fail.
  - So the defect is covered, just not by the engine row.

**LOW — `verify.go`, `recordGuardHalt` default branch and `changedSince`**
- The default branch now records an unreturned session's halt with an empty `At`.
- Take a halt over a session that never returned, then a restore, then a resume that respawns the judge.
- If that judge returns with a fresh change, `judgeSessionChanged` calls `changedSince` with `returned` true and a non-nil `mark.GuardHalt`.
- The lead then reads "the resume re-checked the judge guard … restored since the last check", although this is a first check of a returned session.
- The `At` is set correctly by the `at == "" && returned` branch, and the wording is only slightly off.
- `restored since the last check` stays accurate, since the earlier changes are compared against the new ones.

Branches I walked step by step with no defect found:
- **`cappedFindings`:**
  - Every kept finding reserves the separator and omission note for the findings after it, so the joined text stays within `haltFindingsLimit`.
  - A dropped finding at index `i` uses `omittedFindingsNote(len-i)`, which is exactly the note the previous finding reserved.
  - A cut first finding is `limit - reserved - 3` bytes plus the ellipsis, and the cut index is below `len(finding)` whenever it doesn't fit.
- **`groupRuledCells`:**
  - The in-place stable sort sits after the single-cell and added/removed-case `continue`.
  - `low` and `high` come from the sorted ends.
  - The reversed row fails without the sort: the first old value would be `"149"`, not `"0"`.
- **`replacedDecisionRecord`:**
  - It skips trailers that name another path or are unparseable.
  - The first match not after `sinceHead` returns `found` false, which is right because commits come newest first.
  - An empty `sinceHead` skips the check.
  - An `IsAncestor` error, such as a missing object, is now returned as an error. Previously there was none.
  - A rebased or amended snapshot head reads as not after, so it only costs an exemption.
- **Signature callers:**
  - The `Deps` field, `appendedDecisionRecord`, the envkit stub and the loomcli wiring are all updated.
  - `go build` and `go vet` passed for `battencli`, `envkit`, `loomcli` and `driftengine`.
  - `battencli`'s seam list refers to the seam by name only.
- **New `verify_test.go` subtests:**
  - The resume subtest's capture-fault pass leaves the mark open, and `GuardHalt` is cleared by the empty-changes branch.
  - The before-session test fails if the `modified.Before(opened)` branch is removed.
  - The never-returned test fails if the default branch writes nothing, since `GuardHalt` would be nil.
- **`ChangePath_` rows:** each expected path matches how `checkoutChanges` and `scratchChanges` build it.

Tool calls made: none; my context already held every changed file and the test results.
