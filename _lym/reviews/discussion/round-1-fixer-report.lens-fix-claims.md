```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checking that every claim the fix added or rewrote holds against the code at HEAD.

no findings

These claims hold against the code at HEAD:
- **The per-user lock's home (D10).** `standalonestate.UserStateDir` exists (`internal/standalonestate/standalonestate.go:132`). The machine record of hubs sits there (`internal/machinecli/hubs.go`), and so does the desktop launcher's `launchers.lock` (`internal/desktoplauncher/desktoplauncher.go`).
- **The blocking acquire and the 10 ms poll (D10 and the support log).**
  - `lock.AcquireWriteLock` blocks on `fl.Lock()`.
  - `AcquireWriteLockWithin` polls at `boundedWaitRetryDelay = 10ms` (`internal/lock/lock.go:17,62-78`).
  - The Slack holder takes its lock through `lock.AcquireWriteLock` (`internal/slackengine/holder.go:176`), which matches the PATTERN line "its lock wait blocking without waking".
- **`lym machine start` runs no `window_command` today (D12).** No file under `internal/machinecli/` names `desktophook` or `window_command`.
- **The order of `lym machine start` (D12).** `runBeamStarts` runs before `openWindows` (`internal/machinecli/start.go:253-254`). `openWindows` opens only worktrees whose outcome is `startStarted` (`start.go:353-357`).
- **`UpdateJSON` serializes only writers that all go through it (D9).** This matches `internal/state/doc.go:11-15`.
- **SIGTERM, then SIGKILL after 5 s (D3).** This matches `endGrace = 5 * time.Second` (`internal/runproc/end.go:21`).
- **`clientArea` with the placement area (D6).** This matches `arrange-to-tiles.sh`'s `workspace.clientArea(0, screen, desktop)`.
- **Konsole's single-process setting (D2).** Neve's `konsolerc` holds no single-process key.

Not checkable against the code: the KWin API facts (`sendClientToScreen`, writing `frameGeometry`, a script's body running whole once `run` is called) and the `kdotool` pid and caption observation. Constraints now lists the KWin facts as unobserved, and the `kdotool` result was the reviewer's own run, not this fix's.

Tool calls:
- One `grep` over `internal/slackengine/holder.go`, `internal/machinecli/`, `internal/standalonestate/` and `internal/runproc/`: none of these files were in my context, and each claim above names one of them.
