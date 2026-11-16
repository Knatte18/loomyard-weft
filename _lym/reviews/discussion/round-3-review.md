---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/discussion/decision-record.md:209-215 and 236-239 (D12)
    summary: "D12 says every Codex model call is an exec cell and every payload tool_use_id is a CommandExecution or FileChange item id, but the cited rollout also holds top-level function_call items (spawn_agent, send_message), so the nudge's cell grouping is undefined for them."
    fix: "In D12, replace the sentence 'every model call is a custom_tool_call named exec' in the Facts with what the 0.164.0 lym rollout shows. The 21 model calls named exec are custom_tool_call items, and the collaboration namespace's spawn_agent and send_message are top-level function_call items (4 in rollout-2026-11-15T19-12-44-01a1e069); the tool tree also lists wait_agent, followup_task, interrupt_agent, list_agents, sleep, wait and clock. Then state the nudge's rule for a payload that is not a nested exec call. The rule must say which id a non-exec payload carries and where the nudge finds the call in the rollout, that such a call is neither a lookup nor an apply_patch-only cell and so ends a lookup_run or edit_run chain as a message holding an Agent call does on Claude, and that it is never attributed to the latest exec cell. Reword 'The nudge reads the rollout ... the payload's own call belongs to the latest cell' and 'Each payload's tool_use_id is the id of the call's item_completed item' so each holds only for a nested exec call. If whether PostToolUse fires for a collaboration or functions call is not known, run the spike (a spawn_agent call under a *-matcher PostToolUse hook logging its payload) and record its tool_name and tool_use_id with the rollout path, in place of leaving the rule silent."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:44-49 and 93 (D1 producer table and scan, D4 scan)
    summary: "The D1 producer table and the D4 scan treat every production shuttleengine.Spec literal as a launch spec, but internal/shedadapters/bouncer.go builds two probe specs (awaitLiveJudge, awaitLiveSeed) that set no prompt, model choice or engine and only reach AttachIfLive, so the scan fails on them and the table has no renderer to call for them."
    fix: "In D1's scan paragraph and in D4's Bound, narrow the scan to a spec literal that reaches a launch, the Prompt-carrying specs passed to Start, Run or Attach-then-run, and state that a literal passed only to an attach probe (bouncer.go awaitLiveJudge and awaitLiveSeed, which carry only OutputFiles, Role, Segment and Round) is skipped by D4's prompt-engine check and by the producer table. State the test the scan uses to tell the two apart, such as 'a literal that sets Prompt'. Also state in D1 that a producer which fills its prompt inline in the function that launches it, as bouncer.go judgeCall and runSeedSpawn do, gets its prompt render split out in that function's package so the table entry has a renderer to call with the engine; the 'Notes for the plan writer' line that says an entry 'calls that renderer' assumes one exists."
  - id: F3
    severity: LOW
    class: design
    location: _lym/discussion/decision-record.md:222-223 (D12 read-again rule)
    summary: "The read-again rule compares the file of a sliced Bash read with the path of an earlier parsed_cmd read, but parsed_cmd records the path as the command wrote it (notes.txt, relative), and D12 names no resolution against the call's working directory or the mtime comparison Claude's rule uses."
    fix: "In D12's read-again bullet, say that both paths are resolved to absolute against the call's working directory, recording in D12 which rollout or payload field carries it, and that an earlier read counts only when its message is not older than the file's last modification, as Claude's rule states. The 2026-11-16 spike rollout shows parsed_cmd entries of type read with a relative path and name, such as cat notes.txt and sed -n '1,2p' notes.txt."
---

### [BLOCKING] D12's call shape is false for the non-exec calls the rollout holds

