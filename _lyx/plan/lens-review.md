# Lens review

Every fork returned a report.
Duplicates merged: test-economy 1 and row-reachability 2 (ancestor row); criteria-coverage 8 and forced-edits 5 (parity row).

- criteria-coverage 1, MEDIUM, fixed as proposed: card 1 adds a `TestRenderBatchGate` row with `internal/a#Run` and `internal/a_test#TestRun` across two cards, expecting `./internal/a` once, and lists `internal/websterengine#TestRenderBatchGate`.
- criteria-coverage 2, MEDIUM, fixed differently: the proposal put the row in `TestGateTest_Scenario`'s `refusals` loop, but that loop runs the fixed arguments `test ./pkg` (`internal/gatecli/test_integration_test.go:656`), so a missing-package row does not fit its shape; card 3 adds its own subtest instead, which runs `test --tags integration ./nope` from the prime and asserts exit 1, no stand-in record and `requireFree`.
- criteria-coverage 3, LOW, fixed as proposed: card 3 adds a `./nope ./gone` row naming both.
- criteria-coverage 4, LOW, fixed as proposed: card 2 adds a firing `go build ./nope` row.
- criteria-coverage 5, LOW, fixed as proposed: card 2 adds a firing `go test ./internal/new_test` row over a Create card on `internal/new/x.go#Foo`, whose detail names `./internal/new`.
- criteria-coverage 6, LOW, fixed as proposed: card 2 adds a silent overview `go test ./nope` row in a plan holding a Move card.
- criteria-coverage 7, LOW, fixed as proposed: card 4's rows capture the logger through `logger.SetOutput` (as `internal/websterengine/runlevel_test.go:390` does; `internal/shedverbs/loop_test.go:3` already keeps the file unparallel for the global logger) and assert the WARN line naming the status path.
- criteria-coverage 8 and forced-edits 5, LOW, fixed as proposed: card 2 adds a `VerifyPackageMissing` row to `TestGateParity_PlanGate` built with `verifyPlanFixture(..., "", "go test ./nope")`, and lists `internal/loomcli#TestGateParity_PlanGate`.
- test-economy 1 and row-reachability 2, MEDIUM, fixed as proposed: card 2 replaces the `go test ./internal` row, which only met an on-disk directory, with a Create card on `pkgnew/sub/x.go#Foo` and Verify `go test ./pkgnew`, `pkgnew` absent on disk.
- test-economy 2, LOW, fixed as proposed: card 2 adds a firing Delete-only row, a Delete card on `internal/gone/g.go#` with Verify `go test ./internal/gone`.
- test-economy 3, LOW, fixed as proposed: card 1's Intent says the gate and estimate rows pin the acceptance criterion's named consumers, which the `TestCardTargetDirs` merge row cannot.
- test-economy 4, LOW, fixed as proposed: card 4 adds `TestLoop_InterruptedChildFillsTheErrorForm` to the Enforcement line of `pattern/PATTERN-failure-stop-visible.md`.
- row-reachability 1, BLOCKING, fixed as proposed: card 2's rows build their plain Verify card with `cardOfType(1, "a", planparser.CardTypeEdit, []string{"internal/x/x.go"})`, never `validCard`, whose `pkg/card<N>.go` target (`internal/planparser/validate_test.go:36`) would make `./pkg` a created directory.
- row-reachability 3, MEDIUM, fixed as proposed: card 2's table gains a `wantDetail` column checked against `Detail`, set on both `_test` hint rows.
- row-reachability 4, LOW, fixed as proposed: card 3's `-C sub ./pkg` row names `sub` among its setup directories.
- row-reachability 5, LOW, fixed as proposed: card 4 says each row sets `Hooks.AfterInterrupt` and `MissingStatusWayForward` on its own fixture, leaving `newLoopFixture` unchanged, and that each `ReadStatus` fake passes its first call through so the first child runs.
- row-reachability 6, LOW, disputed, premise false: the premise is that only the plan-level verify runs card 3's integration-tagged subtests, but Webster's batch gate runs `lyx gate test --tags integration` over the batch's packages (`internal/websterengine/cardgate.go:137`, pinned by `TestRenderBatchGate`'s `--tags integration` want), so the batch holding card 3 runs them before the plan-level verify.
- forced-edits 1, LOW, fixed as proposed: card 1 lists `internal/planparser#CardTargetDirs` and rewords its doc comment for the external-test mapping.
- forced-edits 2, LOW, fixed as proposed: card 2's Intent rewords `verifymodule.go`'s head comment to name the third check.
- forced-edits 3, LOW, fixed as proposed: card 2's Intent says the `validate.go` head-comment edit is a file comment that rides on the card, since listing the file beside `#validate` is refused as `redundant-file-target`.
- forced-edits 4, LOW, fixed as proposed: card 4 lists `internal/shedverbs#Hooks` and rewords the `AfterInterrupt` field doc for the loop-log trace file.
