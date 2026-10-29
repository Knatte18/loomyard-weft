```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the diff; no tool calls needed.

**List (function/branch, case run, verdict)**

- `BoundaryBlock`, tool-boundary event, text `read "role.md"`: `Reason` and `SystemMessage` are both the text, and `HookSpecificOutput` is nil, so `omitempty` drops the key. The JSON is `{"decision":"block","reason":…,"systemMessage":…}`, the same fields in the same order as before, so it stays byte for byte. Correct.
- `BoundaryBlock`, Stop event, same text: `Reason` becomes `stopBlockReason`, `SystemMessage` stays the text, and `hookSpecificOutput` is `{hookEventName:"Stop",additionalContext:text}`. The mail text appears only in `systemMessage` and `additionalContext`. `decision: block` is kept. Matches the `StopBlock` row in `TestHookContext`. Correct.
- `BoundaryBlock`, empty text on a Stop: gives `additionalContext:""` and the fixed reason. `hook-read` writes only when messages were claimed, so this can't happen on the real path. Correct.
- `BoundaryBlock`, an event other than Stop, such as the zero value: takes the tool-boundary branch. `hook-read` rejects every other event earlier, so this is unreachable. Correct.
- `BoundaryBlock`, `json.Marshal` failure: wrapped and returned as before. The structs hold only strings, so it can't fail. Correct.
- `hookRead` `writeBlock`: passes `input.Event`. A subagent call returns before the closure is built, and an event other than tool-boundary or Stop is refused before it. A failed write leaves `wrote` false and the error is returned unchanged. Correct.
- `hookReadExit`: unchanged. A Stop that wrote still exits 0, so the merged turn-end command still sees a block. Correct.
- Interface change: the only implementer in the manifest is `claudeengine.Claude`, and the callers are `hookread.go`, `cli_test.go`, `settings_test.go` and `shuttleengines_test.go`, all updated. A search of the manifest found no other `BoundaryBlock` caller. Correct.
- `TestScheduler_HookWake`, blocked-chain row, with messages posted before `e.start`: no worker exists during the posts. `settle(e, start)` takes `seen = len(probeWaits(0))` before the start, then waits for one more probe wait. The new worker probes once at startup, and no wake is buffered because nothing was posted after the start. Before the start, `SetTurn` and `register` run without a worker, and `register` (`HookWait`) calls no probe. The probe then sees the registered waiter and two claimable messages, so alpha's waiter is rung, which `wantKind(first, Ring)` expects. Plausibly correct. Whether the posts wake a not-yet-running scheduler, or whether `start` runs an extra initial probe, depends on `e.start` and the worker code, which I did not read. The commit message reports 0 of 900 failures, which supports it.
- `TestScheduler_HookWake`, Stop-cap row, 8 posts then start-settle: the same reasoning. The `ARingAfterTheStopCapStartsAFreshCount` row has no waiter registered before the start, so the startup probe rings nothing. The later steps in the row aren't visible in the diff, so I can't judge them. No defect shown.
- Other rows in `TestScheduler_HookWake` that post several messages in one trigger: they aren't in the diff. I did not check whether any remain with the same double-wake pattern. This is the coverage question the focus asks about, and I can't answer it without reading the rest of the test.

**Findings**

None from the code I could run through step by step.

**Tool calls:** none made; the manifest already held the diff, `hookread.go`, `engine.go` and the `deliver_test.go` helpers.
