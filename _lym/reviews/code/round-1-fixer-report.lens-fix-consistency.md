```yaml
lens: fix-consistency
outcome: reported
```

No findings.

The remaining "test-map refreshes" wording in `internal/dobbyengine/config.go:36` and `:132` is correct. It describes the retired landing-count setting, not the running pass.

The other `dobbyengine` "pass this dobby started" lines are accurate. The dobby does start the pass.

I did not check these things:
- The F1 fix leaves `spawnPass` (the batten pass) writing to a `tailBuffer` pipe. That twin was outside F1's scope, which names only `startTestMap`.
- `readFileTail` and `outputPath` match the new test comments.
- `RoleTestMap` in `internal/runproc/record.go:50` already says the pass registers itself, so the orchcli rewording agrees with it.

Tool calls made:
1. A grep for stale refresh, upkeep, `tailBuffer` and pipe wording in `*.go` and `*.md` under `internal` and `contracts`. Together with a read of `internal/runproc/record.go` lines 48-52, it checked for twins of the reworded text outside the files the fixer changed. My context only held the files that were read or edited.
