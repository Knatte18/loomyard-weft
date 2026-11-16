```yaml
lens: bound-breaker
outcome: reported
```

Restating the task: bound-breaker lens. I attacked the record's bounds, guards and claims about existing code. Four claims break.

**Claims attacked:**
- D12: the question deny's `*` matcher with a payload name ending in `request_user_input` is the counterpart of Claude's AskUserQuestion deny. **Broken** (F1).
- D12: the nudge judges Codex shell calls, sliced shell reads and `apply_patch` calls, and responses made only of `apply_patch` calls, from the Codex rollout. **Broken** (F2).
- D12: the nudge as `additionalContext` plus `systemMessage` does not mark the call failed. Held: case 5 in creel-hook-channels shows the tool result, then a `↳ Hook` line, with no failure.
- D1: a scan fails on a production launch-spec literal in a package with no producer-table entry, so every producer is covered. **Broken** (F3).
- D4: only `lym shuttle run` marks a spec hand-written, and a scan holds that. Held: no production Go code runs `lym shuttle run` as a subprocess, and no fallback path swaps a spec's engine. The only `ModelChoice` reassignment is `shuttlecli/open.go:87`, which is `lym shuttle open` and out of scope.
- D4: the orch is pinned to Claude under a Codex default engine. Held as stated.
- D5: the developer instructions ride one argument, capped at 128 KiB by `MAX_ARG_STRLEN`. Held: reed types a `source` of a launch script (`reedengine/panebin.go:317`), so the per-argument exec limit is the binding one, not tmux's message size.
- D10: the verb writes nothing outside its temporary run directory except the Codex home. **Broken** in a small way (F4).
- D10: the verb runs no sandbox probe or plugin hook. Held, provided the reporting probe and the unrun hook runner replace `launchSandboxMode` (`codexengine.go:118`) and D8's runner, as D10 says.
- D3: the bound on a hub-edited seeded directive. Held: `stencilstore` keeps edited copies.
- D7: the bound that an older lym refuses a store holding `agents-prompt`. Held as stated.
- D2: the allowlist bound is held by review. Held as stated.
- D6: 16 threads sits above the widest fan. Held: six-lens forks are the widest named.
- D9: the cap is set above the path's instruction-file total. Held as stated.

**Findings:**

F1 — MEDIUM — design — `decision-record.md` D12, first paragraph and its acceptance criterion
- Summary: the suffix match lets `request_user_input_async` through.
- Counterexample: the Codex 0.164.0 tool tree recorded in a lym run's rollout lists two question tools side by side in the `functions` namespace, `request_user_input` and `request_user_input_async`. The async one is described as "Ask the user one or more questions during ongoing work". A name ending in `request_user_input_async` does not end in `request_user_input`, so an autonomous Codex session can ask the operator through the async tool and the deny never fires.
- The record also leaves the payload name unrecorded ("No run has recorded the question tool's payload name"). The tool tree above is evidence the writer could have recorded.
- Evidence: `~/.lym/codex/sessions/2026/11/05/rollout-2026-11-05T10-11-18-01a1aafa-15e4-7721-9a74-5f03119fc17b.jsonl` line 3 (`additional_tools`); `decision-record.md:185-186, 205, 256`.

F2 — MEDIUM — design — `decision-record.md` D12, the nudge bullets and the last acceptance criterion
- Summary: the record's Codex call model does not match what a Codex 0.164.0 rollout records.
- In that rollout, every model tool call is a `custom_tool_call` named `exec`: a JavaScript cell ("Run JavaScript code to orchestrate/compose tool calls"). Inside it, `tools.exec_command({cmd:…})` and `tools.apply_patch(patch)` run, and one cell can hold several of them. The commands and patches show up only as `item_completed` `CommandExecution` and `FileChange` items.
- So no rollout record is "a shell call" or "an `apply_patch` call" at the level of a response. "A response holding only `apply_patch` calls" has no recorded unit to count.
- Nothing records `PostToolUse` firing for `apply_patch`, or its `tool_name` there. The only recorded Codex tool names in hook payloads are `Bash` and `collaborationspawn_agent`.
- The "read-again rule" also needs an earlier whole-file shell read, and the record never says what counts as one.
- These are facts the design rests on, left for the implementer to find out.
- Evidence: same rollout, `custom_tool_call` name `exec` ×30 and items `CommandExecution` ×46, `FileChange` ×3; `exec` input `const r = await tools.apply_patch(patch)`; `codex-spike-2026-10-24/report.md:210, 251`; `stencil-engine-audit-2026-10-24/report.md:361`; `decision-record.md:188-196, 257-258`.

F3 — MEDIUM — design — `decision-record.md` D1, second paragraph, and acceptance criterion 1
- Summary: the producer scan works per package, so a second producer in a package that already has an entry is never compared.
- Counterexample: `internal/shedadapters/bouncer.go` builds four spec literals (lines 909, 937, 1728, 1882) and `internal/websterengine` builds three (`planfixer.go:158`, `runlevel.go:752`, `recoverbatch.go:328`).
- One table entry per package satisfies "a package the table has no entry for". A new or existing literal in the same package then passes the scan without its prompt being rendered or compared. That breaks the Goal's "a gap is a bug, and a test fails on it".
- Evidence: the `shuttleengine.Spec{` grep results; `decision-record.md:37-40, 238-239`.

F4 — LOW — consistency — `decision-record.md` D10, the bound and its acceptance criterion
- Summary: "writes nothing outside that temporary directory but the Codex home" is broken by the logger.
- Claude's `Prepare` logs at Info (`claudeengine.go:158`, `logger.Info(... prompt-cache TTL)`). Codex's `readyHome` warns through `c.warn` (`codexengine.go:242`). Both write to lym's log outside the temporary directory.
- The bound should name the log, or say "writes no run state outside".
- Evidence: `claudeengine/claudeengine.go:158`, `codexengine/codexengine.go:242`, `decision-record.md:167-168, 254`.

**Tool calls:**
1. Grepped `reedengine` for how a pane command is typed (new-session or send-keys), and every production `ModelChoice` reassignment: to test D5's argument bound and D4's single-engine claim one hop past shuttle. My context held neither.
2. Grepped the `shuttleengine` and `modelspec` docs for "fallback": to check whether any path relaunches a spec on another engine and so meets D4's refusal. My context lacked it.
