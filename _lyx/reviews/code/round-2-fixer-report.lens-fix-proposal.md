```yaml
lens: fix-proposal
outcome: reported
```

- F2, LOW: `TestLoomArbiterSeam_JudgesFirst` pins the builder, not the call from `goDriverDepsFor`; replacing the `loomArbiterSeam` call with a bare `shedarbiter.Seam` keeps every test green.
- F3, LOW: the row "master renders each parent decision in order under the decision section" still names a placement it no longer asserts; rename it or accept the trade-off.
- F1 and the none row: no finding.

Further tool calls: one grep of reentryStep in internal/websterengine/runlevel.go.
