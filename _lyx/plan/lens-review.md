# Lens review

Every lens returned a report.

- criteria-coverage, MEDIUM, D3 ancestor rule unreached: fixed, card 2 gains a row creating `internal/new/sub/x.go#Foo` with a Verify on `./internal/new`, absent on disk.
- criteria-coverage, MEDIUM, D3 `_test` hint on a created directory unreached: fixed, card 2 gains a hint row over `internal/y/y.go#Foo` created by an earlier card.
- criteria-coverage, MEDIUM, D4 prebuild ordering unpinned: fixed, card 3's integration refusal subtest runs with `--tags integration` and asserts no stand-in record, so neither the prebuild nor go test ran (`internal/gatecli/run.go:96-107` builds only when tags are set).
- criteria-coverage, MEDIUM, D5 departure leaves `pattern/PATTERN-failure-stop-visible.md` stale: fixed, card 4 edits that background; its resume bullet (`pattern/PATTERN-failure-stop-visible.md:7`) is false for a run left reading running.
- criteria-coverage, LOW, D5 `logger.Warn` unasserted: rejected, a log line is observability, not a public-surface behavior the halt note depends on.
- criteria-coverage, LOW, Verify on the Move card itself unreached: fixed, card 2 gains that row.
- criteria-coverage, LOW, earlier-card delete bound unpinned: rejected, the directory exists on disk in any such row, so it passes on the existence branch and pins nothing distinct.
- row-reachability, MEDIUM, card 2's hint assertion cannot fail since the echoed `./internal/x_test` contains `./internal/x`: fixed, hint rows assert `external test package` and every other reporting row asserts its absence.
- row-reachability, LOW, card 3 scenario hint assertion is a substring of the echo: fixed, merged with test-economy's finding; the scenario subtest drops the hint assertion and the unit row, which checks `external test package`, pins the wording.
- row-reachability, LOW, `--` stop in `gateTestDirAndPackages` unreached: fixed, card 2 gains `lyx gate test ./internal/x -- ./nope` reporting nothing.
- row-reachability, LOW, subcommand filter untested: fixed, card 2 gains `go list ./nope` and `go run ./nope` reporting nothing.
- row-reachability, LOW, earlier-card delete row missing: rejected, a duplicate of the criteria-coverage finding above, for the same reason.
- row-reachability, LOW, spent re-step then a status read error untested: rejected as a row; the combination follows from the single `restepped` flag, and card 4's doc text now states the shared budget (merged with forced-edits' finding).
- row-reachability, LOW, the pre-created `nested` meets the later `os.MkdirAll` (`internal/gatecli/test_integration_test.go:379`): fixed, card 3 states it is the same directory.
- forced-edits, LOW, `validate.go:3` file comment sits outside `validate`'s span: rejected, listing `internal/planparser/validate.go` beside the member is refused by `redundant-file-target`, and the Intent names the comment.
- forced-edits, LOW, `internal/planparser/doc.go:12-19` chokepoint section stays stale: fixed, card 1 names that section.
- forced-edits, LOW, sandbox suite lacks the new gate refusal: rejected, `PATTERN-sandbox-coverage` asks that the module be exercised, which `tools/sandbox/SANDBOX-CORE-SUITE.md`'s gate scenario already does, not one watch per refusal.
- forced-edits, LOW, the re-read spends the single re-step and the doc says otherwise: fixed, card 4's doc text states the shared budget.
- test-economy, LOW, single-glyph `TestCardTargetDirs` row duplicates the shared-entry row: fixed, dropped.
- test-economy, LOW, `TestPeakContext` row depends on `CardTargetDirs`: rejected, the acceptance criterion names the batcher's estimate, and the row pins the merge at that reader.
- test-economy, LOW, `go test ./nested/pkg` no-report row repeats the existing-directory row: fixed, dropped.
- test-economy, LOW, scenario refusal subtest repeats the unit row's hint: fixed, the hint assertion dropped and the before-slot assertion kept.
