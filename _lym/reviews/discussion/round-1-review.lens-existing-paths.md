```yaml
lens: existing-paths
outcome: reported
```

The existing-paths lens traced each record decision against the code already running.

Paths traced, per decision, held or broken:
- D4, the exported `CLAUDE_CONFIG_DIR` against reed's relaunch from the stored command (`reedengine/strand.go:613-621`): holds for a strand left alone, as the record says. It breaks under D7's relaunch; see F2.
- D4/D8, the user-setup and plugins source reading the process's `CLAUDE_CONFIG_DIR` (`claudeengine/doc.go:707`, `scope.go:144`): holds, because D8 makes the lookup take the launch's effective directory.
- D5, the status line in the `--settings` file, Prepare path (`claudeengine.go:234-240`): holds.
- D5, the status line in the `--settings` file, OpenLaunch path (`command.go:474-480`): breaks; see F3.
- D6, `CheckResume` reached from `Runner.checkResume` before launch (`run.go:974-982`): holds, since D6 moves the copy into the check.
- D6, the orch resume's `SessionHeldError` retry, which ends its own holder and resumes again (`orchcli/start.go:807-817`): holds when every known registry is read as one set.
- D7, Wait's held-turn branch, where a usage limit suppresses the hold notice (`wait.go:347`): holds.
- D7, Wait's `retryAPIError` on the same usage-limited turn end (`wait.go:418`, `apierrorretry.go:46,81-82`): breaks; see F1.
- D8, `LocateSessions` callers, the session-end usage recorder (`shedadapters/sessionusage.go:107`) and `tools/tokencount/reads.go:42`, both through `RecordReaders`: holds, since D8 rebuilds those readers with the hub's `Dirs`.
- D8, `SessionHolder` and `Run.SessionHolder` (`resume.go:46`, `run.go:1000`): holds under D8's combining rule.

Findings:

- **F1. BLOCKING, design, D7** (`decision-record.md` D7). The record never orders its switch against shuttle's existing API-error retry on the same turn end, so that retry can end the run before the switch takes effect.
  - Evidence: `retryAPIError` "re-sends and hands on like any other API error" a usage-limited session (`internal/shuttleengine/apierrorretry.go:46`). Each limited turn end counts toward `APIErrorRetries`, and past them the run ends `OutcomeAPIError` and its own-window strand is removed (`apierrorretry.go:81-82`, `99-110`).
  - Wait runs it on every tick after the held-turn branch (`wait.go:418`).
  - D7 relaunches "inside the same run" and promises "no stopped outcome … the run ends once". It does not say whether the retry is skipped for a limited turn end that a switch will handle. It also does not say whether the retry's error count and pending state carry into the relaunched session. A re-send can still type the retry line into the old pane or the new one.
  - D7's "Otherwise the limit is handled as today" leaves both paths live on one turn end.
- **F2. BLOCKING, design, D7** (`decision-record.md` D7). The in-run relaunch is modelled on "the way shuttle's own respawn replaces a strand", but no in-run respawn exists, and reed's stored resume command survives the switch.
  - Shuttle's respawn is a new run that a caller starts after Attach judges a candidate respawn-eligible (`attach.go:189-204`), and "Attach never relaunches or moves a strand" (`attach.go:413`).
  - Reed relaunches a strand matched by name with its stored `ResumeCmd`, and its add path never rewrites `Cmd` or `ResumeCmd` (`reedengine/strand.go:609-621`).
  - The record does not say whether the relaunch is a new strand carrying the new account's command or the same strand under its old stored command. In the second case a later reed resume, such as after a reboot, brings the session back on the out-of-headroom account.
  - D4's "a reed relaunch replays the stored command … until a launch the engine builds starts the strand again" does not settle which of the two this is.
- **F3. MEDIUM, design, D5** (`decision-record.md` D5 and its acceptance criterion "a session whose launched account is listed gets settings carrying the status line"). `OpenLaunch` writes no settings file and passes no `--settings` when the creel flag directory is empty (`claudeengine/command.go:474-480`).
  - A `lym shuttle open` session without creel hooks on a listed account therefore has nowhere to carry D5's `statusLine`.
  - The record must either have `OpenLaunch` write settings whenever the account is listed, or state that such a session gets no status line.

Tool calls:
1. `apierrorretry.go:60-130` and the `retryAPIError`/`usageLimited` call sites. The manifest held only the header, not the retry body or where Wait calls it.
2. `command.go:460-492`, `claudeengine/doc.go:700-715` and the reed `ResumeCmd` sites. The OpenLaunch settings condition and reed's use of the stored command were not in context.
3. `wait.go:320-430`. Needed the order of the held-turn branch against `retryAPIError` in Wait's loop.
4. `reedengine/strand.go:596-625`. Needed reed's relaunch-by-name behaviour, one hop beyond the manifest.
5. The `LocateSessions` callers and `claudeHomeDir` readers. Needed the existing readers D8 changes.
