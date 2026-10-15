# Lens review

Every lens fork returned a report: criteria-coverage, branch-coverage, row-reachability, forced-edits and test-economy.
Duplicates across lenses are merged; each line names every lens that raised the finding.

## Card 1

- test-economy, LOW: the CLI `tree_moved` row repeats the engine row's cause selection — fixed as proposed: `TestBeginBatchCmd_PlanConflict` keeps one cause row, and card 1 names the engine row as the `tree_moved` coverage.
- criteria-coverage, MEDIUM: no test pins that record-batch and recover-batch keep `plan_drifted: true` (criterion 4) — fixed as proposed: new `TestFingerprintRefusals_CarryPlanDrifted` table over both verbs after a card edit.
- criteria-coverage, LOW: no claim pins D4's standalone half — fixed as proposed: the new claim also requires the section to say a standalone run's way forward is the refusal's own.
- criteria-coverage, LOW: every record row has one blocking finding, so a writer keeping only one passes — fixed as proposed: the `plan_defect` row's card carries two blocking findings (create-already-exists and bare-symbol-target), and the record must hold both lines.
- criteria-coverage and row-reachability, LOW: no row shows the begin-batch refusal note carrying `plan_conflict` — fixed as proposed: the CLI row runs `beginBatchCmd` wrapped by `noteRefusals` with a friction dir and asserts the note holds `plan_conflict: plan_defect`.

## Card 2

- test-economy, LOW: `TestBeginBatchCmd_ConflictRecordLifecycle` is a second top-level test for the same verb — fixed as proposed: its rows join `TestBeginBatchCmd_PlanConflict`, and the separate test is dropped.
- criteria-coverage and row-reachability, MEDIUM: quarry-unavailable leaves no record only by argument — fixed as proposed: a `TestBeginBatchCmd_PlanConflict` row points `geom.WorktreeRoot` at a missing directory, the input the engine row uses for `ErrQuarryUnavailable`, and asserts no `plan_conflict` and no record.
- branch-coverage, LOW: the record write on a failed fingerprint persist is untested, and that exit prints no class — fixed differently: the proposed row would need a forced `SaveState` failure. Instead the card now writes the record only after a successful persist. On a failed persist, the refusal already prints the combined message with no class (`internal/webstercli/beginbatch.go:123-126`), so it writes no record and halts as today, which removes the branch.
- branch-coverage, LOW: the `RemoveRoutedConflict` failure on a successful begin and `WriteConflictRecord`'s directory-creation branch are unreached — fixed as proposed for the first: a successful-begin row with a non-empty directory at `RoutedConflictPath` exits 0. Fixed differently for the second: the proposed row cannot exist, because every caller holds a lock file in the scratch dir (`internal/webstercli/beginbatch.go:71`, `internal/websterengine/runlevel.go:356-360`). The card drops the creation step instead.
- row-reachability, MEDIUM: the Run-entry removal-failure row cannot tell Warn-and-go-on from a refusal — fixed as proposed: the row also asserts the scripted starter was reached (`stop before spawn`).
- forced-edits, LOW: the `Geometry.ScratchDir` field comment (`internal/websterengine/geometry.go:54`) and `internal/websterengine#ScratchDir`'s comment go stale — fixed as proposed: both listed on card 2, which updates them to name the conflict files.
- forced-edits, LOW: `runVerbs` builds a `websterCLI` with no geometry, so the first-step removal resolves against the working directory — fixed as proposed: card 2 edits `internal/webstercli#runVerbs` to set `geom.ScratchDir` to a temp dir.

## Card 3

- forced-edits, LOW: `docs/overview.md:516` lists only the Merge-In reroutes — fixed as proposed: card 3 adds the Webster reroute to that paragraph.

## Card 4

- test-economy, LOW: `TestWebsterProducer_StuckPlanConflict` is a new test for the stuck arm `TestWebsterProducer_OutcomeStuck` covers, and its no-record row repeats it — fixed as proposed: `TestWebsterProducer_OutcomeStuck` becomes the table, its existing case is the no-record row, and the new test is dropped.
- branch-coverage, MEDIUM: an unreadable or unwritable routed file is unreached — fixed as proposed: a row with a new record and a non-empty directory at the routed path wants plain `Stuck`, no `Reroute`, and the directory left in place.
- branch-coverage and criteria-coverage, LOW/MEDIUM: no row seeds a record in the cancellation tests, so a claim before `cancelErr` passes — fixed as proposed: the `Stuck` row of `TestWebsterProducer_CancelledDuringRun_StuckPausedAndErrorBecomeContextError` seeds a new record and asserts the routed file stays absent.
- branch-coverage, LOW: a record with no finding line equals the empty routed set and claims a false recurrence — fixed as proposed: `ClaimConflictRoute` treats such a record as not found, and a row pins plain `Stuck` with the stuck reason.
- criteria-coverage, LOW: the reroute reason "routed to plan review" is false in a capped halt, which quotes it — fixed as proposed: the reason now reads "asking a reroute to plan review", and the new-record row asserts it claims no review.
- criteria-coverage and forced-edits, LOW: the existing stuck-arm tests leave `Geom.ScratchDir` empty, so the claim reads the working directory — fixed as proposed: every adapter test reaching the stuck arm gets a temp scratch dir. That includes `internal/loomshed`'s `TestWebsterProducer_Call` and `TestWebsterProducer_PassesStuckReasonThrough`, whose `RunDeps{}` reaches the same arm (`internal/loomshed/webster_test.go:137-138`).

## Card 5

- test-economy, LOW: `TestEditOne`'s empty row repeats the `ErrAborted` arm the editor-failure row reaches — fixed as proposed: row dropped, and `TestEditOne` leaves the card.
- test-economy, LOW: the local-layer empty row repeats the pre-existing restore inside `EditPath` — fixed as proposed: row dropped, and the card names the pre-existing-file row as its coverage.
- branch-coverage, MEDIUM: no row restores entry bytes after a parse-error pass — fixed as proposed: a `TestEdit` row writes invalid then empty bytes over a pre-existing file and wants `original` back.

## Card 6

- test-economy, LOW: the four per-layer no-terminal rows reach one branch — fixed as proposed: one row with the board and local files seeded and the worktree file absent covers it.
- branch-coverage and row-reachability, MEDIUM: no row proves `--print` and `--set` need no terminal — fixed as proposed: two `TestDispatchLayers` rows with no terminal, `--print` and `--set`, each exit 0.
- row-reachability, LOW: the no-terminal default-layer row's nil hub-write guard cannot pin the guard order — fixed as proposed: that row's guard fails the test when called.
- branch-coverage, criteria-coverage and row-reachability, LOW/MEDIUM: `stdinIsTerminal` and the production wiring are untested — fixed differently: the proposed alternative of calling `stdinIsTerminal` directly needs a new top-level test in a package at its budget, and still leaves the `runConfig` wiring unreached. `runConfig` instead takes `cmd.InOrStdin()`, and `TestConfigOverRealHub` drives `Command()` with `SetIn` set to a regular file (`loom`) and to a string reader (`menu`). Both refuse, which reaches both non-terminal branches and both wirings.

## Disputed

- criteria-coverage, MEDIUM: criterion 13 (the refusal-spec row) has no test — disputed, every fix worse: the only fix is a test that scans `contracts/specs/refusal-spec.md` for row text. `pattern/PATTERN-refusal-way-forward.md:8` says enforcement of the spec rows "is review discipline plus the tests that reach each row; there is no scan". The refusal itself is reached by card 1's engine and CLI rows.
