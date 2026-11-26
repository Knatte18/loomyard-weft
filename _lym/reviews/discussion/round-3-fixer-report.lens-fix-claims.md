```yaml
lens: fix-claims
outcome: reported
```

Task: check that every fact the fix states about the code holds at this worktree's HEAD.

no findings

Every code claim the fix made holds:
- **`squash` defaults to true:** `internal/landingshed/template.yaml:2` reads `squash: true`.
- **Finalize's merge-in runs a non-squash `MergeStart`:** the path is `mergeresolve`, then `Fabric.MergeIn`, then `mergeIn`, ending at `f.code.MergeStart(sources.warpSHA, false)` (`internal/fabricengine/merge.go:277`). That call runs `merge --ff --no-commit` (`internal/gitrepo/merge.go`).
- **Teardown's commit seeds the exclude before it stages:** `commitRecordsPathsTry` in `internal/fabricengine/remove.go:857` calls `ensureWeftLockDirAt` before it stages.
- **The status commit seeds the exclude before it stages:** `CommitAnchoredPathsWithTrailers` calls `CommitAnchoredPaths`, which calls `CommitRecordsPaths`. That function calls `ensureWeftLockDirAt` before it takes the lock and stages (`internal/fabricengine/commitweftpaths.go:49`). `ensureWeftLockDirAt` calls `seedWeftArtifactExcludes` (`recordsgit.go:52`), and `commitWeftAt` seeds before its own stage-all.
- **Wiring seeds the exclude:** `wireJunctionSet` calls `seedWeftArtifactExcludes` (`internal/fabricengine/junction.go:112`).
- **The exclude covers every linked records worktree:** the doc comment of `seedWeftArtifactExcludes` says it lives in the common gitdir.
- **`pullrequest.Client.Merge` returns only an error:** confirmed at `internal/pullrequest/client.go:128`.
- **The merge-in is in `Finalize.mergeParent`:** it calls `fz.resolver.Resolve` ("merge-in against parent branch").
- **The cited test pins the empty squash:** `TestMergeCrucible_DerivedAlreadyUpToDateIsReadFromTheRecord` shows that an empty squash from a source that is no ancestor reports up to date.
- **Both resets keep the work:** the protected-branch way forward and `adopterFirstRepair` both reset to `origin/<parent>`, which holds the work.

Calls made:
- Read `template.yaml`, `commitweftpaths.go` and the merge-resolve call sites, to confirm the squash default, the status commit's seeding and the merge-in path; my context did not hold them.
- Read `fabricengine` `mergeIn` and `remove.go` around the teardown commit, to confirm the non-squash `MergeStart` and teardown's seeding; my context did not hold them.
