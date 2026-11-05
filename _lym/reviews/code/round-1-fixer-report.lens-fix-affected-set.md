```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Sets I checked:
- **F1 (lock built beside the file):** I grepped every non-test `+ ".lock"` and `.swaplock` construction. The sites that build a lock from a record path are:
  - `treadleengine`, `websterengine`, `shuttleengine` and `driftengine`
  - `dobbyengine`, `adoptcli`, `pauserecord`, `pawlengine` and `battencli`
  - `fabricengine`: `corrindex`, `index`, `failedpush` and `moverecord`
  - `shedrun.StatusLock`
  
  Each sits under `.lym` scratch or in a git dir (`corrIndexPath` is `weftGitDir()` plus a file name), so none sits beside a `_lym` or `_bolt` file. The new refusal now blocks any other such site at run time, in both trees.
- **F3 (record-lock sites):** the four `lock.TrackedFileLockPath` sites in `settlerecord.go` are the only record locks under a `_lym` run directory. `shedrun.StatusLock` already uses `ScratchDir` (under `.lym`).
- **F5 (stale `.gitrepo-push.lock` mentions):** none remain in non-test code. The remaining hits are:
  - Test comments, in `internal/fabriccli/cli_test.go:157`, `internal/fabricengine/coalesce_integration_test.go:3,58-59` and `internal/gitrepo/push_test.go:478,492,496`. They still name `.gitrepo-push.lock`, which is now `lym-push.lock`.
  - The legacy exclude lists in `recordsgit.go:44` and `weftgit_exclude_test.go`, which intentionally name old files.

Unchecked: these sites were not traced to their path origin, because the call cap ran out: `fabricengine.mergeStatePath`, `fronttask.lockPathOf`, `battenshed` `innerRunMemoryLockFile` and `fabricengine/failedpush.go:103`. The new run-time refusal covers them if any sits in a tracked tree.

- **LOW** — the stale test comments above. They are outside the production-code set F5 named, and they are not a regression from this change.

Tool calls:
1. Grep of non-test Go files for `+ ".lock"` and `.swaplock` constructions, because my context held no such site list.
2. Grep of the repo for `gitrepo-push`, because my context held no stale-mention list.
3. Grep of non-test calls to `state.ReadJSON`, `WriteJSON`, `WriteFile` and `UpdateJSON` with a lock argument, because my context held no caller list.
4. Read of `shedrun` `StatusLock` and `RunDir` and of `driftengine` `recordLock`, to see whether those lock paths sit under `.lym`.
5. Read of the path origins in `fabricengine` (`mergestate`, `corrindex`, `index`) and `fronttask`, to see whether those lock paths sit under `.lym`.
