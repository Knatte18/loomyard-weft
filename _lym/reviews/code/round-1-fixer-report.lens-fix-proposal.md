```yaml
lens: fix-proposal
outcome: reported
```

No findings.

I tried to build a failing case for each proposal and couldn't.

- **F1 (`!entry.Prunable &&` at `originreconcile.go:640`):** a run scope with a prunable task worktree and no `PairMayBeAbsent` now returns the "no pair worktree" error. Before, the default branch ran and the entry was silently never classified. I checked the run-scope callers for anything that relied on the old tolerance:
  - `fabriccli/remove.go:212`, `loomcli/landingdeps.go:662`, `loomcli/ownercheck.go:58`, `battencli/wire.go:1067` and `battencli/resumereconcile.go:37` pass no `PairMayBeAbsent`. A prunable pair already took the skip at `:663`, so it reached no branch of its own there either. Those callers now get the same loud error a plainly unregistered pair always got, which is the review's "today's error" case.
  - `battencli/readopt.go:133` sets `PairMayBeAbsent`, so it takes the new absent path. The `destroy.go:1898` `originTargets` call has no option and discards `absent` and `pairs`, so it only gains the error.
- **F2 (`ArchiveTagName` exported, `ArchiveTagSHALen` kept):** `ArchiveTagName` still slices `tip[:ArchiveTagSHALen]`, which panics on a short tip. The only new external caller, `battencli/readoptstop.go`, keeps the `len(tip) < fabricengine.ArchiveTagSHALen` guard directly above the call. The `destroy.go` swap to `ArchiveTagName(slug, sha)` is identical to the old inline form.
- **F3 and F4 (new rows):** each new row's setup fails the matching guard if the guard is removed.
  - The empty marker and the move record both sit at the prime's records worktree path, which is the fork point the guards read.
  - The detached HEAD hits `HeadRef` and `CurrentSHA`.
  - The `RemoveFollowUp` test uses a temp-dir hub with no repository, so `PrimeName` fails.
  - The lock-error row removes any pre-existing `teardown` directory first, so planting the file works after `sweepFixture`'s `Add`.
  - The gate ran green on tiers 0 and 1.

Further tool calls:
1. `grep -rn "ReconcileScope{" --include=*.go .`: my context lacked the list of run-scope callers, which I needed to check the new error for prunable pairs without `PairMayBeAbsent`.
