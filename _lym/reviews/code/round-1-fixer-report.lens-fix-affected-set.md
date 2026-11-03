```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run each set-premise finding's search and check the fix covers every member.

**Findings**

1. **MEDIUM**, `internal/loomcli/commitrecords.go:43-62` with `prepush.go` `guardedCommit`.
   - When the check stops `Commit`, it returns `(false, nil)`. `commitRecordsVerb` discards `committed` (`_, err := d.Commit(...)`) and then calls `d.Push`.
   - `Push` runs the guard again. The pause the first stop wrote now covers the run, so `skip` is true and the guard passes. `Push` returns nil, and the verb reports `"committed": true` with no `push_stopped`.
   - That is wrong in two ways: nothing was committed, and the stop is invisible to the caller. The review's F2 premise was that a stop must reach `commit-records` as `push_stopped`. A stop at the commit step never does.
   - Whether the push then really goes depends on the pause's network gate, which I did not verify.
   - F6's fix makes `Commit` the first stop point in this verb. The envelope doc in `commitrecords.go:29-30` covers only a stopped push.

2. **LOW**, `internal/loomcli/drivewiring.go:260`.
   - This is a fourth entry point, `commitRecordsVerb(&envelope, loomCommitStatusDeps(location, c.runID))`. It inherits the same `Commit`-stop behaviour as finding 1.
   - It was not named in the review. Resolving finding 1 in `commitRecordsVerb` resolves this too.

**Sets rerun**

- **F6 and F8, pushes and commits of the run's pair.** The `PushPairAnchored` call sites are `wiring.go:221` and `landingdeps.go:340`. Both are wrapped in `guardedPush`. `spawnStatusPush` is called only inside `guardedSpawn`. The status `CommitAnchoredPaths` is wrapped in `guardedCommit`. No unguarded push site remains in `loomcli`, `landingshed` or `shedverbs`.
- **F4, the `stops()` rename.** No `.stops()` use remains in `.go` files. The remaining `.stops` hits in the earlier grep were unrelated test fields.
- **F9, docs naming the verdict gates.** Only `PATTERN-owner-machine.md` and `loomcli/doc.go` carry the text. Both were updated.

**Not checked:** the other `CommitAnchoredPaths` sites (`CommitWebster`, `CommitPlan` and similar). The review named only the status commit, so I left them out.

**Tool calls**
1. `grep` for push, commit and guard call sites in `loomcli`, `landingshed` and `shedverbs`: my context lacked the full call-site set.
2. `grep` for stale `stops()` and pre-push or verdict-gate text across `.md` and `.go`: my context lacked a repo-wide search.
