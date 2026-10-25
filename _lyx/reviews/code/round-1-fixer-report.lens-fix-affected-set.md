```yaml
lens: fix-affected-set
outcome: reported
```

No findings. Two minor leftovers sit outside the review's named sites; I rate both LOW or below.

Task: rerun each set-premise finding's search to check the fix covers every member, not only the sites the review named.

**Checked**
- **F2** (every acquirer of a probed lock): the two named sites changed, and the driver-lock caller `battencli/cancelcmd.go:485` already waits with `AcquireWriteLockWithin`. The other `TryAcquireWriteLock` hits take the run lock, the land lock or a different lock, which the pane never probes.
- **F7** (stale prime-only wording): no `prime's runs strand` text remains, and `battencli/cli.go:20` and `overview.go` already say prime and each beam.
- **F1** (callers of `NewDobbyViewReader`): `battencli/overview.go:559`, now guarded by `worktreeHasDobby`, and `beamcli/cli.go:117`.
- **F3–F6**: not set-shaped, so I left them unchecked.

**Leftovers**
- **LOW (comments left from F2):** these still describe the old behaviour.
  - `internal/loomcli/doc.go:178` says the loom driver "takes the run's driver lock without blocking … a second driver refuses at once". That driver goes through `sheddriver.DriveVerb` (`loomcli/drivewiring.go:75`), which now waits `lock.ProbeGrace` before refusing.
  - The `TestDriveVerb` comment at `sheddriver/drive_test.go:825` says "refuses at once".
- **No finding (F1):** `beamcli` status reads a view for worktrees that `beam status` iterates itself, which I take to be beams and the prime. The loop's source wasn't in my context, so I didn't check it.

**Tool calls**
1. `grep` for stale wording and `NewDobbyViewReader` callers (F7 and F1 sets), because my context had no repo-wide search.
2. `sed` on `beamcli/cli.go`, a `DriveVerb` caller grep and `loomcli/doc.go:176-179`, to see the F1 sibling caller and the other `DriveVerb` users.
