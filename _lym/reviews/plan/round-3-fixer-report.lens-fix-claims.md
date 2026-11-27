```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote in cards 14 and 15 holds against the code at HEAD.

**MEDIUM: card 15, the rewritten `TestSeedWarpExclude_SeedsWarpOnlyOnce` line (old line 149)**
- **Changed text:** "its existing once-under-`/backend` and never-at-root checks then hold the deleted subtest's rule for `SeedWarpExclude` at a subpath anchor."
- **Summary:** That claim is only partly true. With `SeedWarpExclude(l)` called after `WireJunctions`, the checks catch root lines and duplicate lines. They do not catch a `SeedWarpExclude` that seeds nothing. The deleted subtest caught that case, because it checks for exactly one `/backend` line after `SeedWarpExclude` runs alone.
- **Evidence:**
  - `internal/fabricengine/warpexclude.go:36-37` skips any line already present (`if present[line] { continue }`).
  - The pair worktree and the prime share one exclude file. Subtest `seeded_once_warp_only_and_kept_by_remove` (`warpexclude_integration_test.go:68-81`) seeds through `SeedWarpExclude(l)` on the prime, then checks `requireWarpExcludeOnce` on the pair's worktree.
  - So after `WireJunctions` has seeded `/backend/...`, the added call finds every line present and writes nothing. A broken `SeedWarpExclude` that wrote nothing would still pass the "once" and "never at root" checks.
  - The deleted subtest (`warpexclude_integration_test.go:132-147`) runs `SeedWarpExclude` alone and wants each `/backend` line exactly once.
  - The claim comes from the reviewer's proposed fix.
- **Possible change:** call `SeedWarpExclude(l)` before `WireJunctions` and run the anchor checks between the two calls.

The other changed claims hold:
- **Card 14, auth refused:** `PushRebaseFree` returns `ErrPushAuthRefused` for a hook that prints `Permission to hub denied to tester.`. `mapPlainPushError` checks the auth triggers before the rejection triggers (`push.go:278-280`).
- **Card 14, rejected:** a diverged origin tip gives `ErrPushRejected`. The `PushRebaseFree` doc says so and that function never rebases (`push.go:248-251`).
- **Card 14, transport:** an `ls-remote` refused by the remote counts as a transport failure. `ls-remote` is one of the remote subcommands `IsTransportFailure` accepts (`transport.go:67-68`). The `connection refused` phrase matches, per the `refusedOriginURL` doc at `pushclasses_integration_test.go:30`.
- **Card 15, line 136:** the refusal returns the literal `PushResult{}` before any push step runs (`pushanchored.go:210`). `TeardownLockHeldPastTheWaitOwesTheRecordsPush` discards the result (`_, err :=`).
- **Card 15, line 144:** `RefusesMainWorktree` and `AcceptsPrimeWarpWorktree` both call on `repoDir == l.WorktreePath()`. That matches the two calls the deleted subtest made (`destructivegaps_integration_test.go:468-525`).

Tool calls:
1. Read `push.go:255-330` and grepped the trigger lists, to see how `PushRebaseFree` sorts auth refusals from rejections. My context had only lines 20-64.
2. Read `SeedWarpExclude` and `seedWarpExcludeLines`, to see whether it skips lines already present. My context did not have the code.
3. Read the `PushRebaseFree` doc at `push.go:236-256`, to confirm it never rebases and returns `ErrPushRejected` for a non-fast-forward. That doc was not in my context.
4. Read `transport.go:40-110`, to confirm `ls-remote` counts as a remote subcommand for `IsTransportFailure`. My context had only the function's signature line.
