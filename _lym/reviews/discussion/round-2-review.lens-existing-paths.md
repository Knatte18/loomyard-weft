```yaml
lens: existing-paths
outcome: reported
```

The existing-paths lens traced each decision's new state against the code that already writes, reads or resumes it.

**Paths traced, per decision**

- **D3 (scope from config):**
  - **Survives:** run_settings carrying `scope: partial` plus `skip`. `PrepareRunSettings` gives the retired `skip` to `ValidateLocal`, which admits a CategoryNone retired key, and the load drops it.
  - **Survives:** the `RunCapture` wiring sets `request.Skip = scope.Skip` for an old `partial` run. Once `{skip_file}` is refused, the current capture writes every case, and D3 leaves the skipped cases out of the diff.
  - **Survives:** the runprofile `docs` darn block, carried by the seed-only template follow.
- **D15 (reinterpretation of `partial` with its `skip` list): breaks.** The hub-wide walk runs `ReconcileHubWideAt` before `applyReinterpretations`, and the sparse reconcile prunes retired keys first.
- **D15 (renames):** **survives.** `Migrate` runs before the reinterpretation, in the walk and in `PrepareRunSettings`.
- **D6 (expect re-entry reads `AgentSessionOpen`): breaks.** The mark lives in `runs.json`, which D14 removes. `driftcli` verb admission reads the same mark.
- **D4 and D14 (an old Drift-Verify agent live when this lands): breaks.** Today's `pass` probes the old agent and resumes it. The new engine has neither the mark nor the agent, and the record says nothing about it.
- **D4 and D14 (old Drift-Baseline agent live mid-scope): breaks.** Same gap: the agent's `lym drift scope` verb is removed and the row records scope from config.
- **D12 (old run with no counted capture, or a waived run): breaks.** Today's halt names `lym drift waive`, which is removed. D12 admits `defer` only when a counted capture's directory is missing.
- **D13 (Verdict callers `landingdeps` and the loom wiring):** **survives.** They read the records without a capture.
- **D6 (run position read from `Deps.Status` or the `driftcli` status read):** **survives.**
- **D4 (row name `Drift-Recheck` kept; the tapestry comment's in-flight Recheck-back route):** **survives** as D4 states it.

**Findings**

1. **MEDIUM, consistency.** D6's re-entry rule against D14's removals.
   - D6 tells an interrupted expect session from a finished one with `AgentSessionOpen`. That mark is `runsRecord.SessionOpen`, kept in `runs.json`, which D14 removes as "the run budget and `runs.json`".
   - It is one record per run, written only by Drift-Verify's `ResetRuns`, and `driftcli`'s `verbStatus` reads it to admit verbs.
   - The record names neither a new home for the mark nor a separate mark per seat for the expect agent and the judge.
   - Evidence: `internal/driftengine/records.go:144-151`, `:247-281`; `internal/driftcli/wiring.go:76`; `internal/driftengine/verify.go:125-138`.
2. **MEDIUM, design.** D4 and D14 leave an old agent that is live when this lands.
   - Today Drift-Verify's `pass` finds an open mark, probes the chair and resumes the live verify agent (`runAgent`). Drift-Baseline does the same for its scope agent.
   - Under D4 both rows are code-only, and D14 removes the mark and the agents' verbs. A run parked with a live old Drift-Verify or Drift-Baseline agent keeps that pane running the old stencil, which edits code and calls removed verbs, while the new row runs beside it.
   - The record says neither that the old chair is ended nor how.
   - Evidence: `internal/driftengine/verify.go:125-138`, `:235-255`; `internal/driftengine/baseline.go:113`.
3. **MEDIUM, design.** D12 has no way forward for a run whose capture never counted, or one waived under the old Drift.
   - Today Drift-Verify halts "no baseline capture counts … way forward: lym drift waive". The darn recipe comment settles a run without `awaited_at` with `lym drift waive`, and a run with `WaivedAt` set is Done.
   - D12 removes `waive` and admits `defer` only where a counted capture's baseline directory is missing. A run with an uncounted capture, or with a waiver recorded before this lands, has no stated outcome or way forward.
   - Evidence: `internal/driftengine/verify.go:111-113`, `:170-171`; `contracts/recipes/darn-recipe.yaml:53-54`; `internal/driftengine/records.go:205-209`.
4. **LOW, design.** D15's flag for a `partial` base cannot name its `skip` list.
   - The hub-wide walk runs `configsync.ReconcileHubWideAt` before `applyReinterpretations`, and the sparse reconcile prunes a module's retired keys on every run.
   - Once D15 declares `skip` retired, it is gone from the base before the reinterpretation reads it. The flag "with its skip list" sees no list, and the operator's skipped cases are lost from the notice.
   - Evidence: `internal/hubreconcile/walk.go:217-239`; `internal/configsync/configsync.go:459`, `:620`, `:685-692`.

**Tool calls**

1. `internal/driftcli/wiring.go:55-110` and `internal/hubreconcile/walk.go:204-275`: verb admission's read of the session mark, and the order of reconcile against reinterpretation. Neither was in context.
2. `internal/driftengine/verify.go:105-300`: Drift-Verify's resume, live-agent and waiver paths. Only function names were in context.
3. A grep for drift record and config readers outside `driftengine`, to find every reader of `scope.json` and the config.
4. `internal/loomcli/driftwiring.go:140-210` and `internal/driftcli/cli.go:595-660`: the capture's use of the skip list, and the status verb's use of the run count.
5. A grep of `configsync` for retired-key pruning and of the runprofile template: whether reconcile prunes `skip` before the reinterpretation, and the profile's drift block.
