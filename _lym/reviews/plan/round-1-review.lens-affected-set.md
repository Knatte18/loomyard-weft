```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens. For each change a card makes, I listed what reads or asserts it and which cards cover that set.

**Per change: the set found, and the cards that cover it**

1. **New `MidMergeKind` value `MidMergeRecordsUnsettled` (card 10).**
   - Readers:
     - `landingshed/finalize.go:529` (`!= MidMergeNone`): unaffected.
     - `finalize.go:629` `dirtyParentReason`: card 10.
     - `premerge.go:48-50` `preMerge.clear`: card 10.
     - `loomcli/start.go:236` `refuseOverUnfinishedMerge`: card 10.
     - `loomcli/start.go:180` `ownMergeInLeftover`: no card.
   - Partly covered; finding F1.
2. **`statuscommit.Deps.MergeActive` replaced by `MergeOperation`, plus `Stalled` and `Protected` (card 11).**
   - `statuscommit.Deps` is built only in `loomcli/wiring.go:343` and `battencli/commitstatus.go:214`, both covered by card 11.
   - The `commitStatusDeps.MergeActive` fields stay, so these tests keep compiling: `wiring_commitstatus_test.go`, `drivewiring_test.go`, `commitrecords_test.go`, and battencli's `commitstatus_test.go`, `run_test.go`, `driverdeps_test.go`, `cancel_test.go`, `leftoverpairs_test.go`.
   - `wiring_commitstatus_integration_test.go:338` swaps `deps.MergeActive`; card 11 rewrites that row.
   - Covered.
3. **The status seam waits where it skipped (card 11): doc text describing the skip.**
   - Covered: `statuscommit` package doc and `New` doc (card 11), loomcli `commitStatusDeps` doc (card 11), `docs/overview.md:339` (card 15).
   - Missed: loomcli `newCommitStatusSeam` doc ("skip-while-mid-merge"), battencli `newCommitStatusSeam` doc ("It skips while a merge is in progress"), and loomcli `statusCommitPathspec` doc ("a transition whose commit was skipped mid-merge is caught up by the next one").
   - Partly covered; finding F2.
4. **`MergeStateActive` meaning in one repository (card 10).**
   - Callers: `loomcli/wiring.go:219` (replaced by `MergeOperation`, card 11); `battencli/commitstatus.go:137` (`commitRunRecords`), `:162`, `leftoverpairs.go:71` and `cancelcmd.go:475`; `loomcli/commitrecords.go:43`; `tendercli/runintents.go:240`.
   - Card 10 keeps the signature and meaning for its outside callers, and card 10's records route handles their commits.
   - Covered.
5. **New `PushFailureProtected` class (card 4).**
   - Readers: `IsPushOwed` switch (`pushanchored.go:53`, not owed, unchanged); `FailedPushesOwedBy` (`failedpush.go:284`, excluded); `recordPushFailure` notice gate (`failedpush.go:170`, notifies as non-deferred); `FailedPushWayForward` (card 4); `Finalize.pushParent` (card 4); seam `Protected` (card 11).
   - Covered.
6. **New `PushFailureOperatorUnpushed` class (card 14).**
   - Readers: `IsPushOwed` (`pushanchored.go:53`), `FailedPushesOwedBy` (`failedpush.go:284`), `FailedPushWayForward`, the `recordPushFailure` notice (sent as non-deferred), the runfsm act rule and `hubRunReader`.
   - Covered by card 14.
7. **A deferred record carrying `Commits` sends a notice (card 13).**
   - Gate at `failedpush.go:170` in `recordPushFailure`: card 13.
   - Deferred reader at `pushanchored.go:344`: unchanged meaning.
   - Covered.
8. **Merge trailer on every fabric merge commit (cards 2, 9) and anchor-change trailer (card 8).**
   - Subject assertions use `strings.Contains` on `%s` (`anchorchange_integration_test.go:230`, `:578`; `add_anchor_integration_test.go:89`), so a body trailer breaks none of them.
   - Covered.
9. **`protectedBranchRefusal` and `protectedBranchMarkers` moved (card 4).**
   - The only references are in `landingshed/finalize.go` (lines 848, 898, 901).
   - Covered.

**Findings**

- **F1 — MEDIUM, consistency, `_lym/plan/10-records-commit-during-merge.md`.**
  - Card 10 gives `MidMerge` a new kind but leaves `loomcli`'s `ownMergeInLeftover` unlisted.
  - That function matches only `MidMergeParked`, so a Parent-Align or Finalize row's own merge-in that stopped before step 3 is no longer recognised as the row's own leftover. `refuseOverUnfinishedMerge` then refuses the start or resume instead of letting the row abort and redo it, which D5 says stays as it is.
  - Evidence: `internal/loomcli/start.go:180`, `:236`.
  - Missed member: `internal/loomcli#ownMergeInLeftover`.
- **F2 — LOW, consistency, `_lym/plan/11-transition-seam-waits.md`.**
  - Card 11 turns the seam's skip into a wait, but its Intent names only loomcli's `commitStatusDeps` doc and statuscommit's own docs.
  - Three doc comments still describe the skip:
    - loomcli `newCommitStatusSeam` (`internal/loomcli/wiring.go`, the doc above line 342, "skip-while-mid-merge");
    - battencli `newCommitStatusSeam` (`internal/battencli/commitstatus.go`, the doc above line 211, "It skips while a merge is in progress");
    - loomcli `statusCommitPathspec` (`internal/loomcli/wiring.go`, the doc near line 106, "a transition whose commit was skipped mid-merge is caught up by the next one").
  - The first two are edited members whose doc change goes unstated; `statusCommitPathspec` is listed by no card.

**Tool calls**

1. Grep for `MidMergeKind`, `statuscommit.Deps`, `MergeActive` and `MergeStateActive` users. The manifest lacked the full reader and test sets.
2. Grep for push-class switches and docs mentioning the mid-merge skip. The manifest lacked the class readers and the docs set.
3. Read `failedpush.go:155-180` (`recordPushFailure` notice gate) and grepped specs and doc.go files for skip text and the protected-marker references. The manifest lacked the notice gate.
4. Grep doc.go files for skip text and tests for merge-message assertions. The manifest lacked trailer-sensitive tests.
5. Grep merge-message construction and tests pinning merge and anchor commit subjects. Needed to confirm trailer-sensitive assertions.
