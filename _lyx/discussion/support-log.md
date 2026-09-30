# Support log — trace-log-retention

## Interview

Autonomous session: no operator answered questions; every pick below is recorded in the Question ledger as an auto-pick.

1. Board task read.
   It names two options (config keys for both bounds, or arm the sink only on failure or when a trace id is set) and one hard requirement (never delete a still-running invocation's trace), citing run `operator-surface`, where a Webster step's `trace_file` pointed at a missing file.
2. Codebase check against the requirement.
   `retention.go`'s `Sweep` already skips files whose pid is live, including the sweeping process, and has done so since the sink landed.
   So the literal "swept while live" story needs another cause.
3. Candidate cause found.
   Files are ranked by filename timestamp, which is the sink-arm time.
   A long step arms at its start; lyx children it spawns in tmux agent sessions arm later and likely carry their own trace ids.
   Once the step exits, its file is the oldest of 50+ and the next sweep deletes it — before or just as the driver reads `trace_file`.
   The `operator-surface` worktree is gone, so this was not reproduced.
4. Turnover measured.
   `loomyard/.lyx/logs` held 50 files spanning about 25 minutes, almost all `board upsert`, `board set-deps`, and `fabric add`.
5. Option 2 (suppress routine files) weighed against the logger's level policy, which says an unrecorded mutation or step cannot be repaired from.
   Agents' board/fabric calls inside a run are those mutations, and an inherited trace id does not reliably mark them.
   Rejected; the "fewer files" aim is met by making routine traces stop evicting run traces.
6. Config plumbing.
   `internal/configengine` imports `internal/logger`, so the logger cannot load config itself.
   The sweep moves to process exit, where the CLI layer has loaded config and can pass bounds in, and where no config-load logging can re-enter `sinkMu`.
7. Settled design: trace-grouped count bound, activity (mtime) ranking and ageing, exit-time sweep, `_lyx/config/logger.yaml` with count and days, degrading loader, bad config falls back to defaults with a Warn.

## Rejected alternatives

- **Arm the sink only on failure or with an inherited trace id.**
  Loses Info mutation records the level policy requires for repair, including agents' board/fabric calls inside runs, since tmux-spawned sessions do not reliably inherit `LYX_TRACE_ID`.
  Also leaves a top-level `lyx shed step` (no inherited id) needing a separate opt-in seam.
- **Only make the two bounds configurable, keep file counting and filename ranking.**
  A bigger number delays the Webster-step eviction but does not remove it: a long step with many children still ranks oldest at exit.
- **Environment variables for the bounds (`LYX_TRACE_KEEP` style, matching `LYX_LOG_LEVEL`).**
  Processes sweeping one directory could disagree, and the smallest bound wins; tmux children do not reliably inherit the environment.
- **Keep the sweep at sink arm and inject a lazy config provider.**
  The provider would load config while `sinkMu` is held, and config loading logs, which re-enters the non-reentrant mutex.
- **Separate subdirectories or budgets per command class (runs vs routine).**
  Requires classifying every command and changes the filename/location contract `shed status` and `shuttle` rely on; trace grouping plus activity ranking gets the same outcome without it.
- **Put the keys in an existing file such as `loom.yaml`.**
  Retention governs every lyx process, not loom runs; one file per owning module is the existing pattern.
- **Keep default count at 50.**
  Observed 25-minute turnover in the main worktree; 200 small files cost nothing.

## Review rounds

_No rounds yet._

## Question ledger

- **Q1 — Build config keys, sink-arm suppression, or both?**
  Auto-pick: config keys plus trace-aware, activity-ranked retention; no suppression.
  Rationale: see Interview items 3 and 5.
- **Q2 — How to guarantee a running invocation's trace survives?**
  Auto-pick: keep the live-pid skip, extend it to the whole trace group, and move the sweep to exit.
  Rationale: the live-pid skip already exists; the gap is post-exit eviction by rank, handled by D2.
- **Q3 — Rank by filename timestamp or mtime?**
  Auto-pick: newest mtime per trace group, filename timestamp as stat-failure fallback.
  Rationale: filename time is arm time and misranks a long step; the prior "never mtime" rule has no recorded rationale in the code or tests.
- **Q4 — Config file and key names?**
  Auto-pick: `_lyx/config/logger.yaml`, `trace_retention_count`, `trace_retention_days`.
  Rationale: one file per owning module; integer units match `*_timeout_min` in `loom.yaml`.
- **Q5 — Defaults?**
  Auto-pick: 200 traces, 14 days.
  Rationale: observed turnover at 50; age bound had no complaint.
- **Q6 — Invalid config behaviour?**
  Auto-pick: compiled-in defaults plus one Warn; exit code unchanged.
  Rationale: housekeeping must not fail an unrelated command.
- **Q7 — Which config applies to redirected sinks (reed watchdog, standalone)?**
  Auto-pick: template defaults.
  Rationale: those directories have no owning worktree config; YAGNI on a hub-level file.
- **Q8 — Is the `operator-surface` failure caused by retention?**
  Open.
  Not reproducible (worktree removed); recorded as an open risk in the decision record.
