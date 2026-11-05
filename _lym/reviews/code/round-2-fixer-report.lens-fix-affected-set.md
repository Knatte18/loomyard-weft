```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1:** The refused-Sketch notice at `internal/loomcli/wiring.go:258` was the only bare `lym board set-deps` in a user-facing notice that the parent has to act on. The sibling mentions in `internal/boardengine/store.go` (374, 378, 389) and `internal/boardcli/cli.go:1245` are board refusal texts that the review did not name. Those are outside the finding and the fix did not change them.
- **F2:** No other doc or comment states the removal-before-guard order. The single other hit is an unrelated test comment in `internal/shuttleengine/waiting_test.go:325`.
- **F3:** `startClearsMotifRun` has exactly two callers: `internal/battencli/dobbyreads.go:329`, which the new row now covers, and `clearsMotifRunOfRaddleEntry` at `internal/battencli/startguard.go:170`, whose error path `TestClearsMotifRunOfRaddleEntry` already pins.

Calls made:
1. Grep for `lym board set-deps` in non-test Go files, to find every site worded like F1's.
2. Grep for the "before the start guard" and "removes the run before" wordings across Go and markdown, to find every site worded like F2's.
3. Grep for `startClearsMotifRun(` callers, to check F3's call-site set.
