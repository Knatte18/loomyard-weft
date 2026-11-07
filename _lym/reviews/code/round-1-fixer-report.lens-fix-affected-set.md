```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

**Findings left unchecked:** none. F1 is the only finding.

**F1 search (set: template copies taken after a seam install):**
- In `worktreeseam_integration_test.go`, the only `newOriginHub`, `newMovesPair` and `newOriginTaskHub` calls are lines 172 and 187–190. Both sit before the install at line 193.
- The only `CopyRegistered` hit in the file is inside those helpers, so no other template copy follows the install.
- The `InstallWorktreeSeam(heddleengine.Seam{})` at line 287 is a nested install inside one subtest. No template copy follows it in that subtest.
- `internal/heddleengine/seam_integration_test.go:78` also installs the seam (`TestSeam_PullTakeIn`). It has no `CopyRegistered` call, so the finding doesn't reach it.

Tool calls:
1. `grep` of `worktreeseam_integration_test.go` for the install and copy sites. My context held the file but not the post-edit line numbers.
2. `grep` of `internal/heddleengine/seam_integration_test.go` for template copies after its install. That file was not in my context.
