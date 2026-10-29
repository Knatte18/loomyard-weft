```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- F2: the only tier-name join left is `tierNames` in `scope.go`.
- F1: the only passing `--card` row that plans vet runs is the one the fix covers; the failing vet row asserts no line, and the other `--card` rows plan no vet run.
- F3: set premise not applicable; `doc.go:10` already describes the shorter wording.

Tool calls:
1. grep of `tierName(` call sites in non-test `internal/gatecli` files, for F2; my context held only `run.go`, `tiers.go` and `tierNames`.
2. grep of the `--card` and vet-args rows in `scopes_integration_test.go`, for F1; my context showed only lines 380-540.
