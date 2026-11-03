```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F5 change set for stale twins and siblings.

Findings:

1. **LOW**, `internal/loomcli/prepush_test.go:35`. `boltMain` still builds a `BranchOutcome` with the bare literal `Repo: "bolt"`. F1 exported `RepoCode` and `RepoRecords` but not a bolt side, so this literal is the one repo-side literal left in the file. The test's own `Repo: "code"` and `Repo: "records"` literals were converted. The review's F1 fix named only the code side, so this is left over rather than a regression. No other `loomcli` literal needs changing.

2. **LOW**, `pattern/PATTERN-owner-machine.md:6` (new bullet). It says "bar those with pushes off". The guard's `skipsCheck`, and the doc comment on the `skip` field in `prepush.go`, skip the check for `SkipGit` as well as `SkipPush`. The bullet therefore omits the git-off case, which is the same under-statement F5 reported. The `## Origin reconcile first` section also still describes only the reconcile, not these skips, but that matches the review's scope.

3. **LOW**, `internal/fabricengine/failedpush.go`. `RepoCode` and `RepoRecords` are exported next to `failedPushSideCode` and `failedPushSideRecords`, and `failedPushSideBolt` stays unexported. Fabricengine's internal uses still go through the unexported names, so there are now two names for one value inside the package. This is what the review proposed ("set equal to failedPushSideCode"), so it is acceptable. It is a latent twin: a later rename of one name will not update the other.

Checked and consistent:
- The `codeRepo` and `parentNoticeFile` references are gone from non-fabric production code. The `codeRepo` hits in `reportcli` are unrelated local variables.
- `ParentNoticeFile` is added to `paths.go`, `paths_test.go` and `doc.go`, and the `prepush.go` call site uses it.
- `commitStatusDepsGuardedBy` and `pullRequestDepsGuardedBy` are used by the new test. The doc comments on `loomCommitStatusDeps` and `pullRequestDeps` still hold.
- The F4 `stoppedBranches` doc comment matches `classifyEntries` at `originreconcile.go:864`.

Tool calls (1):
- One grep for repo-side literals, `codeRepo`, `parentNoticeFile` and the notice file name outside `fabricengine`, plus a grep of `pattern` and `docs` for other mentions of the pre-push check. My context held only the files the parent had read, and it lacked the other call sites.
