```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the fix-affected-set lens on this round's change set, covering any finding whose premise is a set.

Findings left unchecked: none.

no findings

The review holds no findings and the fixer changed no file. The only set in play is the gate's three functions with no mapped test, and each has a direct tier1 test in what the context already shows:
- `hubCancelDeps`: called in `internal/battencli/cancel_integration_test.go:229`, and the `TestCancelIntegration_*` tests that ran passed.
- `Topology.SettleGonePairTeardown`: called in `internal/fabricengine/remoteteardown_rerun_integration_test.go:100` and `:108`, and `TestRemove_PendingRecordNamesTheRerunItsCallerGave` passed.
- `Topology.RemoveLocal`: exercised by `TestRemoveLocal`, which passed.

Tool calls made: none.
