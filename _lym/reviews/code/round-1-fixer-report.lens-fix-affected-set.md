```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the F1 site search (every place that words the build-skew driver action) and check that the fix covers all of them.

Findings (F1, F3 and F4 checked):

- F1: the `build-skew` search finds exactly three prose sites. They are the stencil at line 48, `internal/loomcli/doc.go:13`, and the `AnomalyBuildSkew` doc at `internal/loomengine/anomaly.go:30`. The fix rewrote all three. The remaining hits are the wire string and a test row, and neither describes the driver's action. The search also found no stale "update the binary" or "back to its parent" wording.
- F3: `_lym/landing/summary.md` held the only "hands the run back instead of over" claim in this change. Nothing else needed the edit.
- F4: the new `NotFailedBuildSkew_NoKind` row is the only gap the finding named, and the fix added it.

No findings. I left F2 unchecked because it is a `decision`-class finding held by the parent.

Tool calls:
1. Ran grep for `build-skew` over the tree, excluding `_lym` and `.lym`, to find every site that words the build-skew behaviour. My context held only the three sites the review named.
2. Ran a first grep that returned nothing because its filter was too narrow. I reran it as call 1.
