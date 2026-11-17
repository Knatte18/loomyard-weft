```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1 fix (`label-duplicate` in `vocabulary()`) for code correctness.

Traced inputs:
- `Labels = ["mod-a","mod-a"]`: the first pass records `mod-a` as seen, and the second pass finds it seen and not yet reported. That emits one `CheckLabelDuplicate` finding. `mod-a` is in the vocabulary (row 103 relies on it), so no `label-unknown` finding follows. The result is exactly `[label-duplicate]`, which is what the new row wants.
- `["mod-a","mod-a","mod-a"]`: the `repeatedLabels` guard gives one finding per repeated label, as the fix specified.
- `["ghost","ghost"]`: one `label-duplicate` plus two `label-unknown`, one per occurrence. This is acceptable, because each occurrence is genuinely unknown.
- `Labels = nil`: the loop body never runs, so `labels-missing` is unaffected.
- The `seenLabels` and `repeatedLabels` slices are re-declared inside the per-part loop, so state does not leak between parts.
- The new test row would fail with the defect in place: without the check, the finding list is empty and the row wants `[label-duplicate]`.
- The signature of `vocabulary()` is unchanged, and `Check` is unaffected. The only addition is a new constant.

Findings: no findings.

Calls made:
- None. My context already held all of `check.go`, the spec and the test rows. I did not verify the one thing I could not see, whether another test or doc enumerates every `Check*` constant and would need the new one.
