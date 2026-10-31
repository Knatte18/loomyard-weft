```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Task: rerun the review's set-premise searches (F5 stale "unwrapped" claims, F1 `HasUnpulled` callers, F3 divergence-text pins) and check the fix covers every member.

- **F5, sites claiming the rejection is unwrapped:** only `docs/benchmarks/test-redundancy.md:713` still names the old test `...ReturnsErrPushRejectedUnwrapped`. It is a captured measurement report, which the prose skill exempts from staleness fixes, so it is not a finding. `gitexec/doc.go:5` uses "unwrapped" for a different meaning.
- **F1, `HasUnpulled` callers:** the callers are `pushclassified.go:214` (`repushOverOriginContained`) and `pushclassified.go:331` (`failRejection`).
  - The retry caller reads a missing upstream as true and returns `errOriginAhead`. `settleRejection` then returns the original `rejected` error, which reaches `failRejection`, and the new `UpstreamSHA` guard there records `other`. So the fix covers that path too.
  - No other production caller exists.
- **F3, "remote diverged" text:** the remaining hits are the unrelated `ErrWarpDivergedUnpushed`, doc comments and research docs. No other test pins the old sentinel text, and the new assertion in `push_test.go:260` covers the `PushRebaseFree` sentinel.

Calls made:
1. One combined `grep` (three searches: "unwrapped", `HasUnpulled()` callers, "remote diverged") because my context held only the files read, not the repo-wide sets.
