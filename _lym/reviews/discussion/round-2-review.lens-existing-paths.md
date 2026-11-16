```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths — trace every existing code path that writes, resets or reads state the record's decisions introduce or change.

**Paths traced, per decision**
- D1, manifest record in the run directory, against shuttle's done cleanup (`wait.go:2033-2041` removes the run dir on `OutcomeDone` without `KeepPane`): **breaks**.
- D1, manifest record, against the Prepare-failure cleanup (`run.go:644`): survives. The record is not needed once the launch is refused.
- D1/D11, the bench report that reads "the run's own manifest record" after the run: **breaks**. The record is gone once the run is done (see above).
- D4, prompt engine "set from the same model choice", against `ModelChoice.Engine` being empty for the default engine and the runner resolving it alone (`modelchoice.go:10`, `run.go:561`, `engineset.go:65-73`): **breaks**.
- D4, against the runner's carry-over that appends operator instructions to `spec.Prompt` after the producer renders it (`run.go` `noticeCarriedInstructions`, `carryover.go:173`): survives. The text is engine-neutral.
- D5, creel trust sentence moving out of `prompt.md`, against the creel `PostToolUse`/Stop block delivery that needs it in the session's instructions (`codexengine/doc.go:64,105-110`): survives. It is still in the session's instructions, at developer level.
- D5, against `OpenLaunch` (no prompt file, no creel hooks): survives. It is untouched.
- D5/D8/D9, developer instructions, against the Codex resume line (`buildResumeCmd` exits and starts no Codex): survives. Nothing is replayed.
- D9, `AGENTS.override.md` carrying `AGENTS.md` and `CLAUDE.md` with imports expanded, against heddle's `codex-file` re-render trigger (`heddleengine/target.go:65`, `seam.go:56`): **breaks** for an imported file.
- D7, the reconcile render of file targets, against the worktree-event render through `ReRender` (`seam.go:56`): survives. Both are renders of the same table.
- D12, the nudge entry joining the one `hooks.PostToolUse` setting (`command.go:247-251`, "a second `-c` of the same key replaces the first"): survives. The record says it joins.
- D12, the nudge state's per-chain count keyed by `agent_id` (`batchingnudge.go:154,159-179,195,208-220`), against Codex payloads that carry no `agent_id` and give a child the parent's `session_id`: **breaks / unstated**.
- D12, against the Codex revive path: survives. No session restarts, so the state file is not reseeded.

**Findings**

1. **MEDIUM** · design · D1 and D11 (`decision-record.md` D1 "`Prepare` writes the manifest into the run directory as a record"; D11 "Its report opens with the run's own manifest record").
   - Summary: shuttle deletes the run directory, and the manifest record with it, when a run ends done without `KeepPane`. A real-producer bench's run therefore leaves no record to open its report with.
   - The record is also lost for checking "an omitted item counts as missing in the record's verdict" after a done run.
   - The record is silent on how the record survives cleanup.
   - Evidence: `internal/shuttleengine/wait.go:2033` (`cleaned := outcome == OutcomeDone && !run.spec.KeepPane`) and `:2041` (`os.RemoveAll(run.runDir)`). Producers' `shuttleengine.Spec{` literals set no `KeepPane`; for example, `internal/shuttlecli/run.go:155-171` sets it only from a flag.

2. **MEDIUM** · design · D4 (`decision-record.md` D4 "set by the producer from the same model choice it launches with … refuses a spec whose prompt engine … is unset").
   - Summary: an empty `ModelChoice.Engine` means "the engine set's default", and only the runner resolves that name.
   - A producer that copies the engine from its model choice gets an empty prompt engine for every default-engine choice, and D4 refuses that as unset.
   - The record does not say how a producer learns the default engine name it must render for.
   - Evidence: `internal/shuttleengine/modelchoice.go:10` ("empty for the engine set's default"); `internal/shuttleengine/engineset.go:65-73` (`engineFor` fills `DefaultName()` for an empty name); `internal/shuttleengine/run.go:561`.

3. **LOW** · design · D9 (`decision-record.md` D9 "`AGENTS.override.md` carries … `CLAUDE.md` … with the CLAUDE.md imports expanded") and D8 (heddle's copy).
   - Summary: heddle re-renders `codex-file` only when `AGENTS.md`, `CLAUDE.md` or `AGENTS.override.md` changes, so an edit to a file `CLAUDE.md` imports through `@` leaves the override's expanded copy stale.
   - Claude reads the import fresh, so the two engines diverge, and the record is silent on it.
   - Evidence: `internal/heddleengine/target.go:65` (`ReRender: []string{"AGENTS.md", "CLAUDE.md", "AGENTS.override.md"}`); `internal/heddleengine/seam.go:56`.

4. **LOW** · design · D12 (`decision-record.md` D12 "judges the calling chain's recent calls from the Codex rollout").
   - Summary: the existing nudge state keeps one count per subagent, keyed by the payload's `agent_id`, and `max_nudges` binds "each of its subagents and forks".
   - A Codex hook payload carries no `agent_id`, and a child's payload carries the parent's `session_id`; only `transcript_path` tells the chains apart.
   - D12 does not name the key a Codex chain's count is held under. If the existing state shape is reused, every child shares the main session's count and cap, so a subagent's nudges use up the parent's.
   - Evidence: `internal/shuttleengine/claudeengine/batchingnudge.go:154-164,195,208-220`; `docs/research/creel/creel-hook-channels-2026-10-28/report.md:414` (payload keys); `docs/research/codex/codex-twill-spike-2026-10-24/report.md:199-200`.

**Tool calls**
- Searched `internal/shuttleengine` for `RemoveAll` of the run dir, the `Prepare(` callers and engine fallback: my context lacked the run-dir lifecycle that D1's record lives in.
- Searched for `AGENTS.override.md` and `project_doc` writers, the edit-directive guard and heddle render sites: my context lacked who owns the override file D9 changes.
- Read the `heddleengine/target.go` target table and the `ReRender` users: needed the override's re-render triggers.
- Read `run.go:600-660`, `wait.go:2025-2050` and the engine-selection greps: needed the Prepare call site, the cleanup path and where the engine is resolved.
- Read the `Engine` fields in `spec.go`/`modelchoice.go`, `engineset.go:55-90` and the `carryover.go` functions: needed how an empty engine resolves and what the runner appends to the prompt after the producer.
