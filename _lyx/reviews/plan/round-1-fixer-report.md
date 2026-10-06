# Round 1 fixer report

## Changed

All fifteen findings are fixed in the plan under `_lyx/plan/`; the target files were touched only after `round-1-review.md` was saved.

- F1: card 1 no longer edits `contracts/stencils/loom/loom-rubric-plan-review.md`; the Edit list, the Intent and the ImpactSummary say so.
- F2: card 1's tally sweep now covers `internal/planparser/doc.go`, `internal/planparser/normalize.go` and the spec's shape-classifier sentence; both planparser files joined its Edit list.
- F3: card 1 names `handlePlan` in `internal/websterengine/rebaseline_test.go` as a known fixture to reorder and adds the file to its Edit list; the grep for others stays.
- F4: card 1 creates one table-driven `TestValidateCmd_PlanGateChecks` in `cli_test.go`; cards 2 and 3 add rows to it instead of functions.
- F5: card 3 now names `delete-target-gone` in the spec paragraph card 2 adds, and lists the spec in its Edit list.
- F6: card 4 deletes `TestBeginBatch_UnknownRoleErrors` with the other three tests.
- F7: card 6 drops its `recoverbatch_test.go` test and target, and cites `TestRecoverBatch_FirstCallSpawnsArchivesStaleReportAndStopsLiveStrand` as the coverage.
- F8: card 8's "what ends it" now follows the shuttle outcome: done, Master's stuck and the gate demotion remove the strand, and only asking, died and timeout leave it for the next run.
- F9: card 8 builds the `RunResult.Warnings` entries after the demotion from the note's outcome description, appends them on done and stuck, and adds a Master-stuck test row.
- F10: card 9 builds `CommitsNotIn` on `git rev-list` through `runChecked`, validates both SHAs, and extends `doc.go`'s CLI-bound method list; its ImpactSummary matches.
- F11: new card 10 (`10-fabric-remote-only-commits.md`) owns `Fabric.RemoteOnlyCommits` with a direct integration test; the reset card became 11 and Publish became 12, with the Card Index, the `cli-drive` Shared Decision (text and Applies to) and cross-references updated.
- F12: card 11 adds a forced-checkout-failure test at the fabric tier and the webstercli tier.
- F13: card 11 states that a failure after `remote_branch_updated` returns `mutations` and `partial: true` and drops the "refused" prefix, with the tests asserting it.
- F14: card 11 adds `internal/websterengine/doc.go` and names the `Kind` vocabulary list in `fabricengine/doc.go`.
- F15: the `## verify:` tmux line now runs `./internal/loomcli/...` too.

The `approved:` key in `00-overview.md` was not touched.

## Deferred

None.
