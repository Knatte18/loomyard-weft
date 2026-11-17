```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal, F1 (duplicate part label): no findings.

I looked for a case where the proposed `label-duplicate` check is wrong or breaks something, and found none.

- **Match rule:** the board's duplicate test, `seen[label]` in `internal/boardengine/labels.go:28-30`, is exact string equality. The new check's `slices.Contains` uses the same rule, so it neither lets through a repeat the board refuses nor refuses one the board accepts.
- **Type label vs. part label:** the filing line prepends `part.Type` to `part.Labels`. A part label equal to the type label is not in `LabelVocabulary`, which holds module labels only, so it already fails as `label-unknown`. The new check covers no further collision.
- **Repeats of an unknown label:** `["ghost","ghost"]` yields one `label-duplicate` and two `label-unknown` findings. That is noisy but each finding is accurate and has a way forward, and no test or caller expects a different count.
- **Three or more repeats:** `repeatedLabels` limits the report to once per repeated label, as the `fix` key asks.
- **Other sites naming the check set:** a grep for `label-unknown`, `CheckLabelUnknown` and `CheckLabelsMissing` finds only the spec, `check.go` and `check_test.go`, all three updated. No stencil or other doc lists the checks and was left stale.
- **Existing tests:** no other `TestCheck` row sets duplicate labels, so no existing row changes its expected findings.

Further tool calls:
1. Read `internal/boardengine/labels.go`: my context lacked the board's duplicate rule that the new check must match.
2. Grepped the repo for the check names: to see whether any sibling site listing the checks was left stale.
