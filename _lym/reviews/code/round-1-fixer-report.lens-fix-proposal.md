```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1 to F5).

Findings:

1. **LOW** — `internal/shuttleengine/apierrorretry.go`, the `if watch.rejected` branch and the `endOnAPIError` call after it. The F1 fix leaves the same live-strand defect on the other api-error end.
   - F1's premise is that `finalize` removes the strand only for done, so an api-error end leaves the pane registered.
   - The retries-exhausted end (`watch.errors > run.spec.APIErrorRetries`) also goes through `endOnAPIError` with `OutcomeAPIError`. It gets no `removeEndedOwnWindowStrand` call.
   - A settler run that spends its re-sends still ends api-error with a live strand. The tackler and arbiter hand-back paths stop it themselves, but `lym shuttle run` does not.
   - The row "an API error past the re-sends ends the run naming the error" runs with the default `liveStrandStatus`, which is not own-window, and checks no removal. The inconsistency is therefore untested and unnoticed.
   - The new `doc.go` and `apierrorretry.go` sentences say only that the rejected end removes the strand. A reader would take that as the intended asymmetry.
   - Call the removal inside `endOnAPIError`, or in both branches, and give the retries-exhausted row `ownWindow: true` and `wantRemoved: 1`.

2. **LOW** — `internal/shuttleengine/wait.go`, `removeEndedOwnWindowStrand`. The F1 proposal ("the way a died own-window run does") leaves a strand that is not own-window live after a rejected-model end.
   - A run with a spec anchor, or one on window 0, still ends api-error with its strand registered.
   - The helper's doc comment states the skip, but the new `doc.go` line says "removes the run's own-window strand" and does not name the bound for anchored runs.
   - This is acceptable for died runs, where reed reconciles dead panes. A rejected-model pane stays alive, so reed does not reconcile it.
   - Either document the bound at the `doc.go` and `retryAPIError` sentences, or remove through the stop verb, which has no own-window restriction on a live strand.

F2, F3, F4 and F5 hold up. I found no case where the proposed change is wrong.
- F3: the `sheddriver` import in `internal/shedarbiter/call.go` is still used by `sheddriver.CauseAPIError`, and `SettlerAPIErrorRetries` stays referenced elsewhere in `sheddriver`.
- F4 and F5: the added rows would fail with the defect in place.

Further tool calls: none. My context already held every file the lens needed.
