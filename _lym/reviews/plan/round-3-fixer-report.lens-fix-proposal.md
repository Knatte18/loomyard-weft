```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the proposals behind F17 (card 14) and F18 (card 15), both fixed as proposed.

**MEDIUM: card 15, the rewritten `TestSeedWarpExclude_SeedsWarpOnlyOnce` bullet.** It says that after the added `fabricengine.SeedWarpExclude(l)` call, the subtest's "existing once-under-`/backend` and never-at-root checks then hold the deleted subtest's rule for `SeedWarpExclude` at a subpath anchor". That claim does not hold for a `SeedWarpExclude` that writes nothing.
- Failing case: `SeedWarpExclude` becomes a no-op that returns the paths but skips `seedWarpExcludeLines`. `WireJunctions` has already seeded the `/backend/...` lines once, so both checks still pass. The deleted `anchored_hub_seeds_the_lines_under_the_anchor` calls `SeedWarpExclude` alone on a fresh hub, so it fails in that case.
- Same hole at the root anchor: `seeded_once_warp_only_and_kept_by_remove` calls `SeedWarpExclude` and then `WireJunctions`, with no line check in between.
- Evidence:
  - `internal/fabricengine/gitexclude.go:2` says `info/exclude` is repo-wide, resolved to the common gitdir, so both calls write the one file.
  - `internal/fabricengine/warpexclude.go:66`: `SeedWarpExclude`'s only write is `seedWarpExcludeLines(nil, l.WorktreePath(), l.AnchorRel, paths)`.
  - `internal/fabricengine/warpexclude_integration_test.go:149-171`: the sibling checks the lines only after `WireJunctions`.
- A change that holds the rule: in the sibling, call `SeedWarpExclude(l)` before `WireJunctions` and check the `/backend` lines (once under `/backend`, none at root) right after it. Or keep the deleted subtest.

**F17's proposal, no findings.** I built the cases and each holds against the code:
- **Transport:** a `gitexec.Run` `ls-remote` error reads as a transport failure, because `remoteSubcommands` covers `ls-remote` (`internal/gitexec/transport.go:68-69`). The old wording needed `Args` to begin with `push`; this way does not.
- **Auth refused:** `mapPlainPushError` checks the auth class before the rejection class, so a hook that prints `Permission to hub denied to tester.` maps to `ErrPushAuthRefused` even though git also prints a "rejected" line (`internal/gitrepo/push.go:278-281`).
- **Rejected:** a diverged origin tip matches the "non-fast-forward" and "rejected" triggers and maps to `ErrPushRejected` (`internal/gitrepo/push.go:283-290`). `PushWithRetry` wraps the push (`push.go:164`). I did not read which errors it retries, so a rejection being returned without a retry is unconfirmed.

Tool calls beyond my context:
1. `transport.go:60-110` and `push.go:259-330`, to check how the provoked transport, auth and rejected errors are classified; my context did not hold those bodies.
2. `SeedWarpExclude` and the `info/exclude` location in `gitexclude.go`, to check whether `WireJunctions` and `SeedWarpExclude` write one shared file.
3. `pushPlainWithRetry`, to check whether `PushRebaseFree` wraps the push in a retry.
