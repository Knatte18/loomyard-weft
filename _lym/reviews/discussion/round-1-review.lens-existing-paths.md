```yaml
lens: existing-paths
outcome: reported
```

Existing-paths lens: for each decision, trace the code that already runs and check whether it undoes or misreads the change.

**Paths traced, per decision**

- **D1** — `Runner.launch` calls `engine.Prepare(runDir, …)`, and on error runs `os.RemoveAll(runDir)` (`internal/shuttleengine/run.go:642-646`). A launch that D4 refuses therefore leaves no manifest record. That is fine, because the refusal's error carries the verdict. A relaunch or re-step goes through `launch` again, so it writes a fresh run dir and record. **Survives.**
- **D3** — the assist path reads `editdirective.EngineDirective(..., seat.stencilEngine(geom.DefaultEngine))` (`internal/seatengine/prompt.go:73`), so today it picks the engine from the shuttle default engine, not the launch's engine. D3 together with D4 makes the producer pass the engine its launch resolves to. **Survives.**
- **D3** — the edit-rule guard skips any stencil where `path.Base(name) == "edit-directive"` (`contracts/stencils/parentdirective_test.go:163`). `EngineBlockName` is `path.Join(EngineBlocksDir, engine, block)` (`internal/stencilstore/engineblock.go:25-26`), so both engine copies already match that exemption. Removing the shared stencil leaves the guard as tight as before. **Survives.**
- **D3** — the other producers (`treadleengine`, `sheddriver`, `samplerengine`, `websterengine`, `landingshed`) call `editdirective.Directive(stencilsDir)`. D3 changes that signature to take an engine. **Survives.**
- **D5** — the creel trust sentence moves from the prompt file to `developer_instructions`. Codex treats a hook's block reason as a failed tool call unless its instructions say `[creel]` lines are mail (`internal/shuttleengine/codexengine/doc.go:64`). The sentence is still in the instructions, now at developer level. **Survives.**
- **D5 / D8** — the launch line is written to the strand's launch script and sourced, not typed as one long `send-keys` (`internal/reedengine/doc.go:386-388`), so tmux command limits do not apply. A revive of a Codex strand replays `buildResumeCmd`, which only exits (`codexengine/command.go:115-117`), so plugin hooks never rerun on a revive. **Survives.**
- **D7** — the hub reconcile today renders only the heddle overview (`internal/hubreconcile/walk.go:296`), which matches D7's stated premise. A worktree with a merge in progress is skipped and the build stamp stays stale, so the next run retries it (`walk.go`, the `reconcileWorktree` doc). Heddle's render holds a per-worktree write lock (`internal/heddleengine/render.go:84`), so a reconcile render cannot race a `lym heddle set` render. **Survives.**
- **D12, nudge channel** — Codex reads a `PostToolUse` block as the tool call having failed, and may run the call again (`codexengine/doc.go:110`; `capabilities.go:51-56`). **Breaks.**
- **D12, PreToolUse deny** — guard entries share one `hooks.PreToolUse` setting, and a second `-c` of the same key replaces the first (`codexengine/command.go:234, 244-245`). The deny joins that setting. **Survives**, as long as it goes in through `hookSettings`.

**Findings**

1. **MEDIUM — design — D12 (decision-record.md:162).** The batching nudge on Codex marks a successful call as failed and invites a rerun.
   - D12 sends the nudge line through the hook's `reason`, "as creel's Codex hook does". That is the `{"decision":"block","reason":…}` shape (`codexengine/capabilities.go:51-63`, `doc.go:105`).
   - The existing Codex path reads a block after a tool call as that call having failed, so the session may run it once more (`codexengine/doc.go:110`).
   - For creel mail, the trust sentence keeps the `[creel]` line from reading as a failure (`doc.go:64`). A nudge line carries no such marker.
   - So every nudge would turn a successful call into a "failed" one and invite a repeat call. That works against the nudge's purpose, which is fewer, batched calls.
   - Claude's nudge uses the non-blocking `additionalContext` channel (`claudeengine/batchingnudge.go:297`, `claudeengine/settings.go:88-92`). Codex's `post_tool_use` already took `additionalContext` plus `systemMessage` in a recorded run (`docs/research/creel/creel-midturn-trust-2026-10-28/report.md:199-206`, Codex 0.164.0).
   - The record is also silent on a `PostToolUse` where the creel mail entry and the nudge entry both block in the same setting.
   - Fix: choose the nudge's Codex channel from that evidence and state it, or state why a block is acceptable here.

Tool calls:
1. grep of `internal/shuttleengine` for spec persistence and respawn paths: to check whether a stored spec could reach D4's refusal. It found none.
2. grep of `internal/hubreconcile` for heddle use, and of `stencilstore` for orphan handling: D7's reconcile site and D3's leftover stencil were not in my context.
3. grep of `heddleengine/render.go` for the override file, and of callers of the render API: to find who writes file targets, for D7.
4. Read of `run.go` around Prepare and of `hubreconcile/walk.go` around the heddle render: D1's Prepare failure path and D7's reconcile behaviour were not in my context.
5. grep of the heddle render lock, plus the `launch` span of `run.go`: to check concurrency between a D7 reconcile render and a heddle write.
