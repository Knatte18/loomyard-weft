```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD and the cited docs.

**[LOW] D5's claim that opened sessions carry the status line**
- **Changed text:** D5: "The settings an opened session writes (D4) carry the same command by the same rule."
- **Problem:** `Claude.OpenLaunch` writes a settings file only when the hub has a board directory. Without one it writes no settings, so the opened session gets no status line, but the sentence states no such condition.
- **Evidence:** `internal/shuttleengine/claudeengine/command.go:470-477`. There, `settingsPath` stays `""` unless `creelFlagDirForHook != ""`. The doc comment at line 415 says: "Without a hub board directory, it writes nothing, creates no dir and marks nothing."

These claims hold:
- **Open path:** `lym shuttle open` reaches `Claude.OpenLaunch` through `SessionOpener`, called at `internal/shuttleengine/open.go:80`. That launch has both a `Cmd` and a `ResumeCmd` line (`command.go:483-486`).
- **Reed relaunch:** a relaunch replays the strand's stored `ResumeCmd` (`internal/reedengine/lifecycle.go:874`) and reads no record.
- **Windows credentials path:** the Authentication docs place it at `%USERPROFILE%\.claude\.credentials.json`, under `CLAUDE_CONFIG_DIR` when that is set.
- **Seven-day bound:** a forged input lowers eligibility for at most seven days. This follows from the new rule that drops a window whose `resets_at` is more than one window length past now.

Tool calls:
- I grepped `internal/shuttleengine` and `internal/reedengine` for the `SessionOpener` path and for the command reed stores for a relaunch. My context did not show who calls `OpenLaunch` or which field a relaunch replays.
