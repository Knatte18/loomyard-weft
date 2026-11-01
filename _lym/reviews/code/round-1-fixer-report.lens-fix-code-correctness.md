```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fixer's code and test changes through the correctness lens. I made no tool calls and read nothing beyond my context.

Findings:

1. **LOW**: `resolveArchiveTip` in `internal/fabricengine/archive.go` (`tip := strings.Fields(listed)[0]` followed by `FetchWithArgs(originRemoteName, tip)`).
   - The new code fetches by SHA instead of by `refs/heads/<branch>`, which narrows behavior in one case. Take the case where the branch moves between the `ls-remote` and the fetch. The old code fetched the branch name and succeeded with the newer tip. The new code asks for a SHA the server may no longer advertise. A server on protocol v0/v1 without `allowReachableSHA1InWant` then fails with "not our ref".
   - The failure is an error returned through the unchanged "fetch weft branch" text, not a wrong tag, so no wrong result ships. The archive-tip tests passed against a local bare origin, which cannot show this case.
   - Separately, `ls-remote --heads origin refs/heads/X` matches by tail. If a second ref ends in `refs/heads/X`, `Fields(listed)[0]` could pick the wrong line. The old refspec fetch named the exact ref. This is unlikely, since a branch name would have to contain `refs/heads/X`.

2. **LOW**: the added `judgeAgainstOrigin` doc comment in `internal/fabricengine/originreconcile.go` ("bounded by opts' attempts and deadline").
   - The only retry fields visible in my context are `RetryBound` and `RetryInterval` (tests at `originreconcile_integration_test.go:469-470`). The doc names "attempts and deadline", which are not those fields.
   - Each ls-remote attempt also has its own attempt deadline, per the `lsRemotePair` doc. The bound itself is correct; only the wording could mislead a reader.

Checked, no finding:
- **`PushRebaseFree` doc:** only the comment changed.
- **`judgeAgainstOrigin` hook:** `callOriginVerdictHook()` sits inside the verdict loop, so the deferred releases still run after the last verdict. If the locks were released after `fetchMovedHeads`, `freeDuringVerdicts` would be non-empty and the subtest would fail.
- **Release-path rows:** if the `defer` were dropped, the rows (judge-only, unreachable, no-repository) would leave the locks held and fail `freeRemoteRefsLocks`.
- **`TestRemoteRefsLock_EveryFetchPullAndPushMethodWaitsWhileTheLockIsHeld`:**
  - It can never fail falsely, because a blocked call never returns inside the window.
  - Removing the lock from any of the eight methods makes the call return in well under 500 ms, so `t.Fatalf` fires.
  - After release, every row's call completes without backoff: nothing to push, a missing remote ref, and a bad object all fail fast.
- **`cmd/lym/remoterefslock_test.go`:**
  - A `[]string{"fetch", ...}` literal without `--refmap=` outside the two allowlisted files fails the test.
  - The stale-allowlist check proves the scan sees the allowlisted literals.
  - The new helper names (`startsWithFetch`, `holdsString`, `stringValue`) compiled and ran in package `main`.
  - A regression inside `recordsprobe.go` or `originreconcile.go` stays invisible, because the allowlist is per file.

Calls made: none; my context held every changed file and the test outputs.
