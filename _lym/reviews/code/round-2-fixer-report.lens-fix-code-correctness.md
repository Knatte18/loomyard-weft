```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1 and F2 test changes through the fix-code-correctness lens (concrete inputs, callers, and whether each test fails with the defect in place).

**Findings**

1. **MEDIUM** — `internal/testcover/sourcescan_test.go`, the rows `KitBehindCycleEnteredFromOuter` and `KitBehindCycleEnteredFromInner`. Both rows pass with the memo defect in place.
   - The defect is a false answer memoized for `inner` while `outer` is still on the path, which the `lowest &gt;= depth` guard in `ScanSourceReaders` prevents.
   - Single-test row, entered from outer: `outer` → `inner` → `outer` is on the path, so `inner` returns false with lowest 0.
     With the defect, `inner` is memoized false, but `outer` then reaches `kit.Walk` and is found.
     The only test in the package, `TestKitBehindCycleFromOuter`, is marked either way, so the row passes.
   - Single-test row, entered from inner: `inner` → `outer`, which finds `kit.Walk` and is memoized true.
     `inner` is true, so the defect never shows.
   - The defect shows only in a package holding both tests, with the outer test walked first.
     The memoized-false `inner` then leaves `TestKitBehindCycleFromInner` unmarked.
     The old 16-draw loop caught this, with high probability.
     The review's third row, which holds both tests, would have kept that coverage.
     The fixer dropped it, so the regression coverage for the call-cycle memo fix is lost.
   - The "both tests, wants both marked" row is not deterministic either, because the walk order is random.
     A deterministic variant needs an order-independent construction, such as a test whose only route to `inner` comes after a memoized-false visit.

2. **LOW** — `internal/gatecli/redundant_integration_test.go`, the gate string and `redundantTestSecond`. Nothing asserts that the floor binds.
   - The fixture's test bound is `20 × 250 ms` = 5 s and the floor is `10 × 250 ms` = 2.5 s.
   - If `mutantTimeout` ignored the floor, or used only the test bound, the endless `Tally` mutant would end at 5 s.
     The kill counts (`MutantsKilled: 7`) would be the same, so the test would still pass, only slower.
   - The comment claims the endless mutant "ends at the floor and not at the test bound", but no assertion pins that.
   - A healthy fixture test finishing inside the 2.5 s floor is a timing assumption, but it is reasonable.

**Checked and holding**
- `runRedundant` is called with the same `pkg`, `goBinary` and `environ` that `redundantCmd` passes.
  The seam cwd comes from `lymcwd.WithCwd`, `minuteUnit` is `time.Minute`, and `grace` is `defaultPipeWaitDelay`, as in `Command()`.
  The `secondUnit` field is the only difference.
- The imports `context` and `lymcwd` are used.
- No signature changed in production code, so no caller is affected.
- The `go vet` and gate runs were green, and the test dropped from about 41 s to about 11 s.
- The `other.Run` var in `scanSourceHelpers` keeps the `other` import used, and the scan only parses it.

**Tool calls:** none. My context already held every file I needed.
