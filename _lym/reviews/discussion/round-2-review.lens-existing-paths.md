```yaml
lens: existing-paths
outcome: reported
```

Existing-paths lens: tracing which code paths that already run write, reset or read the state the record's decisions change, and whether each decision's result survives them.

**Paths traced**

- **D7, step 1** (stop verb relaunch mode, strand removed, record marked relaunching):
  - Wait / attach probe: survives, because the record states the PATTERN change.
  - Opportunistic orphan sweep in another `Start` in the same worktree: breaks (F-A).
  - reed's resume of non-live strands: survives, since the strand is removed.
  - Driver crash or restart between steps 1 and 4: breaks. Nothing restarts the relaunch, and the record stays relaunching (F-C).
- **D7, step 4** (new strand GUID written into the run record): breaks for every caller that persisted the old GUID (F-B):
  - sheddriver tackler marker: attach and `StopStrand`
  - webster's recover-batch and run-level records: reclaim `StopStrand`
  - loomshed sampler: `StopStrand`
  - beam dobby tidy: `StopStrand`
  - `FindRun` by GUID
- **D7, step 4**, the new strand's startup gates against the persisted `Started` fact: breaks. The startup probe is skipped (F-D).
- **D7**, `retryAPIError` cursor and count reset: survives as stated.
- **D7**, session-end usage record: survives. It is one run and one session id, and D8 reads the newest copy.
- **D6**, fresh-start fallback through D7's relaunch: it inherits F-A, F-B and F-D, since it reuses the same relaunch.
- **D4**, env module exports ahead of the launch command (`internal/reedengine/spawn.go:125`, `sh.Chain(paneEnvExports, launchCmd)`): survives. The engine's export comes after them.
- **D4**, reed relaunch, which replays the stored command and re-reads PaneEnv: survives as the record states.
- **D4**, `ScrubbedEnv` gaining `CLAUDE_CONFIG_DIR`: survives.
- **D5**, settings passed with `--settings` (Prepare and OpenLaunch): survives. Nothing else writes `statusLine`.
- **D8**, `CheckResume` / `SessionHolder` / `LocateSessions` / `projectsRoot` / `sessionRegistryDir`, all hard-coded to `~/.claude` today: they are covered by D8's changes. Survives.
- **D2 / D3**, pick prune under a told names reader: survives.

**Findings**

**F-A — BLOCKING, design, `_lym/discussion/decision-record.md:202`**
- Summary: during D7's relaunch window, any other `Start` in the worktree deletes the relaunching run's directory. Its strand is gone from reed, and the orphan sweep checks only the strand GUID, never the run's outcome. That directory holds `run.json`, `events.jsonl`, `settings.json` and the launched-account record. The record says only attach probes and `Wait` read a relaunching record as live.
- Evidence:
  - `internal/shuttleengine/rundir.go:355-363`: `sweepOrphans` removes any run dir older than minAge (2× startup timeout) whose `rs.StrandGUID` is not in reed's strands. A limited run is old enough.
  - `internal/shuttleengine/run.go:604`: `sweepOrphansOpportunistic` runs on every start.
- Fix: in D7 step 1, state that the orphan sweep keeps a relaunching record's directory. Add that to the PATTERN change text and to D7's attach-probe acceptance criterion.

**F-B — BLOCKING, design, `_lym/discussion/decision-record.md:207`**
- Summary: D7 step 4 writes a new strand GUID into the run record. Every caller that persisted the old GUID then stops, finds or reattaches by a GUID no record names. `StopStrand(old)` records nothing and removes nothing, so the relaunched agent keeps running unowned. A driver restart's reclaim then launches a second agent beside it, which is the duplicate-agent hazard.
- Evidence:
  - `internal/sheddriver/tackler.go:166-171`: `StopStrand(marker.StrandGUID)` and attach use the marker's persisted GUID.
  - `internal/websterengine/recoverbatch.go:295-296`, `internal/websterengine/runlevel.go:305-331`, `internal/loomshed/samplersession.go:235-255` and `internal/beamcli/dobbydeps.go:861-866` all stop by a stored GUID.
  - `internal/shuttleengine/stop.go:76-79`: no record found means only that GUID's strand is removed.
  - `internal/shuttleengine/rundir.go:288`: `FindRun` matches `rs.StrandGUID == guid`.
- Fix: in D7, either keep the run's strand GUID across the relaunch (reed relaunches under the same GUID with a new stored command), or state how `FindRun`, `StopStrand` and attach by GUID resolve a superseded GUID to the run. Add an acceptance criterion: `StopStrand` of the pre-relaunch GUID stops the relaunched strand and records the stop.

**F-C — MEDIUM, design, `_lym/discussion/decision-record.md:203`**
- Summary: the driver process that owns `Wait` runs the relaunch. A crash between step 1 and step 4 leaves a relaunching record with no strand, and no path ever starts one. Because attach and `Wait` read relaunching as live, the restarted driver's attach waits on a strand that never comes, until the run's deadline. The record names no restart path for a relaunch cut short.
- Evidence: `internal/shuttleengine/stop.go:43-56`. The relaunch mode, unlike `Stop`, leaves the record non-terminal. The Wait/attach liveness check is at `internal/shuttleengine/wait.go:258-266`.
- Fix: in D7, state what an attach does with a relaunching record whose strand is absent past a bound: finish the relaunch, or end the run as a strand that died. Add an acceptance criterion for an attach after a driver killed mid-relaunch.

**F-D — MEDIUM, design, `_lym/discussion/decision-record.md:207`**
- Summary: the relaunched strand starts in a run whose record already holds `Started: true`. `Wait`'s startup probe (trust dialog, readiness) runs only for runs never marked Started, so nothing awaits the new pane's startup gates. The record does not say that step 4 runs the start path's `awaitStartup` or resets `Started`.
- Evidence: `internal/shuttleengine/wait.go:277-286` ("Only a persisted Started fact skips the probe"), `internal/shuttleengine/wait.go:1566`, and `internal/shuttleengine/wait.go:727` (`awaitStartup`).
- Fix: in D7 step 4, state that the new strand passes the start path's startup gates (`awaitStartup`) before the GUID is written. Add that to D7's first acceptance criterion.

**Tool calls**

1. `internal/shuttleengine/run.go` 1060–1140 and the attach/`FindRun` signatures: the orphan sweep body and the GUID entry points were not in context.
2. The `sweepOrphans` body, and callers outside shuttleengine that stop or find by a persisted GUID: needed to test whether a GUID change and a missing strand survive them.
3. The sheddriver tackler reclaim, webster's recover-batch stop, and the startup-marking sites: needed to confirm the persisted-GUID reuse and the `Started` gate.
