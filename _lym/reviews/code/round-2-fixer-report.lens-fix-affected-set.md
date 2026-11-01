```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Searched F1's set: every direct `fx.fz.Call` in `internal/landingshed/finalize_gate_test.go`. Four sites turned up, one in the helper `fx.call` and three in subtests. After the fix:
- **Line 86:** it is the `fx.call` helper itself.
- **Line 552:** the "an approval is kept in the attempt record" subtest. It deliberately stops the landing with `pushErr`, and it asserts `len(fx.merger.calls) == 1` plus the `Reviewed` record, so it does not rest on the error alone.
- **Line 608:** the "fix review session that fails to start" subtest, which already asserts the transient error and zero merges.
- **Line 712:** the fixed site. The `wantLand` row now goes through `fx.call` and `fx.requireLanded`. The `reviewStartErr` row asserts an error and `len(fx.merger.calls) == 0`.

Calls made this round:
1. `grep -n 'fx.fz.Call\|err == nil)' internal/landingshed/finalize_gate_test.go`: to list every direct `Call` site and error-only check in the file. My context showed only part of the file.
2. `Read finalize_gate_test.go` lines 535–562: to confirm line 552 asserts more than the error. My context held that region only as the fixer's earlier read, not as a changed form.
