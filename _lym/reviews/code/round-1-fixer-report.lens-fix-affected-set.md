```yaml
lens: fix-affected-set
outcome: reported
```

No findings in the fix-affected-set lens. The searches turned up four stale statements, all LOW and none a defect in the fix's own code.

**F2 set (statements that api-error needs `APIErrorRetries`):**
- LOW, `internal/shuttleengine/engine.go:34`: the `OutcomeAPIError` doc reads "past the re-sends Spec.APIErrorRetries allows". A `ModelRejected` end falls outside that. Evidence: the same grep shows `doc.go:75`, `wait.go:8` and `wait.go:221` were updated but this twin was not.
- LOW, `internal/shuttleengine/doc.go:121`, in the same package doc as the fixed `doc.go:75`: "A run whose Spec.APIErrorRetries is set re-sends ... the one past the last re-send ends the run". It stays true as written, and `doc.go:123` states the rejected case.

**F3 set (hand-back text claiming re-sends were spent):**
- LOW, `internal/shedarbiter/doc.go:15`: "a run whose re-sends did not clear it ends as a hand-on with cause `api-error`". A rejected-model end spends no re-send. Its twin `internal/sheddriver/doc.go:91` ("the error past them is a hand-back") has the same gap.
- The two code strings the review named (`tackler.go`, `call.go`) are fixed. No other `%d re-sends did not clear` format string remains.

**F1 rename:**
- Covered. A grep for `removeDiedOwnWindowStrand` and `died run: ` finds nothing left in `.go` or `.md` files. The remaining `diedCaptureFileName` uses are in `wait_test.go`, and that constant was kept.
- The `docs/overview.md` line for shuttle (line 426) already states the at-once rejected-model end. It does not mention the strand removal, which is not a contradiction.

Tool calls:
1. Grep for `APIErrorRetries` near "only/api-error" across `.go` and `.md` files: to rerun F2's set; my context held only the files the review named.
2. Grep for the re-send-did-not-clear phrasings, the old helper name and the `died run: ` log prefix: to rerun the F3 and F1 sets.
