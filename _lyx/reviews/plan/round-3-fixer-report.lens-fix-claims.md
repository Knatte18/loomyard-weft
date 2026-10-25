```yaml
lens: fix-claims
outcome: reported
```

I checked the claims the fix added or rewrote against the code at HEAD. There are no findings.

- **Card 14 (F1):** both claims hold.
  - `gitexec` has no other way to give a local git child an environment variable. In `runAttempt`, the local path sets `childEnv(nil)` (`internal/gitexec/gitexec.go:274`). `Run`, `RunWithin`, `RunStdin` and `RunGit` take no environment.
  - The snapshot's `commit-tree` (`internal/fabricengine/snapshotuncommitted.go:207`) and the fork's `read-tree` (`internal/fabricengine/forkrecords.go:45`) already run in `fabricengine` through `gitexec.Run`.
  - "`extraEnv` appended after the variables `gitexec` sets" matches how `childEnv` already orders its `extra` argument.
- **Card 8 (F2):** all claims hold.
  - Card 6 creates `anchormarker.go` (`_lyx/plan/06-…:42`).
  - `Bolt.CommitWritten` takes a `write func() ([]string, error)` callback (`internal/fabricengine/bolt.go:49`).
  - `hubworkspace_write.go:82` removes through `root.Remove` and tolerates `fs.ErrNotExist`, which supports "an absent file no error".
  - `root.Remove(` matches none of `destructiveGuardBannedTokens`; the list holds `RemoveAll(` and `os.Remove(` but not a bare `.Remove(`. So no allowlist entry is needed.
- **Card 13 (F5):** card 15's `Close` does remove the record through `RemoveMoveRecord` (`_lyx/plan/15-…:61`), so `Close` belongs in the caller list.
- **Card 17 (F3):** the pass-wait, parent-lock, dobby-end and log-sink-close errors on a fresh run, and the standing record after a sink-reopen error, all match the card's own Intent (the paragraphs beginning "With the record gone" and "With the record standing").
- **Card 17 (F4):** the adopt claims hold.
  - A plain `lyx adopt <slug>` over an owned-elsewhere slug with a local pair and stopped branches gives a way forward naming both `--keep-local` and `--drop-local` (`internal/adoptcli/adopt.go:334`, `:356`). Card 11 adds `anchor_stopped` to `stoppedBranches`.
  - `--keep-local` skips that stop and records the owner.
  - `sharedStopText` runs before every adopt, `--prime` included (`internal/adoptcli/adopt.go:154`). Card 11 has it render a prime-weft `anchor_stopped` with the `--set` as the way forward.
  - One small gap: when the pair has a live driver, adopt names only `--keep-local` (`internal/adoptcli/adopt.go:343`). This does not affect the fix's sentence, because the verb already refuses a held driver lock separately.

Tool calls:
1. Read `destructiveGuardBannedTokens` in `cmd/lyx/destructiveguard_test.go`, plus the lines in `internal/adoptcli/adopt.go` that handle `stoppedBranches`, `sharedStopText`, `KeepLocal` and `req.Prime`. My context had only the start of the allowlist and none of adopt's stop handling.
