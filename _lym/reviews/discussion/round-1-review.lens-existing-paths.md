```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the code paths that already write, replay or read the state each Decision changes.

**Paths traced, per Decision**

- **D1, layer load.** The claude engine's `loadConfigOrRefuse` runs at `Prepare` and at `OpenLaunch`. A refusal there fails both launch paths. Survives.
- **D2 and D4, opened sessions.** `lym shuttle open` reaches `shuttleengine` `Runner.Open`, then `Claude.OpenLaunch`. That is a second launch path that builds its own `Cmd` and `ResumeCmd`. **Breaks:** the record is silent about it (finding 2).
- **D2 and D4, reed relaunch.** Reed relaunches by replaying the stored `ResumeCmd` (`reedengine/state.go`), and the line it replays already carries the export. That survives while the record is unchanged. **Breaks** once the record has been replaced or its directory is gone (finding 3).
- **D4, pane launch line.** `paneEnvExports` puts the `env` module exports ahead of the launch command, and the engine's export comes after them. Survives. When the `env` read fails, it logs and exports nothing, and the account export still stands. Survives.
- **D4, usage-limit wait and resume.** The orch watcher waits, then resumes through `ResumeCmd`, which carries the export. Survives.
- **D5, session settings.** `buildSettings` runs from `Prepare`, so the status line reaches lym-run sessions. Opened sessions use `buildOpenSettings`, and they write no settings at all when there is no creel flag directory. Same gap as finding 2.
- **D6, readers.**
  - The resume check (`CheckResume`, by session id and workdir) survives once it is told the recorded directory.
  - The prober's liveness read by id survives under the "search every known directory" rule.
  - The installed-plugins lookup (`installedPlugins` / `claudeConfigDir`) survives: in-pane lym processes inherit the pane's `CLAUDE_CONFIG_DIR`.
  - The hook readers, which take `transcript_path` from the payload, survive.
- **D6, cost readers.** `tools/tokencount` and `lym report` reach transcripts through the engine's worktree-told `Transcripts.LocateSessions(worktree)`. **Breaks** for a run whose worktree has gone (finding 1).
- **D2, existing transcripts under the `env` module directory.** A hub can set `CLAUDE_CONFIG_DIR` through the `env` module today (board, "Built since"). **Breaks:** D2's resume search does not look there (finding 4).
- **Env scrub.** `claudeengine.ScrubbedEnv` strips only `CLAUDECODE` and the `CLAUDE_CODE_` prefix, so `CLAUDE_CONFIG_DIR` is not scrubbed from reed's server spawn. Survives as it is, and the record does not depend on it either way.

**Findings**

1. **MEDIUM, design, D6.** A landed run's cost undercounts in `tools/tokencount` and `lym report`.
   - Both reach transcripts through the engine's worktree-told reader `Transcripts.LocateSessions(worktree)`.
   - D6 sends a worktree-told reader to the worktree's recorded directory, and with no record to go by, to the `env` value, then the process's, then the default.
   - D2 says the record dies with the worktree. For a landed run, the worktree-told reader therefore finds no record and falls back to the default.
   - The result is that every session that ran on a non-default account is missed. D6's own third bullet ("read every known directory") cannot hold for these runs.
   - A related case: an account removed from the list after its runs landed drops out of D6's known directories, so its history disappears from cost reports.
   - Evidence: `tools/tokencount/reads.go:42`, `tools/tokencount/main.go:267`, decision-record.md:80 (D2: the record dies with the worktree), decision-record.md:161–166 (D6).
2. **MEDIUM, design, D4 and D5.** Opened sessions are a second launch path the record does not cover.
   - `lym shuttle open` (`internal/shuttlecli/open.go:80`) reaches `Claude.OpenLaunch` (`claudeengine/command.go:429`). That builds its own `Cmd` and `ResumeCmd`, and its own settings through `buildOpenSettings`, not `Prepare`.
   - D4 places the export in "the launch command and the resume command it builds when it prepares a session". D5 puts the status line in the `--settings` file the engine renders.
   - Neither names the open path, and nothing says whether a fresh open with no record picks.
   - So an opened session in a worktree recorded on account B can run under the default directory. Its transcript then sits where D6's worktree reader never looks, which breaks D2's "Every Claude session of the worktree runs under the recorded directory".
   - Evidence: `command.go:429-487` (the `OpenLaunch` `Cmd` and `ResumeCmd` carry only the TTL export), `settings.go:235`.
3. **MEDIUM, design, D2 and D4.** A reed relaunch bypasses D2's refusal of a resume whose recorded directory has gone.
   - Reed relaunches by replaying the `ResumeCmd` stored in its state (`reedengine/state.go:28`), without calling the engine. D4 treats this as a feature: "a reed relaunch that replays the stored command keeps the directory" (decision-record.md:118).
   - Once the directory has been removed, or the record has been replaced by a fresh re-pick (D2, decision-record.md:68–69), the relaunch still exports the old directory.
   - D2 says such a resume is refused, naming the way forward (decision-record.md:74), but the relaunch never reaches that check.
   - Claude Code then starts on a missing config directory, which is at least an unlogged-in start: an unattended pane stalls. If it also creates the directory, D2's "directory exists" test reads true again.
   - The record is silent on all of this.
4. **MEDIUM, design, D2 and D7.** Pre-change sessions under an `env` module `CLAUDE_CONFIG_DIR` are not found on resume.
   - The board's "Built since" says a hub can set `CLAUDE_CONFIG_DIR` through the `env` module today, so pre-change transcripts may sit there.
   - D2's resume search covers only "the listed accounts' directories and the default directory (D6)". That is narrower than D6's known set, which includes the `env` module's value and the process's.
   - Such a session's transcript is found nowhere, so it is recorded as the default. D4 then unsets `CLAUDE_CONFIG_DIR`, overriding the `env` value, and the resume check refuses.
   - The orch's long-lived session is the case D7 leans on, and it is lost in this way.
   - Evidence: decision-record.md:71–73, 116, 165, 179; board body, "Built since".

**Tool calls**

1. `claudeengine/scrub.go`, the `OpenLaunch` and `buildOpenSettings` callers, a tokencount home-dir grep and the reed relaunch grep. The manifest did not cover the env scrub, the open launch path, tokencount's reader or reed's replay.
2. `command.go` lines 415–540 and the `OpenLaunch` callers, to see the open path's commands and settings, which the manifest lacked.
3. The `OpenSpec` construction sites, tokencount's home-directory reads and the orch start, to find who opens sessions and how tokencount locates transcripts.
4. tokencount's `reads.go` and `main.go` transcript lookup, to confirm it goes through the worktree-told engine reader and covers landed runs.
