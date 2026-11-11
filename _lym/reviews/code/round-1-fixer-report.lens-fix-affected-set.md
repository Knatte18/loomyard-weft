```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The review's finding names `upstreamAnchorStop`, which has two call sites: `pull.go:199` and `pushclassified.go:647`. The push path was already pinned by the changed anchor-change row, and the new test `TestPull_UpstreamAnchorStopRunsInTheAnchoredDir` covers the pull path.

I did not check whether the other `lymVerbDir` calls (`pull.go:457`/`468`, `failedpush.go`, `originreconcile.go`) have anchored-hub coverage. The review's fix and finding named only the pull-path `upstreamAnchorStop` call, so I left them out of scope.

Unchecked: none.

Tool calls:
1. `grep` for `upstreamAnchorStop(` and `lymVerbDir(` across `internal`. My context lacked the full call-site list for both symbols.
