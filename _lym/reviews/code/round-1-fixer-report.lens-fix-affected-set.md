```yaml
lens: fix-affected-set
outcome: reported
```

no findings

Both sets are covered:
- **F1:** `hubDobbyDeps` at `internal/beamcli/dobbydeps.go:93` is the only production site that builds a filled `dobbyengine.Deps`. Its one `TaskLimit: proc.ReadTaskLimit` at line 146 serves both the beam and the prime, and the new test checks both returned deps. The two `dobbyengine.Deps{}` sites in `dobbytrace_integration_test.go` are error-return zero values, so they need no check.
- **F2:** `taskLimitHolds` has one call site, `internal/dobbyengine/starts.go:99`, inside the per-task loop right before `startFirstPass`. The new subtest therefore covers every start path.

Unchecked: none.

Tool calls made:
- One `grep` over `internal` and `cmd` for `TaskLimit`, `taskLimitHolds` and `dobbyengine.Deps{`. My context did not hold the full set of sites that build `Deps` or call the hold.
