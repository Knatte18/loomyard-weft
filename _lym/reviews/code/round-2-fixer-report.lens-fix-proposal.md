```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the F1–F5 proposals as the fixer carried them out. I found two LOW issues and no BLOCKING or MEDIUM ones.

**LOW — F1, `internal/fabricengine/failedpush.go` (new `RepoCode`, `RepoRecords`) and `internal/loomcli/prepush_test.go:35`:**
- The new constants cover only two of the three repo sides.
- `prepush_test.go:35` still builds `boltMain` with the bare literal `Repo: "bolt"`.
- `failedPushSideBolt` has no exported twin, so the "one declaration referenced everywhere" premise is only partly met in the file the finding named.
- Other packages' tests still use bare `"code"` and `"records"`, for example `internal/adoptcli/adopt_test.go:114`, `internal/orchcli/start_test.go:1027`, `internal/battencli/cancel_test.go:487` and `internal/fabriccli/anchor_integration_test.go:181`.
- Those sit outside the finding's named scope, and the review didn't ask for them. They show the constants don't yet bind every consumer.

**LOW — F3, `internal/loomcli/prepush_test.go` (`TestPrePushCheckWiredIntoDeps`):**
- The test calls `commitStatusDepsGuardedBy` and `pullRequestDepsGuardedBy` with an injected check.
- It does not call `loomCommitStatusDeps` or `pullRequestDeps`, the functions production calls.
- Concrete case: if `loomCommitStatusDeps` builds `newPrePushCheck(location, "")` instead of `newPrePushCheck(location, runID)`, or `pullRequestDeps` passes the wrong run id, no test fails.
- Dropping the wrapper inside the `GuardedBy` functions is caught.
- The review proposed a hub-fixture test of `loomCommitStatusDeps` itself, with an origin record naming an owner elsewhere. The fixer chose a seam split instead, which is an own change and not the proposal.
- The skip test, `TestNewPrePushCheckSkip`, covers pushes off, the prime and a plain pair. It never reaches the `CurrentProducer == Finalize` read inside the closure.

I also checked F2, F4 and F5 and found no case where the proposal breaks.
- **F2:** `ParentNoticeFile` sits under `ScratchDir` for the same run id the check is built with.
- **F4:** the busy-records skip uses the same conditions as `classifyEntries` (`internal/fabricengine/originreconcile.go:864`), and `ownBranches` already restricts to the run's own slug. A busy code branch is still named, matching `classifyEntries`.
- **F5:** the new PATTERN text matches the guard. One small gap remains: `skipsCheck` also skips on `SkipGit`, and the bullet says only "pushes off". Treat that as a nit.

Tool calls:
1. `grep` for bare `Repo` literals outside `fabricengine`. My context showed only the loomcli sites, so I needed to see how widely the literals recur.
