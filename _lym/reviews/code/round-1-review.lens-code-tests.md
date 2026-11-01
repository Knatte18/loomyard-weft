```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it, or none.

I made no tool calls and judged from the diff and test files already in context.

**Behaviour → pinning test**

1. One lock per repository, shared by all its worktrees (`HoldRemoteRefs`, lock file in the common git dir): pinned by `TestHoldRemoteRefs_OneLockServesEveryWorktreeOfARepository`.
2. `HoldRemoteRefs` waits instead of failing: no direct test, no row blocks a second holder. `TestRemoteRefsLock_ConcurrentFetchesAndPushesOfOneRepositoryAllSucceed` only shows concurrent calls succeed.
3. `Fetch` and `PushRebaseFree` (via `pushPlainWithRetry`) run under the lock: only the concurrency test, which is probabilistic. `PushRebaseFree` already retries "cannot lock ref", so it can pass with the lock removed. Nothing deterministic fails if either method drops the lock.
4. `Pull`, `FetchWithArgs`, `Push` and its `pull --rebase` step, `PushCoalesced`, `DeleteRemoteBranch`, `DeleteRemoteBranchLeased` and `UpdateRemoteBranchLeased` take the lock: none.
5. `Pull` and `Fetch` keep their exit-code handling and error text (`code` assigned in the closure and read after): the existing `pull_test.go` and `fetch_integration_test.go` pin the no-`fatal:`-leak surface for a non-zero exit. The new lock-acquisition-failure branch has no test.
6. The eight fabricengine fetch sites (`add.go` ×2, `archive.go`, `checkout.go`, `cleanupremotewarp.go`, `remoteonly.go`, `remove.go`, `taskbranchdiscard.go`, `weftbranch.go`) go through `FetchWithArgs`: no test fails if one reverts to a bare `gitexec.Run` fetch. Existing behaviour tests only show the fetch still works.
7. `ReconcileOrigin` holds both repositories' locks across the `ls-remote` pair and the moved-heads fetch, then frees them: pinned by the new subtest "A reconcile holds both repositories' remote-refs locks…" in `originreconcile_integration_test.go`. It checks at each hook call and after return, on the moving path with one moved code branch.
8. The lock is held through the branch verdicts (`judgeBranch`) and not only through the remote calls: none. The hook fires only on `ls-remote` and `fetch`, so a `judgeAgainstOrigin` that released the locks right after the fetch would still pass. That is the case the record's "push in flight finishes before origin is judged rewound" depends on.
9. The locks are released on the unreachable and error returns, and on the judge-only scope (`fetchMovedHeads` with `judgeOnly`): none. The subtest runs `ReconcileScope{}` against a reachable origin only.
10. The code repository's lock is taken before the records repository's: none. The subtest checks both are held and does not check order.
11. A push already in flight lands before origin is read, so it is never judged a rewind (the 14:46 incident): none. No test holds a lock, reads origin mid-push, and then asserts the verdict.
12. Backoff sleeps in `pushPlainWithRetry` run outside the lock: none. `PushWithRetry` is exercised with an injected sleeper, but nothing checks the lock is free during the sleep.
13. Boundary list gains `FetchWithArgs`: pinned by `TestGitrepoBoundary_PinnedRunCallSites`, a set-equality scan.

**Findings**

- **MEDIUM, design** — `internal/gitrepo/remoterefslock_integration_test.go:68`. The new tests do not pin the lock deterministically for any of the nine locked methods in rows 3–4. The concurrency test can pass with the lock removed. Evidence: `PushRebaseFree` retries "cannot lock ref" in `push.go:151`, and the test asserts success only. A table test would fix it. For each method it would hold the lock with `HoldRemoteRefs`, start the call in a goroutine, assert it has not returned, then release and assert it completes.
- **MEDIUM, design** — `internal/fabricengine/originreconcile_integration_test.go:227`. Row 8 and the verdict part of row 11 are uncovered. The hook sees only `ls-remote` and `fetch`, so releasing the locks before `judgeBranch` would pass. The fix is a case that probes the locks again after the fetch, or a held-lock case showing the verdict waits.
- **LOW, design** — `internal/fabricengine/originreconcile_integration_test.go:227`. Release on the unreachable path, the error path and the judge-only scope is unpinned, which is row 9. The `defer` in `judgeAgainstOrigin` makes this likely correct. Add rows to the same subtest, using a transport-failure hook and `JudgeOnly: true`.
- **LOW, scope** — `internal/fabricengine/add.go:415`, and the seven other sites in row 6. No test fails if a fabricengine fetch site reverts to an unlocked `gitexec.Run`. The record asks that each be put under the lock. Cheapest cover is an AST scan like `gitrepoboundary_test.go`, flagging `"fetch"` literals in `internal/fabricengine` non-test code outside the pinned `--refmap=` sites.
- **LOW, design** — `internal/gitrepo/pull.go:21`. The lock-acquisition-failure branch of `Pull` and `Fetch` is untested, where `err` is returned before `code` is read. A row with an unusable common git dir would pin the unchanged-text claim.
- **NIT, design** — `internal/gitrepo/remoterefslock_integration_test.go:43`. `TestHoldRemoteRefs_OneLockServes…` and the concurrency test both start with `publishedClone`. That is shared setup, not duplicate coverage, so I raise nothing.

Tool calls: none.
