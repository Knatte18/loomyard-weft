```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I took one concrete case per decision through its mechanism.

**Decisions walked**
- D1, the darn-writer seat plus a second producer added to `websterengine`: purpose not met. The package-keyed scan passes a new producer inside a package that already has a table entry (finding 3).
- D2, the darn make-change fragment rendered for Codex: met.
- D3, the treadle judge prompt on Codex through `editdirective.Directive(engine)`: met.
- D4, `lym shuttle run --prompt-file` on Codex, then a gatefixer spec with no prompt engine: met. The first is marked hand-written and launches; the second is refused.
- D5, standing items riding `-c developer_instructions=…`: met. The pane line reaches the shell through reed's launch script (`internal/reedengine/spawn.go:76-77`), so the 128 KiB per-argument cap is the binding limit, as D5 says.
- D6, a six-lens fork fan on Codex with the thread cap at 16: met.
- D7, `lym-batching` moved to `agents-file` and re-rendered by the hub reconcile: met.
- D8, the scribe plugin's SessionStart hook run at `Prepare` with plain-text stdout: met.
- D9, a repo with both `AGENTS.md` and `CLAUDE.md` at the root: met.
- D10, `lym shuttle manifest` for the darn-writer seat under `sandbox_mode: auto`: met. Codex `Prepare` writes the home links, trust record and `config.toml`, which D10's bound names.
- D11, the board writes to `codex-engine-trial` and `codex-writer-bench-osl`: met.
- D12, question deny, case a Codex `request_user_input_async` call: not met (finding 1).
- D12, batching nudge, case a lookup chain and an `apply_patch`-only run in a Codex 0.164.0 rollout: not met (finding 2).
- D12, read-again slice, case `cat a.go` and then `sed -n 10,20p a.go`: not met (finding 4).

**Findings**

1. MEDIUM, design. `decision-record.md` D12, the question-tool deny.
   - Summary: the suffix match "ends in `request_user_input`" lets `request_user_input_async` through. Codex 0.164.0 offers that second question tool in a lym run, so an autonomous Codex session can still ask the operator a question.
   - Evidence: the tool tree in rollout `~/.lym/codex/sessions/2026/11/05/rollout-2026-11-05T10-11-18-01a1aafa-….jsonl` line 3, under namespace `functions`, lists `request_user_input` and then `request_user_input_async` ("Ask the user one or more questions during ongoing work").
   - The record's own fact (`decision-record.md:202-205`) names only `request_user_input`.

2. MEDIUM, design. `decision-record.md` D12, nudge classification (lines 188-196).
   - Summary: D12 classifies "shell calls", "`apply_patch` calls" and "a response holding only `apply_patch` calls" read from the rollout. In 0.164.0 the rollout does not hold calls in that shape.
   - The model's tool is a JavaScript `exec` cell. One cell may run any number of `tools.exec_command` and `tools.apply_patch` calls, and they show up only as `item_completed` `CommandExecution` or `FileChange` items.
   - So the step that sorts a response into lookup-only or edit-only is handed JS cells, not the calls D12 assumes. What counts as one "response" is undefined.
   - No source records which `tool_name` an `apply_patch` call carries in the `PostToolUse` payload, or whether the hook fires for it at all.
   - Evidence:
     - the same rollout: every `custom_tool_call` has `"name":"exec"`; one input reads `const patch = "*** Begin Patch…"; const r = await tools.apply_patch(patch);`; the item types are CommandExecution 46 and FileChange 3;
     - `docs/research/codex/codex-spike-2026-10-24/report.md:251` lists only `custom_tool_call`;
     - `stencil-engine-audit-2026-10-24/report.md:361` says no edit tool was ever recorded.

3. MEDIUM, design. `decision-record.md` D1, the producer scan (lines 37-43).
   - Summary: the scan fails only on a spec literal "in a package the table has no entry for". A second producer added inside a package that already has an entry passes the scan, and no parity case covers it. So "every producer" (Goal and acceptance line 238) is not held.
   - Evidence: production `shuttleengine.Spec{` literals sit at four sites in `internal/shedadapters/bouncer.go` (909, 937, 1728, 1882) and three in `internal/websterengine` (`planfixer.go:158`, `runlevel.go:752`, `recoverbatch.go:328`).

4. LOW, design. `decision-record.md` D12, the read-again rule (line 194-195).
   - Summary: a Codex sliced read is "judged against … the read-again rule as a sliced Read is", but D12 never says which earlier shell command counts as a read of that file. Claude's rule matches the structured `file_path` of an earlier `Read` call; a Codex shell command carries no such field.
   - So in the case `cat a.go` then `sed -n 10,20p a.go`, the step has no earlier-read input to compare against.
   - Evidence: `internal/shuttleengine/claudeengine/batchingnudge.go:538-547`, which matches `call.name != toolRead || call.filePath != filePath`.

**Tool calls**
- Grep of `internal/reedengine` and `internal/tmuxkit` for the tmux verbs: to learn how the launch line reaches the pane (D5 walkthrough); the manifest had no reed code.
- Grep of `internal/reedengine/spawn.go` for `send-keys` and the launch script: same reason, to confirm the line goes through a sourced script and not a tmux argument.
