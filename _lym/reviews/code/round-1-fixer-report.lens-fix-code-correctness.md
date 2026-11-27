```yaml
lens: fix-code-correctness
outcome: reported
```

**Task:** fix-code-correctness check of the code the fix changed (`latercaller.go` and its new test rows).

**LOW**: changed doc and spec text, plus `isTestSupportUnit` as applied by the fix (`internal/planparser/latercaller.go`, `isTestSupportUnit` and `cardCanHoldProductionCaller`).
The fix exempts test-support units only for the callee's declaration. On the creating card, a new `func` handle in a test-support directory still cannot be the caller, although the guard roots it.
- **Case:** card 1 creates `plan:internal/watch#Level` and `plan:internal/testkit/watchkit#Probe`, a plain-head func that calls `Level`. Card 2 names `Level` and edits `internal/dobby#Tick`.
- **Trace:** `cardCanHoldProductionCaller(..., false)` keeps `Probe` in `declared`, and `refCanHoldProductionCaller` returns false for its `func` keyword. Card 2 can hold a caller, so card 1 is refused, even though its gate passes: `testonlycode_linux_test.go` roots every test-support function.
- **Stale reason:** the new doc sentence "since a new function is reached only through a caller of its own" does not hold for a test-support function, which this same change describes as counted reached.
- **Scope:** the refusal itself predates the fix, the case is narrow, and moving the `Create` still clears it.

Nothing else found. I traced these by reading the code and ran no mutants:
- Both callers of the changed `cardCanHoldProductionCaller` signature pass the right flag: `false` for the creating card, `true` for the later card.
- With `ownFunctionsCall`, an own non-test func counts. An own func with a `testing.` head stays excluded, and a test-named or `_test`-unit func is rejected by `isTestHandle`.
- `isTestSupportUnit` matches the exact directory and directories below it, and does not match a sibling prefix such as `internal/testkitx`.
- Each added row would fail with its targeted defect in place: the Extends, Custom and Delete filter, the `const` case, the test-handle branch, the not-created handle, the `break`, the bare symbol, the later new func, the `testing.` exclusion, both `_test` conditions, both test-support matches, the test-name prefix and the `Ref` assertion.

Tool calls:
1. Grepped `internal/planparser/normalize.go` for root and handle prefixing, to see whether handle units are root-prefixed before `isTestSupportUnit` compares them; my context lacked that file. The grep matched nothing, so it settles nothing, and I raise no finding on it.
