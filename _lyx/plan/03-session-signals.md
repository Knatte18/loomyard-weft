# Card 3 — session-signals

**Create:**
- `internal/shuttleengine/sessionsignal.go`
- `internal/shuttleengine/claudeengine/signals.go`
- `internal/shuttleengine/claudeengine/signals_test.go`
- `internal/shuttleengine#SessionSignal`
- `internal/shuttleengine#SessionSignalKind`
- `internal/shuttleengine#SessionSignalParser`
- `internal/shuttleengine/claudeengine#Claude.ParseSessionSignals`

**Edit:**
- `internal/shuttleengine/claudeengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine/claudeengine#Claude.ParseEvents`
- `internal/shuttleengine/claudeengine#outstandingBackgroundTasks`
- `internal/shuttleengine#BackgroundTask`

**Intent:**
Add the session-signal parser (D2) as an optional capability beside `Engine`, separate from `ParseEvents`, which stays unchanged together with `EventKind` and the wait loop's classification.

`sessionsignal.go` declares the provider-neutral half:
- `SessionSignalKind` with the kinds turn start, turn end, API-error turn end, ask, idle notice and session end.
- `SessionSignal`: its kind; `At`, the hook-side time, zero when the line has none; `SessionID`, the hook line's session id; `Outstanding`, a turn end's background tasks with their reporting signal (`SignalPayload` or `SignalTranscript`); `ErrorText` for an API-error turn end; `Reason` and `EndsProcess` for a session end, so the provider decides which reasons end its process; `Event`, true when `ParseEvents` also yields an event from the same line (a Stop or an AskUserQuestion record); and `Raw`, the payload line, which the transcript readers of later cards take.
- `SessionSignalParser` with one method, `ParseSessionSignals(data []byte) (signals []SessionSignal, consumed int)`: signals in file order, and `consumed`, the byte count a following incremental read resumes from.
  `consumed` ends at the last complete line, except that it stops before the earliest line of the trailing run of stamp lines with no payload line after them, so an incremental reader re-reads every such stamp with its payload at its next poll (the added decision "Incremental reader holds a trailing stamp"); two hooks that stamp before either payload lands leave two such stamps.

`signals.go` implements it for Claude, with every payload fact kept in `claudeengine` (`PATTERN-shuttle-provider-seam`):
- A stamp line is `{"lyx_stamp":"<hook event name>","lyx_at":"<RFC 3339 UTC>"}` (D1); its format is declared here once, as the constants card 5's hook commands write.
- A payload line takes `At` from the nearest preceding stamp line that names its `hook_event_name` and that no other payload line has taken; a payload with no such stamp, or whose stamp's `lyx_at` is empty or malformed, has a zero `At`.
- `Stop` is a turn end, its outstanding tasks read through `outstandingBackgroundTasks` exactly as `ParseEvents` reads them.
- `StopFailure` is an API-error turn end, its text from `error`, else `error_details`, else `last_assistant_message`.
- `UserPromptSubmit` is a turn start.
- `PreToolUse` with `tool_name` `AskUserQuestion` is an ask; `Notification` whose `notification_type` is a permission prompt or an elicitation dialog is an ask, `idle_prompt` is an idle notice, any other type is skipped.
- `SessionEnd` is a session end with its `reason`; `EndsProcess` is true for `logout`, `prompt_input_exit` and `other`, false for `clear` and `resume`, and false for an unknown reason.
- Any other line, a malformed line and a stamp line yield no signal, leniently, as in `ParseEvents`.

`Claude` asserts it implements `SessionSignalParser` at compile time.

Tests, untagged, in `signals_test.go`, one table over hand-built lines: each hook's payload reads its kind and fields; a payload with its stamp reads that stamp's time; two stamps of different hooks before their payloads pair each payload with its own hook's stamp; a payload with no stamp, an empty `lyx_at` and a malformed `lyx_at` read a zero `At`; a stamp split from its payload across two calls, the second resuming at `consumed`, pairs as one whole-data call does; a trailing stamp with no payload is left outside `consumed`; two trailing stamps of different hooks are both left outside `consumed`, and a second call resuming there with both payloads appended pairs each as a whole-data call does; stamp lines and unknown hooks yield nothing; `Event` is set exactly on Stop and AskUserQuestion lines.
A transcript-fallback row pins the added decision "Transcript-fallback turn ends are per reader": a Stop line with no `background_tasks`, whose transcript lists a running shell, reads a turn end with that shell under `SignalTranscript`; the same data parsed again after the test appends the shell's completion notification to the transcript reads the turn end with no outstanding task.
`cmd/lyx/testdata/test-budget.yaml`: the `internal/shuttleengine/claudeengine` row rises by 1, for the one new top-level function in `signals_test.go`.

Docs: `claudeengine/doc.go` describes the session-signal parse: which hook lines become which signal kinds, the stamp pairing it performs, that a transcript-fallback turn end's tasks are read at parse time and so depend on when it is read, and which `SessionEnd` reasons end the process.

**ImpactSummary:** Adds the optional SessionSignalParser capability and Claude's implementation; the package doc and the test budget row are the only existing files touched, and ParseEvents and every existing caller are unchanged.
