```yaml
lens: fix-consistency
outcome: reported
```

No findings.

I checked each changed text against its siblings:
- **F6 docs:** `OutcomeAPIError` in `internal/shuttleengine/engine.go` and the api-error paragraph in `docs/overview.md` now both name the capped skill-load hold. The only other docs that list api-error paths are `abandonLoad`'s comment, `internal/shuttleengine/doc.go:516` and the claude engine's `StopFailure` doc; they already cover the cap or are unrelated to it. The research note is a dated record, so I left it out of scope.
- **F2 wording:** the new `IdleGap` and `internal/shedverbs/doc.go` lines match each other and `idleGapsOf`'s existing LastAt rule. The sentence above them in `doc.go` still reads "every stretch of the history in which no step ran". The new line scopes it, so the two do not contradict.
- **F1 helper:** the done, stuck, reroute and capped assertions all go through `assertCallTimed`, and the done test's inline copy was removed.
- **F3/F4:** the table's `cappedTurns` now comes from `cappedLoadTurnEnds()`, so the shared row and the new test script the same turns. The back-off sum still covers loadHoldMaxAPIErrors-1 waits, as before.

Further tool calls (1):
1. A grep over non-test Go, markdown and yaml files for the api-error wording (`model_not_found`, `rejects the model`, `rejected model`, `re-send limit`), to find twins of the F6 doc edit that were left stale. My context held only the two files the review named.
