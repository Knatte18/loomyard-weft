```yaml
lens: fix-code-correctness
outcome: reported
```

Task: run the fix-code-correctness lens over the F1–F5 change set.

No BLOCKING or MEDIUM findings.

1. **LOW**, `internal/shuttlecli/cli_test.go`, `TestRunCmd_APIErrorEnvelope`, row `DoneCarriesNone`.
   - The row's `events: "START\nSTOP:ok\n"` cannot be what ends the run as done. `apiErrorEngine.ParseSessionSignals` reads only `REJECT:` lines, and `ParseEventsFn` returns `nil, nil`, so no turn end is ever read.
   - The run most likely reaches done through the pre-written output file and the fake clock running to the deadline (`classifyDeadlineExpiry` returns done when the files exist). That path still proves `apiError` is absent on a done outcome, but the events text suggests a path that is not taken.
   - Fix: drop the STOP line from the events text, or comment that done comes from the output file at the deadline.

2. **LOW**, `internal/shuttleengine/apierrorretry.go`, the retries-spent `endOnAPIError` call.
   - The retries-spent api-error end still leaves its strand live, which is the same premise as F1.
   - The review scoped F1 to the rejected-model branch, and the tackler and arbiter callers stop the strand themselves through `stopRunAndHandBack` and `stopRunAndHandOn`, so this is a pre-existing asymmetry and not a defect of this fix.

What I checked, running each by hand:
- **Rejected branch, no output files:** `classifyDeadlineExpiry(OutcomeAPIError)` returns `OutcomeAPIError`. `endOnAPIError` finalizes without removing the strand, so `removeEndedOwnWindowStrand` runs next. With an own-window, present status it reads status, captures the pane, writes `died-capture.txt` and calls `RemoveStrand` once. A failed status, capture or removal only logs a Warn and still returns the result, with the error propagated unchanged.
- **Rejected branch, output files present:** the outcome is `OutcomeDone`, so `finalize` removes the strand itself and the helper is skipped. The new done row expects exactly one removal, which holds.
- **Caller `finalizeAndRemoveDiedWindow`:** it calls the renamed helper, and `removeDiedOwnWindowStrand` appears nowhere else. The fixer's compile and tier0 gate was green. Tests still read `diedCaptureFileName`, which is unchanged.
- **Tests fail without the fix:**
  - The two rejected-model `OutcomeAPIError` rows and the covered row would see 0 `RemoveStrand` calls against the expected 1, so they fail.
  - The done row passes either way. It pins an exception and is not a defect regression, which is correct.
  - The covered row holds the end until the pause lifts (the existing `lifted` assertion) and then removes.
- **F3 callers:** both hand-back messages now read `result.APIError` alone. The `sheddriver` import in `shedarbiter/call.go` is still used by `CauseAPIError` and others. The gate passed, so no test asserted the old text.

Tool calls: none made, because the context already held every changed file.
