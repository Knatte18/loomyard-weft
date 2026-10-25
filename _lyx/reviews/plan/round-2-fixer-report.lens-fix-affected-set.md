```yaml
lens: fix-affected-set
outcome: reported
```

fix-affected-set: I reran each set finding's search to check the fix covers every member. Unchecked: F1(c)–(e), the import lists. I ran out of tool calls; I did confirm that `gopkg.in/yaml.v3`, `proc.Detach` and the arbiter prompt's three imports exist.

Findings:

1. LOW — card 7. The new Tests sentence and Edit list cover the integration tests that assert a nil status, but one test that reads the old clear value is missing. The F1(a) search was for tests that read a cleared claim.
   - `internal/pairteardown/teardown_test.go:873`, `TestSettleBoardClaim_RunningBattenRecordWayForward`, says the claim was kept by rejecting `write=clear` and `write=done`.
   - The fake writer (`teardown_test.go:174-179`) logs a non-nil write as `write=`+status.
   - After card 7, a wrongly cleared claim logs `write=draft`, so the check passes without testing anything.
   - Card 7 should list `internal/pairteardown#TestSettleBoardClaim_RunningBattenRecordWayForward` and add `write=draft` to the writes it rejects.

2. LOW — card 13. Only the callers of `testInnerRunRefusedResume` are listed, not the helper.
   - The Tests text has every test that ends a Call through `fakeClock.Sleep` drive successive Calls.
   - The helper `testInnerRunRefusedResume` (`innerrun_awaiting_test.go:350`) sets `clock.pauseAtSleep = checks` and `clock.onSleep` (lines 378-380) to end its wait inside one Call.
   - Its caller, `TestInnerRun_AwaitingRefusedResumeRetriesWithoutMarkerThenResumes`, is listed. The helper's body is where the rewrite happens, and card 13 doesn't list it.
   - Card 13 should add `internal/battenshed#testInnerRunRefusedResume`.

Covered with no gap:
- **F1(a):** the integration tests that assert a nil status are exactly the three `TestCancelIntegration_*` tests and the `TestRun_TeardownScenario` row. `TestTeardown_CallSequence` needs no fixture edit, because the fake already logs `write=draft`.
- **F1(b)/F2:** every other battenshed test that uses a sleep hook is either listed on cards 13–16 or is Land, create or row-notice code with its own `Sleep`. That code is `fakePrimeLock`, `waitingPrimeLock`, `TestLand_LandLockWait` and `TestRowNotifierSend_DeliveryBound`.

Tool calls:
1. grep for tests asserting a nil board status: needed to rerun F1(a)'s search over the whole tree.
2. grep for the claim-clear callers and the nil-status and `write=clear` assertions: needed to find unit-level readers of the clear value.
3. sed/awk of `teardown_test.go` around lines 175 and 873: needed the enclosing tests and the assertion.
4. sed of the fake writer in `teardown_test.go` and a count of `write=clear`: needed to see how a draft write gets logged.
5. awk over the battenshed tests that use sleep hooks, checked against cards 13–16: needed to rerun F1(b)/F2's search.
6. grep of `testInnerRunRefusedResume`: needed to check whether the one unlisted InnerRun helper uses the sleep hooks.