**Location:** `_lym/discussion/decision-record.md:209-215`, `236-239`.
**Issue:** D12 rests on "every model call is a `custom_tool_call` named `exec`" and on "each payload's `tool_use_id` is the id of the call's `item_completed` item, a `CommandExecution` or a `FileChange`".
The cited rollout `~/.lym/codex/sessions/2026/11/15/rollout-2026-11-15T19-12-44-01a1e069-….jsonl` holds 21 `custom_tool_call` items named `exec` and also four top-level `function_call` items, two `spawn_agent` and two `send_message`, in namespace `collaboration`.
Its `additional_tools` record lists, besides `exec`, `request_user_input` and `request_user_input_async`, the `collaboration` tools `spawn_agent`, `send_message`, `followup_task`, `interrupt_agent`, `list_agents` and `wait_agent`, and `sleep`, `wait` and `clock`.
The two sibling rollouts of that day hold further `spawn_agent` and `send_message` calls the same way.
The `PreToolUse` evidence the record already cites (the twill spike) shows hooks fire for such calls, so a `*`-matcher `PostToolUse` entry is likely to fire for them too, but the record's own spike (2026-11-16) ran only `exec` cells and shows nothing about their payloads.
A `function_call` has no `item_completed` `CommandExecution` or `FileChange` item.
D12 says "the payload's own call belongs to the latest cell, whose output is not yet written", which is false for a `spawn_agent` call made after a finished cell, so the nudge would attribute it to the wrong cell or find no item for its `tool_use_id`.
D12 also leaves open whether such a call breaks a `lookup_run` or `edit_run` chain.
On Claude a message holding an `Agent` call is no lookup and no edit-only message, so the chain ends.
The units D12 counts and classifies therefore match only the `exec` calls of a run, not the run.

### [MEDIUM] The producer scan reaches probe literals that launch nothing

**Location:** `_lym/discussion/decision-record.md:44-49`, `93`.
**Issue:** D1's scan fails on a production launch-spec literal whose enclosing function has no table entry, and D4's scan fails on a production spec literal that sets neither a prompt engine nor the hand-written mark.
`internal/shedadapters/bouncer.go` holds four `shuttleengine.Spec{` literals in four functions.
Two of them, in `awaitLiveJudge` and `awaitLiveSeed`, carry only `OutputFiles`, `Role`, `Segment` and `Round` and go to `AttachIfLive`.
They set no prompt, model choice or engine, since their comment says building the full spec would mean filling the whole prompt for a probe.
The scan as written fails on both, and the table's rule that an entry "calls that renderer with the engine" has nothing to call for them.
The other two, in `runSeedSpawn` and `judgeCall`, fill their prompt inline in the function that launches, so the Notes' premise that a producer has a renderer its entry can call does not hold there either.
`internal/websterengine` and `internal/treadleengine` hold their literals one per function, which the per-function table handles.

### [LOW] The read-again rule names no path resolution

**Location:** `_lym/discussion/decision-record.md:222-223`.
**Issue:** D12 names the earlier command that counts as a read (a `Bash` call whose `parsed_cmd` lists a `read` of the path), which settles that part of the question.
The spike rollout shows that `parsed_cmd` carries the path as written, `notes.txt`, for `cat notes.txt` and for `sed -n '1,2p' notes.txt`, while Claude's `sliceBreach` compares absolute `file_path` values and the file's modification time against the earlier message's time (`internal/shuttleengine/claudeengine/batchingnudge.go`).
D12 states neither the resolution nor the "unchanged since" test for Codex.

## Settled by evidence

- The question deny: the tool tree lists exactly `request_user_input` and `request_user_input_async` in the `functions` namespace, and D12's matcher covers both by suffix and states both names in its Facts.
- `apply_patch`: the spike names `tool_name` `apply_patch` with the patch text in `tool_input.command`, with the rollout path as its source; the rollout shows a `FileChange` item.
- Manifest record after a done run: `internal/shuttleengine/wait.go` removes only `run.runDir` (`cleaned := outcome == OutcomeDone && !run.spec.KeepPane`), so the copy D1 puts in the `manifests` directory at the run-directory root survives, and the other root scanners skip a directory with no `run.json`; `sweepOrphans` would remove it and D1 says it is skipped.
- D4's engine: D4 names the new call that returns a model choice's resolved engine name, the set's default for an empty `ModelChoice.Engine`, and `EngineSet.DefaultName` already exists for it to build on.
- The shared heddle targets: D1 keys `agents-file` and `agents-prompt` items to the shared target and compares them.
