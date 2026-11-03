```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches behind F1–F4 (callers of every changed signature, and the paths that end a job with no notice) and check the fix covers every member.

**F1/F2/F4 callers.** Covered.
- `noticeRemoteTeardown` and `failedRemoteSteps` have call sites only in `RunTeardownChild` (both branches pass `weftRoot`) and `remoteteardown_test.go`.
- `spawnTeardownHold` is called only from `holdTeardown` and the new test.
- `warnWhenEnded` has its one caller updated.
- `NoticeRemoteTeardownEnded` has three call sites: the signal watcher, `childHold.finish` and the new test.
- The other `childHold{}` literals (`remoteteardown_childhold_integration_test.go:200` and `:250`) leave `hubPath` empty. Their writes succeed, so no notice fires.
- The three `holdTeardown` callers (`cleanup.go:278`, `remove.go:213`, `taskbranchdiscard.go:36`) keep their signature.

**F3 unreadable-job notice.** Covered. Both read failures (`remoteteardown.go:382` and `:392`) go through `noticeUnreadRemoteTeardown`, which now uses the shared way forward.

**F4 `weftRootErr` set.** Covered. `weftRootErr` is read only in `failedRemoteSteps`. The `resolve weft repo root` text elsewhere (`add.go`, `reconcile.go`, `prune.go`, `weftwiring.go`, `cleanupremotewarp.go`) belongs to other verbs and is not part of this premise.

**F2 job ends with no notice.**

- **[LOW] `RunTeardownChild`, the `answer the teardown parent` return (`remoteteardown.go:368`)** — still logs only.
  - **Summary:** this is the one remaining path that ends the child after it took the lock and before it reads the job, with no creel notice.
  - **Evidence:** it returns `fmt.Errorf("answer the teardown parent ...")`, and the CLI's `logger.Warn("fabriccli: teardown child failed")` is the only trace.
  - **Scope:** it fires only when the parent's stdout pipe is already closed. In that case `spawnTeardownHold` has already returned an error to the user (or the parent has died), so the operator is not left blind. I rate it LOW and arguably out of the review's two named paths.

No other set member found.

Tool calls:
1. A grep across the repo for every changed symbol and constructor, because my context held only the edited files, not other callers.
2. A grep of `remoteteardown.go` for `logger.Warn` and `return err`, to find log-only exits, plus a repo grep for `weftRootErr` and `resolve weft repo root`, because my context had no search of the other sites.
