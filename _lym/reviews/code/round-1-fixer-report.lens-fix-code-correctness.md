```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness lens over the round-1 fix, covering `PlanEditError`, `recoveredSuspectReasons` and the `export_test.go` alias.

**Finding 1, MEDIUM: the F2 fix does not make the test fail when the way forward is wrong.**
- Changed text: `internal/websterengine/runplan.go` `recoveredSuspectReasons`, and `export_test.go` `var checkRecoveredSuspects = recoveredSuspectReasons`.
- The wiring is now covered. The test exercises the same function that `PersistRecoveryTerminal` calls, so a wrong argument order or the wrong function would show up.
- The review's premise was that "a wrong or empty argument at `recoverbatch.go:688` would pass every test". That still holds, because the only test that calls the alias never inspects the way-forward text.
- `suspect_test.go:187-193` asserts only `len(reasons) == 1 && strings.Contains(reasons[0], "at tracked.txt")`.
- If `recoveredSuspectReasons` passed `planPathClause("")` or the wrong verb, that test would still pass.
- Suggested fix: add an assertion that the reason contains `lym webster recover-batch 01`. If the "at tracked.txt" reason does not carry the way forward, use a fixture whose reason does.
- I did not check whether `runplan.CheckRecoveredSuspects` appends the way forward to this particular reason. If it does not, the assertion needs a different fixture.

**Checked, no defect:**
- **F1 text:** the runplan wrap is now `...recorded fingerprint %s; %s`, and the websterengine closure supplies `"the plan changed since state.json was created; " + fingerprintMismatchWayForward(...)`. The result is byte-identical to the old webster text and to `runlevel.go:575`.
- **F1 callers:** the only caller of `runplan.PlanEditError` is `websterengine/runplan.go`. A grep found no other occurrence of "since the run state was created" in `internal`. The wording of `batchCardEditError` is unchanged.
- **F2 signature and imports:** `recoveredSuspectReasons` has the old wrapper's signature and body. The call site `recoveredSuspectReasons(deps.Geom, st, bs, number, head)` type-checks, and `fmt` and `runplan` are still used in `recoverbatch.go`. `export_test.go` no longer needs `fmt`. The parent's `go vet` and `lym gate test` runs on both packages were clean.

Tool calls made:
1. `sed -n 175,215p internal/websterengine/suspect_test.go`: I needed to see what the suspect test asserts, and my context did not hold that file.
