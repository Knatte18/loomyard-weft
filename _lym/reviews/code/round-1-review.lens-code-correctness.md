```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. For each function and branch the diff adds or changes, I ran one concrete case, including the failure paths.

**Per function or branch: the case run, and the verdict**
- `editLoopWatch.fold`, a non-file-edit signal: skipped. Correct.
- `fold`, a zero `At`: takes `now`. Correct.
- `fold` with `edits=1`: every edit completes a loop and resets the count. That is correct for the config, and the 1 is the operator's choice.
- `fold` with `edits=3`, `window=60s` and edits at 0 s, 55 s, 110 s: 110−0 > 60 drops the first, leaving a count of 2 and no loop. Correct.
- `fold`, an edit exactly `window` after the oldest: kept, because the drop test uses `>`. Correct.
- `fold`, the count restart after a loop: `delete(w.edits, key)`. Correct.
- `fold`, out-of-order times: with times=[200, 100] (an unstamped edit took run-clock `now` ahead of a later-filed stamped edit), `window=60` and at=250, the front-only trim stops at 200, so the 100 entry stays and is counted although it is 150 s old. This is an overcount (finding 4).
- `fold`, map growth: an entry holds at most edits−1 times per (agent, file) and lives for one `Wait`. It is bounded by the files edited. Not a defect.
- `watchEditLoops` guards (notifier nil, interactive, covered, edits ≤ 0, engine without a parser): each returns before reading. Correct.
- `watchEditLoops`, `readEventsFrom` error or empty data: silent return, then a retry next tick from the same cursor. Correct.
- `watchEditLoops`, a trailing stamp with no payload yet: the parser holds it unconsumed, so the cursor stops before it. Correct.
- `watchEditLoops`, a failed persist: the cursor is already advanced and `fold` has already deleted the loop's key, so the notice is lost for good with one Warn, and there is no retry. `notifyDialog` returns false on the same failure and retries on a later tick. This is finding 2.
- `watchEditLoops`, a reattach after a notice: the cursor restarts at `EditLoopNotifiedOffset`, past the whole batch. Partial counts of other files from that batch and from earlier ticks are forgotten. This is finding 3.
- `watchEditLoops`, a notifier error: logged, and the offset stays persisted. That matches the dialog notice. Correct.
- `persistEditLoopNotifiedOffset`, a `saveState` failure: the previous value is restored under `recordMu`. Correct.
- `editLoopNotice`, a file path with a newline or `«»`: `delimitedAgentText` maps control characters and the delimiters to spaces and truncates at `noticePartMaxRunes` (hold.go:41-51). The line stays single. Correct.
- `editLoopNotice` wording: the window prints as Go's `time.Duration` form, "5m0s". It reads oddly but is correct.
- `RecordEdit`, an Edit with a `file_path`: the stamp and slim line go out in one `O_APPEND` write and parse back to one `SessionSignalFileEdit` with `At` and `AgentID`. This is verified by `TestRecordEdit`. Correct.
- `RecordEdit`, the Read tool, an empty `file_path`, or malformed JSON: nothing is appended, and malformed JSON is an error. Correct.
- `RecordEdit`, an open or write failure: wrapped, and the close is joined into the error. Correct.
- `RecordEdit`, concurrency with the other hooks' appends: its own write is a single call. However, `recordingCommand` writes its payload and its newline as two separate appends (settings.go:397). This is finding 1.
- `fileEditSignal`, a missing `tool_input`: the nil-map index gives "", so no signal is emitted. Correct.
- `fileEditSignal` with an `agent_id`: carried through. Correct.
- `SessionFold.applySignal` with a file edit: returns before setting `newest`, so the turn-start/idle-notice contradiction holds. Correct.
- Other `ParseSessionSignals` consumers (liveness, gatefixer, `send.go`, `eventsFileActivity`, `stampEventTimes`, `positionedTurnStarts`): each filters by kind or `Event`, so a file edit changes none of them. Correct.
- `hookEditCmd` and `hookEdit`, an unknown engine, malformed input, or a stdin read failure: reported on stderr, `RunE` returns nil, and the exit code is 0. Correct.
- `hookEditCmd`, a wrong argument count: cobra's `ExactArgs` fails with a non-zero exit, and the hook's `; true` masks it. Correct.
- `cli.go` `PersistentPreRunE`: skips resolution for `EditHookVerb`. Correct.
- `shuttleengines.EditRecorder`, an unknown name or the codex engine: returns false. For claude, `now` is set through the constructor. Correct.
- `LoadConfig`, edits −1, window 0 with edits on, or edits 0 with window 0: refused, refused, and accepted. Correct.
- `buildSettings`, autonomous with edits=4, interactive, or edits=0: the hook is present only for the autonomous run. The caller passes the run's cfg (claudeengine.go:232). Correct.
- `Wait` cursor init: `max(PromptOffset, EditLoopNotifiedOffset)`. Correct, with finding 3 as the caveat.

**Findings**

1. MEDIUM, design. Location: `internal/shuttleengine/claudeengine/editrecord.go:68-70` together with `settings.go:395-398`.
   - The edit hook adds a new appender to `events.jsonl` that runs at the same time as the recording hooks. A background subagent's or fork's Edit hook can fire while the main session's Stop, Notification or UserPromptSubmit recording runs.
   - `recordingCommand` appends the payload (`cat >>`) and its newline (`printf '\n' >>`) as separate writes. An edit write landing between them produces `{stop payload}{"lym_stamp":"PostToolUse",...}` on one line.
   - `ParseSessionSignals` drops that malformed line (signals.go:105-107). The Stop turn end is lost, and the edit takes a zero time.
   - Before this diff, PostToolUse only touched the activity stamp, so per-edit appends to the events file are new.
   - Evidence: settings.go:397 (`cat >> … && printf '\n' >> …`), editrecord.go:68-70, signals.go:104-107.

2. LOW, design. Location: `internal/shuttleengine/editloop.go:98-113`.
   - When the persist of `EditLoopNotifiedOffset` fails, the loop is lost for good: `fold` has already deleted the loop's key, and `run.editLoops.cursor` is already past the batch.
   - No later tick retries, and the count restarts. The only trace is a Warn.
   - The dialog notice keeps `notified=false` and retries on the same failure (dialog.go:122-126). A transient write failure therefore drops this notice silently.
   - Evidence: editloop.go:104 (cursor advanced), :106 (fold deletes keys), :110-113 (return on error).

3. LOW, design. Location: `internal/shuttleengine/editloop.go:110` together with `wait.go:307`.
   - The persisted offset is the cursor past the whole batch. A reattach resumes there and forgets every partial count before it, including other files' edits in the same batch as the notified loop.
   - Example with `edits=4`: one batch holds four edits of a.go and three of b.go. After a reattach, b.go needs four more edits instead of one.
   - So the persisted offset prevents repeat notices at the cost of undercounting other files after a reattach.
   - Evidence: editloop.go:110 (`persistEditLoopNotifiedOffset(run.editLoops.cursor)`), wait.go:307.

4. LOW, design. Location: `internal/shuttleengine/editloop.go:51-54`.
   - The window trims only from the front and assumes `times` is in time order. An edit whose time is earlier than `times[0]` is appended behind it and never trimmed while the front stays within the window, so the count can include an edit older than the window.
   - It happens when an unstamped edit falls back to the run clock's `now` (it is read later than when it happened) and a stamped edit follows it in the file. It also happens within the second-granularity stamps of parallel hooks.
   - Evidence: editloop.go:44-54.

Tool calls:
1. grep of `delimitedAgentText` (hold.go) and of `buildSettings`'s production caller: the manifest did not show how the notice cleans an agent-written path, or that Prepare passes the run's cfg to the settings.
