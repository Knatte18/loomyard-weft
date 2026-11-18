```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the F1–F3 change set.

No findings.

I checked that the new docs agree with the code and its callers:
- The `PushPairAnchored` doc, the `recordsSideNotLanded` doc, `doc.go` line 1136 and the `IsPushOwed` docs all describe the same two outcomes as owed pushes: a spent backoff as `deferred` and a transport-failed lookup as `transport`. The `IsPushOwed` text already lists `transport` and `deferred`, so it needs no edit.
- The only caller of `recordsSideNotLanded` is `pushRecordsSide`, and it passes the new `opts` argument.
- The other `BackoffSleep` docs (`add.go:41`, `doc.go:1516`, `doc.go:1539`, `pushclassified.go:166`) don't list uses in a way that contradicts the reworded `fabric.go:82`.
- The `finalize.go:762` caller maps an owed push to a transient halt, which matches what F1 intends.

I made two further tool calls, both reads:
1. A grep for `IsPushOwed`, "owed" wording and `BackoffSleep` doc twins across the package. My context did not hold the sibling docs or callers.
2. A read of `doc.go:1314-1326` and `landingshed/finalize.go:755-770`. These are the `IsPushOwed` description and a caller whose behaviour the change reaches, and my context did not hold them.
