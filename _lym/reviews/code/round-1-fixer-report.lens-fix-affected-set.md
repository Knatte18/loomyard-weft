```yaml
lens: fix-affected-set
outcome: reported
```

No findings. F1's premise is a set: the sites that can reach `commitWeftAt`'s seed-failure branch. The fix covers that set.

- `commitWeftAt` has one non-test caller, `Bolt.Commit` (`bolt.go:38`), which just forwards to it. Every test of the branch therefore goes through `commitWeftAt` directly, and `TestCommitWeftAt_SeedFailureFailsBeforeStaging` does that.
- The other `seedWeftArtifactExcludes` callers (`recordsgit.go:64`, `junction.go:127`, `clone.go:336`, `reconcile.go:375`) are separate branches. F1 does not name them.

Calls used:
1. `grep -rn "commitWeftAt\|seedWeftArtifactExcludes"` over all Go files, to enumerate the callers and tests of the finding's set. My context held only the files the fixer read, not the rest of the repository.
