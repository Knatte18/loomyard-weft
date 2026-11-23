```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff's changed functions and branches.

Cases I ran:
- `runStateExemptRels` (`internal/preflight/preflight.go:28-35`), case: anchor `"."`. `UsageDirRel()` is `_lym/usage` (`internal/shedrun/usage.go:29-31`). `ScopedPathspec(".", …)` gives `_lym/usage` (`internal/fabricengine/fabric.go:188-193`). `exemptPathspec` then adds `:(exclude)_lym/usage` after `.` (`internal/fabricengine/warpclean.go:52-61`). `git status --porcelain . :(exclude)_lym/usage` leaves out `?? _lym/usage/session-<id>.json`. Correct.
- Same function, case: anchor `"backend"`. The pathspec is `:(exclude)backend/_lym/usage`. That is rooted like the other entries, which also go through `filepath.Join(lymdirs.LymDirName, …)`: `RunsRootRel`, and `LoomFrictionDirRel` via `LoomDurableDirRel`. Correct.
- Same function, case: a usage record that is tracked and then modified (` M`), or deleted (` D`). An exclude pathspec drops a path whatever its status letter. Correct.
- Same function, case: an untracked usage directory. Git's porcelain shows only the directory, `?? _lym/usage/`. Excluding `_lym/usage` still matches it. Correct.
- Same function, case: a path outside the set, such as `_lym/plan/plan.md` or the sibling `_lym/usage-x`. A plain-path exclude matches `_lym/usage` and the paths under it, not a sibling prefix. So these still count as dirty. Correct.
- The record's lock path. `writeUsageRecord` puts it through `lock.TrackedFileLockPath` (`internal/lock/tracked.go:29-44`). The lock goes in the never-tracked lock directory, not under `_lym/usage`, so it does not create a new dirty path that the exemption misses. Correct.
- Failure path: `Clean` returns an error. That error propagates unchanged (`preflight.go:89-92`), and the diff does not touch that branch. Correct.
- Test addition (`internal/preflight/preflight_integration_test.go:476-479`), case: `WriteSessionUsage` fails. The test stops through `t.Fatalf`, so it reports nothing as a pass. The session id is a single path segment, so the writer's id guard lets it through (`usage.go:95-98`). Correct.

Findings: none.

Tool calls:
- Read `ScopedPathspec` in `internal/fabricengine/fabric.go`. My context had its call from `weftCommitPathspec` but not its body, and I needed the body to confirm the pathspec is rooted at the anchor.
