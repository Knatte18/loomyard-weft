```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 fix to `finalize_gate_test.go` (the pending-findings resume subtest) for agreement with its siblings and surroundings.

no findings

- The `wantLand` row now calls `fx.call(t)` and `fx.requireLanded(t, outcome, ptr)`, as the sibling subtests at `:586-588`, `:658-660` and `:677-679` do.
- The `reviewStartErr` row keeps the `Call` error assertion and adds `len(fx.merger.calls) != 0` with the same message as the sibling at `:613-615`, as the review's `fix` key asks.
- The sibling subtest also checks `shedengine.TransientOf(err)`. The review did not ask for that and the fixer left it out, so it is not a finding.
- The post-call assertions still agree with both rows. The fixer-first and `fix_review_path` checks stay, and the `FixReview`-cleared check is guarded by `found &&`, so it holds when a landing removes the record.
- The subtest's name and the row names still describe what each row asserts. The `tt.wantLand` field keeps its role as the branch selector.
- `lym gate test ./internal/landingshed` passed in the fixer's run.

Tool calls: none; the context already held the diff, the sibling subtests and the gate output.
