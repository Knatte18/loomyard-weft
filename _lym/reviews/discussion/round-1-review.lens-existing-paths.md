```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens over the self-healing-runs decision record. I traced what each decision's new or changed state meets in code that already runs.

**Paths traced, per decision**

- **D1, `hubgeom.Autonomy` → `orchcli.WithRunAutonomy` `%t` stamp** (`internal/orchcli/notify.go:94-98`): survives. D1 changes the stamp to carry the level.
- **D1, board beam modes** (`internal/boardengine/beammodes.go:113,141`): survives. D1 names `off`/`on`/`unattended`.
- **D1, `landingshed` Finalize config-key compare**, which skips `beam.autonomy` (`internal/landingshed/finalize.go:779`): survives. The key is still skipped.
- **D1, `checkTaskSetting`, the resolved check and `CheckLocalValues`** (`internal/dobbyengine/config.go:163-200`): survives. D1 names every path.
- **D1, boolean seams** (`orchengine.State.Autonomy`, drift/loom `ApplyAutonomy`, `AutonomyForcedSetting`, slackcli `orchAutonomous`, pawl `Autonomous`): survives. D1 says on-readers read `true` and `unattended` as on.
- **D3, settle records** (`repairCount`, `lastRestepAt`, `handedOver` keyed by `StopKey` with `HistoryLength`): breaks. See F4.
- **D3, `wakeRestartGap` reset in `drive.go:809`**: survives. D3 states the reset no longer restores the table bound.
- **D4, dobby lift pass** (`triggerFor` sets `liftPending`; `planPass` gives a full pass on `Lift`): breaks. See F2.
- **D4, the orch pause watcher's resume line** (`internal/orchengine/pause.go:1-4`, `pauserecord.ResumeLine`): breaks. See F2.
- **D5, `lym loom resume` refusals** (`internal/loomcli/resume.go:122-170`): survives. The wait sits there.
- **D5, `lym loom start` not-parked path** (`internal/loomcli/start.go:400-416`): breaks for batten's child start. See F1.
- **D5, batten `childSpawnError` retry on `ErrChildNotParked`** (`internal/battencli/wire.go:510-536`): breaks. See F1.
- **D7/D10, the dobby's child read `ChildReading`** (`internal/dobbyengine/runnotice.go:69-85`): breaks. See F3.
- **D7/D8/D12, Slack outbound routing of the scoped dobby program, button drop and `OperatorID`** (`internal/slackengine/outbound.go` `outboundRoute`, `postMessages`; `credentials.go:43`): survives.
- **D8, wait reminders' in-memory `lastReminders` and `OperatorWaitRows` gate** (`internal/dobbyengine/waitreminder.go`): survives. D8 adds a state-record key beside them.
- **D11, `readAgentActivity` held-marker rule** (`internal/battencli/wire.go:700-770`): survives. D11 adds a proof beside it.

**Findings**

- **F1 — MEDIUM, design, decision-record.md D5.**
  Batten's pass runs `lym loom start` as a child process (`internal/battencli/wire.go:1535`) and retries on its not-parked refusal (`wire.go:510-536`). D5 makes `lym loom start` itself wait up to the keep-warm cap. That child has no agent strand, so `KeepWarm` falls back to `DefaultAwaitCap` (`internal/shuttleengines/keepwarm.go:26-40`). So a dobby pass blocks inside the child, which breaks D5's claim that "no dobby or batten tick blocks on it". D5 exempts only "batten's resume through wire.go" and never says how the child `loom start` learns not to wait.
- **F2 — MEDIUM, design, decision-record.md D4 (and the plan-writer note on `queueResumeLine`).**
  The dobby already gives a full pass when a covering pause lifts (`internal/dobbyengine/passes.go:138-160`, `triggerFor` at `:361`). It does so only for runs whose batten state is running (`drivable`, `passes.go:93-95`). D4 adds a dobby resume on lift and is silent on this path, so a run could get two drives or none. It also rests on a premise that does not hold. The orch pause watcher posts `pauserecord.ResumeLine` only to the orch's own interrupted session (`internal/orchengine/pause.go:1-4`, `internal/pauserecord/record.go:30-31`). No line tells the orch to resume runs, and `queueResumeLine` exists nowhere (grep over `internal`).
- **F3 — MEDIUM, design, decision-record.md D7/D10.**
  D7 puts `class:`, `settle:` and `halts_at_row:` in "the run's stop record, the one the dobby's child read carries". But the dobby's child read, `ChildReading` (`internal/dobbyengine/runnotice.go:69-85`), reads only the child status's state, producer, error and history. No such stop record exists. The record does not name which file holds the stamps. It also does not say what clears or rewrites them when the next transition rewrites the status. D10's "reads `settle: orch` and still halted at the same stop" depends on that.
- **F4 — MEDIUM, design, decision-record.md D3.**
  Existing settle records are keyed per stop instance (`StopKey` with `HistoryLength`, `internal/sheddriver/stopkey.go:14-15`). `repairCount` and `lastRestepAt` count only exact-key restep records (`internal/sheddriver/driverules.go:237-257`). The records carry no stop class. D3 needs per-row, per-class counts and a link to the halt entry. It also counts "a tackler's resume", but tackler resumes live in tackler attempt dirs (`internal/sheddriver/tacklerrecords.go` `TacklerAttempt`), not in settle records. The record says neither that `SettleRecord` gains a class and a halt-entry reference, nor which record a tackler resume is read from.

**Tool calls**

1. `landingshed/finalize.go` key skip, dobby `reinterpret.go`, board autonomy display and the stamp format: the manifest lacked the other readers of `beam.autonomy`.
2. `planPass` lift handling and batten's `loom` child commands: needed to check D4's lift path and D5's batten path.
3. `battencli/wire.go:1515-1545`, `passes.go:136-162` and board `settingText`: to confirm the child `lym loom start` spawn and the lift pass.
4. `ChildReading`, `KeepWarm` and `drivable`: to check D7's stop-record premise, D5's cap for a strandless child, and which runs get a lift pass.
