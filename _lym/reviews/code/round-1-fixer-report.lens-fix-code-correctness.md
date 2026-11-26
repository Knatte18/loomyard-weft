```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness check of the F1–F12 change set.

**LOW** — `internal/gatefixer/rowgate.go`, the doc comment of `waitOnly`: it says "a failing one always does" mark the live record terminal, which the F2 change made false.
Evidence: `waitThenRun` now takes the `outcome.deadReason == DeadReasonInterrupted && !replace` branch for a non-carrying entry that fails, calling `markKnown` instead of `markTerminal`. The new row `AnotherEntryFailingOverAnInterruptedFixerLeavesItLive` asserts the record stays live. The fix updated the sibling comments on `WrapRowGate` and `waitThenRun` but left this sentence.

No other findings. I walked these paths:
- **F2:** a left-live interrupted record is re-awaited safely. `stopRecorded` leaves a terminal record as it is, and the F7 change keeps its run directory. The carrying entry then replaces it.
- **F6:** `replaceErr` reaches the note.
- **F1:** the new test fails without the location fields.
- **F3:** no test outside `shedadapters` drives a judge `Call` over a run error that I found; the `shuttleparity` use of `NewBouncer` only builds specs.
- **F7:** the Codex launch records a non-empty session id, so its interrupted runs are kept too.
- **F2 and F8 tests:** they fail with the defect in place.

Tool calls:
1. Grep for `NewBouncer` in tests outside `shedadapters`, for Codex `SessionID` and for sweep tests: my context held only the changed packages.
2. Grep of `internal/shuttleparity/producers_test.go`: my context lacked how it uses the Bouncer.
