```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's purpose.

**Decisions walked**
- D1, machine-layer list `a` (weight 2) and `b` (weight 1), hub pin `b`, with a tracked `claude.yaml` that also holds `accounts`. The load refuses the tracked value and the pin resolves. Purpose met.
- D2, an entry `wt1→b`, then `b` is dropped from the list. The entry reads as no entry and the launch runs as today. Purpose met. Pruning is broken (finding 3).
- D3, weights 2 and 1, six eligible picks. Counts over weights give A,B,A,A,B,A: four and two, never B twice in a row. Purpose met.
- D4, a worktree with entry `b` on `/h/.claude-b`, with the `env` module unset. The account is exported after the env exports and the launched account `b` is recorded. Purpose met.
- D5, a status line run with a listed name and both windows, then a limit hit with a reset. Both windows merge and `blocked_until` is written. Purpose met for `Prepare` launches only (finding 4).
- D6, a shuttle resume whose transcript lies only under `a`, with the worktree now on `b`. `Runner.checkResume`, then `CheckResume`, copies the transcript into `b` and the stat passes. Purpose met. The `OpenLaunch` half does nothing (finding 5).
- D7, a shuttle task session on `a` that hits the limit, with `b` eligible. The entry is set to `b` and an in-run resume follows. Purpose not met: the resume is refused (finding 1). The orch half has no mechanism (finding 2).
- D8, `account set` moves a worktree to `b` while a strand runs on `a`. The fork audit reads the recorded launched directory `a`. Purpose met.
- D9, the prime with no pin on `~/.claude`, listed as `a`, hits a limit. The launched account is `a`, so D7 writes entry `b` for the prime. Purpose met, apart from D7's own break.
- D10, the manual check. Its steps follow from D1 to D7. Purpose met as far as D7 allows.

**Findings**

1. BLOCKING · design · `decision-record.md` D7 (third paragraph). The in-run relaunch has no working mechanism, and its resume would be refused.
   - When the usage limit stops the session, the Claude process stays alive in its pane, registered live in account `a`'s session registry.
   - The relaunch is "a resume of the same session id", so it passes through D6's resume check. That check reads every known directory's registry as one set and refuses a live holder (`internal/shuttleengine/claudeengine/resume.go:87-112`).
   - D7 names no step that ends the old process. The shuttle stop verb, the one sanctioned way to remove a run's strand, records the run as stopped, and D7 explicitly rejects that.
   - The precedent D7 cites, "the way shuttle's own respawn replaces a strand", does not exist. Respawn is a caller starting a new run after Attach judges a candidate respawn-eligible (`internal/shuttleengine/attach.go:182-204`, `:499-501`), and Attach "never relaunches or moves a strand" (`attach.go:413`). No in-run relaunch exists anywhere in `internal/shuttleengine`; a grep for relaunch/respawn found only Attach and doc sites.
   - So the decision's purpose (switch, then go on in the same run) is not reached as written. The record needs to define the in-run strand replacement: how the old strand and its process end without a stopped outcome, and the order of that step against the resume check.

2. MEDIUM · design · `decision-record.md` D7, "The orch watcher … restarts the orch through its resume path". The watcher has nothing it could restart the orch with.
   - The watcher is built with only a session reader, creel, config, paths, stencils, skills and a clock (`internal/orchcli/watch.go:217`, `internal/orchengine/watcher.go:207`). Its `Session` offers usage-limit reads, not session starts (`watcher.go:108-109`). Its limit handling only renews a creel hold and injects a wake line (`internal/orchengine/usagelimit.go:59-82`).
   - The resume path, including `endOwnHolder` for the still-live orch process, lives in `orchcli.resumeOrch` (`internal/orchcli/start.go:796-818`), outside the watcher.
   - The step's input (a way to start or resume a session) is missing, and the record does not say how the watcher gets one.

3. MEDIUM · design · `decision-record.md` D2 "A pick prunes entries whose worktree directory no longer exists in the hub", against D4 and the `PATTERN-cwd-resolution` constraint. The pick has no data to prune with.
   - The pick runs in `claudeengine`, which is told only the worktree's name, its kind and the account file path. It is never told the hub's worktrees or their directories, and it may not derive them (`pattern/PATTERN-cwd-resolution.md`, Told geometry).
   - So it cannot tell which entries' worktree directories are gone, and the prune never runs as stated.
   - The fix is to name what is told for pruning, such as the hub's live worktree names, or move the prune to a caller that holds the geometry.

4. LOW · design · `decision-record.md` D5 "A session whose launched account (D4) is a listed one gets a `statusLine` command in the settings the engine passes with `--settings`". Opened sessions often have no settings file at all.
   - `OpenLaunch` writes a settings file only when a creel flag directory is set. Otherwise `settingsPath` stays `""` and the line carries no `--settings` (`internal/shuttleengine/claudeengine/command.go:474-485`, `:397-398`).
   - A `lym shuttle open` session on a listed account without creel hooks therefore gets no status line and records no figures.
   - D5's bounds mention opened sessions only for `blocked_until`, so this gap is unstated.

5. LOW · consistency · `decision-record.md` D6 "`OpenLaunch` brings the newest copy in the same way when it builds a resume line". The copy has nothing to copy.
   - `OpenLaunch` mints a new session id (`command.go:470-473`), and its resume line exists only for reed's later replay of the stored command (`internal/shuttleengine/open.go:112`).
   - At build time the session has no transcript anywhere. A later replay runs under the same stored account (D4's last paragraph), so no copy is ever needed then either.
   - The sentence should be dropped, or the record should name the open-resume path it means.

**Tool calls**
1. Grepped `internal/orchengine/watcher.go` for start/resume, and read `buildLaunchCmd` and `OpenLaunch` in `command.go`, plus the env-file sites in `claudeengine`. Context lacked whether the watcher can restart a session, the env export order, and when `OpenLaunch` writes settings.
2. Grepped `internal/shuttleengine/open.go` for Resume, and searched for `NewWatcher` callers. Context lacked what the open resume line is for and how the watcher is wired.
