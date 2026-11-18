```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end on one concrete case, using the data each step actually receives.

- D1: a machine-layer list, plus a tracked `claude.yaml` setting `account_pins.hub`. Load refuses and the launch fails. Purpose met.
- D2: a pick runs while worktree `x` is made concurrently. The names reader is called under the lock, so `x` keeps its entry. Purpose met.
- D3: weights 2 and 1, six picks. The sequence is A, B, A, A, B, A: four and two, never B twice in a row. Purpose met.
- D4: an entry on a non-default directory. reed chains `paneEnvExports` ahead of `launchCmd` (`internal/reedengine/spawn.go:125`), so the engine's export comes after the `env` value and wins. Purpose met.
- D5: a session on a listed account runs the status line verb from the worktree, which merges into the usage file. Purpose met. When a limit hit's `blocked_until` is written, relative to D7's resolution, is not ordered (finding 4).
- D6: a shuttle resume copies the transcript, and the first turn ends on an API error. The check is supposed to record that it copied "beside the run directory's launched-account record", but no run directory exists yet when the check runs. Purpose not met (finding 2).
- D7 shuttle: a run whose spec names skills hits a limit, and the switch writes a new entry. Prepare builds a resume launch whose first turn is the skill-load message or the run's whole prompt. The record says nothing of who delivers `PromptLine` or runs startup. Purpose not met (finding 1).
- D7 orch: the watcher's restart goes through `resumeOrch`. That ends the old process and starts a new strand, but leaves the old strand in reed and the orch state bound to the old GUID. Purpose not met (finding 3).
- D7 beams pin: the pinned account hits a limit. Whether the relaunch resolves to the entry depends on `blocked_until` being written first, which the record does not order. Purpose not reliably met (finding 4).
- D8: `LocateSessions` across the known directories, and the live readers on the launched directory. Purpose met.
- D9: the prime under a beams pin; orch resume goes through `StartSession`, `checkResume` and `Prepare`, which resolves the pin. Purpose met.
- D10: the manual wiring steps. Purpose met.

Findings:

1. **BLOCKING, design, `_lym/discussion/decision-record.md:199-208`.** D7's relaunch never says what turn the resumed session gets, or that startup and the deferred prompt line run again.
   - Step 2's "fresh resume launch" through `Prepare` puts the run's whole `prompt.md` pointer on the launch line with `--resume`. For a spec that names skills, the first turn is the skill-load message instead, and the pointer becomes `Launch.PromptLine`.
   - Only `launch` sends that line after `awaitStartup` and persists it (`RunState.PromptLine`). Step 4 says only that the strand starts and its GUID is written.
   - So a skills-loading run, which is most roles, loads its skills and ends a turn held with no task. A run with no skills gets its whole task prompt again, mid-session. Neither continues the limited turn.
   - Evidence: `internal/shuttleengine/claudeengine/claudeengine.go:281-285` and `:301-305`; `internal/shuttleengine/run.go:705-728` and `:637`; `internal/shuttleengine/wait.go:286` (`started` is seeded true, so no startup probe runs for the new strand).

2. **MEDIUM, design, `_lym/discussion/decision-record.md:170-174`.** D6's "the check records a copy it made beside the run directory's launched-account record" has no run directory to write to.
   - On every ordinary resume, `checkResume` runs before `launch` creates the run directory, and `CheckResume(sessionID, workdir)` is given no run directory.
   - So the refused-resume fallback can never learn that a copy happened. Only the D7 relaunch path, whose directory exists, could record it.
   - Evidence: `internal/shuttleengine/run.go:589` against `:645`; `internal/shuttleengine/engine.go:357`.

3. **MEDIUM, design, `_lym/discussion/decision-record.md:217-221`.** This carries prior finding F6. A restart "through `resumeOrch`" does only part of the orch restart.
   - `resumeOrch` ends the live holder only through `endOrch`, which ends processes. It then calls `StartSession`.
   - The start path does the rest around `launchOrch`: removing the old strand, `ResetForFreshLaunch`, `saveFreshLaunch` with the new GUID, `registerOrchProcess`, and spawning a watcher.
   - So after a switch the old pane stays in reed, the new orch's process is unregistered, and the watcher keeps reading the old strand's events and creel name.
   - Evidence: `internal/orchcli/start.go:705-740` and `:801-818`; `internal/orchcli/cli.go:134-135`.

4. **MEDIUM, design, `_lym/discussion/decision-record.md:145-147` and `:186-199`.** Nothing orders D5's `blocked_until` write before D7's switch and the relaunch's D4 resolution.
   - Case: a beams-pinned worktree whose `blocked_until` is not yet written. D4 resolves it back to the pin, which is the launched account. "When the worktree then resolves to another account" is false, so the entry is written but no relaunch happens.
   - On the second bullet's check, a sibling's earlier switch also reads the pin as live.
   - The record should state that the limit hit writes `blocked_until` under the account file's lock before the switch decides and before `Prepare` resolves.
   - Evidence: `internal/shuttleengine/wait.go:347` (the usage-limit read sits in Wait's held-turn switch).

Tool calls:
- One grep in `internal/orchcli` for `endOrch`, `spawnWatcher` and `ResetForFreshLaunch`. The manifest showed the call to `endOrch` but not what it does, and not where the orch's strand and state are rebound.
