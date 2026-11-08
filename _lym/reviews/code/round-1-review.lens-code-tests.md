```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (code-tests lens)**

The diff adds no production behaviour. The only added or changed test-side behaviours are these:

- **Coalesce test file no longer changes cwd.**
  `TestCwdMutation_MigratedFilesStayChdirFree` pins it. The file stays in `cwdMutationSubjectFiles`, the allowlist is empty, and the file has no `Chdir` token (grep returned none).
- **Guard matcher fires on a planted violation.**
  `TestCwdMutationGuard_NotVacuous` pins it. It keeps the `t.Chdir(` planted case, and `//lym:watch` is narrowed to scankit only, which matches the dropped allowlisted-file read.
- **Empty allowlist is accepted.**
  `TestCwdMutation_MigratedFilesStayChdirFree` pins it, through `scankit.NewAllowlist` on an empty slice at `cwdmutation_test.go:55`. This is only compile-level assurance. I did not read `NewAllowlist` to confirm it tolerates an empty slice, but the gate would have failed if it didn't.
- **`refuseRecordsCommits` moved.**
  It is used at `mutation_record_integration_test.go:49`, so the one remaining caller still exercises it.
- **Dropped tests and removed `export_test.go` shims** (`WorktreeDirtyTrackedForTest`, `WorktreeDirtyAllForTest`).
  No test pins these. They were deletions, and the handler covers scope.

**Findings:** none.

**Tool calls**
1. A grep of `Chdir` in `coalesce_integration_test.go`, plus the `refuseRecordsCommits` and `cwdMutationAllowlist` usages. The manifest lacked the post-diff file contents.
