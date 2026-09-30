# Plan-Review round 1 — fixer report

All eight findings fixed; nothing deferred.
After the edits, `lyx loom validate-plan` (built from this worktree) returns `ok: true` with only the three informational `create-new-unit` findings.
The overview's `approved:` key was not touched.

## Fixed

- **F1 (MEDIUM), `01-head-reconcile-helpers.md`:** the `reconcileReportHead` walk now compares each commit with `reportHead` before checking its parent count, then requires two or more parents only of commits not equal to it.
  The card says why the order matters: a merge-commit report head, such as a no-commit fork's `StartSHA` left by an earlier parent merge.
  Added a test: a merge-commit report head with a second `--no-ff` merge on top is accepted, and the warning names only the second merge.
- **F2 (LOW), `06-merge-in-warns-mid-run.md`:** pinned both signature changes as `warnings []string` parameters.
  `setMergeExit` forwards the slice to the ok and conflict envelopes, and the `warnings` key appears only when the slice is non-empty.
  Added `internal/fabriccli/envelope_test.go` to the Edit list; its existing `errConflictsWithRecord` calls pass nil.
- **F3 (LOW), `03-recover-batch-tolerates-merge.md`:** moved `internal/websterengine#RecoverResult` from Uses to Edit, and the Intent now updates its doc comment to list the moved-HEAD notice.
- **F4 (LOW), `02-…` and `03-…`:** card 2 now edits `doc.go`, adding the merge-only head rule and the bisect limit for record-batch in the commit that changes record-batch.
  Card 3 now extends that paragraph to cover recover-batch, and its title says so.
  The overview's Card Index lines for cards 2 and 3 match.
- **F5 (LOW), `05-run-in-flight-predicate.md`:** added an unreadable-`outcome.yaml` row (a directory at the path) expecting an error.
- **F6 (NIT), `06-…`:** the recipe-blind constraint now forbids both recipe names, `loom` and `batten`, as whole words.
- **F7 (NIT), `06-…`:** the test now seeds under `websterengine.Dir(h.Location.AnchorPath())`, the same anchor path the verb reads.
- **F8 (NIT), `04-parent-merge-wedge-scenario.md`:** named the remaining `RecordDeps` seams: `NeverMatches{}`, a `recordFakeSleeper`, and temp-dir `OutcomePath`/`SummaryPath`, matching `newRecordFixture`.

## Deferred

None.
