# Shuttle reads Claude session state from hooks before the pane

Shuttle now takes a Claude session's state from its recorded hook lines whenever they settle it.
It reads the pane only for what no hook carries: startup gates, the input box and its drafts, a menu the operator opened, and operator activity.
This removes several pane-only guesses: a send racing a fresh session (#549's path), a run that waits out its deadline after Claude crashed, and a lone stray character that blocked a resume (#554).

Closes #554

## What changes

- **Recording**: `buildSettings` installs a recording `SessionStart` hook for every source on every run, and a recording `PermissionRequest` hook.
  A run with a context-after-compaction command keeps that command after the recording one, on `compact` only.
- **Fold**: a new state `idle-fresh` (cause `session-start`) covers a started session with no turn yet.
  `resume` and `clear` starts set it; a `startup` start sets it unless a turn is running; `compact`, empty and unknown sources leave the state unchanged.
  `judgeReadiness` reads it like `idle-done`, and the orch watcher and the wait shadow treat it as idle.
- **Provider seam**: `claudeengine` maps Claude's source strings, `Notification` types and the stray-character judgment; `shuttleengine` sees only provider-neutral kinds, sources and box readings.
- **Asks**: `PermissionRequest` reads asking at once.
  Every `Notification` type is in a fixed table: `permission_prompt`, `elicitation_dialog` and `elicitation_url_dialog` are asks, `idle_prompt` is the idle notice, the rest yield no signal.
  The transcript's interrupt marker releases an ask to `idle-stalled`.
- **Startup**: for an engine that records session starts, a run counts as started only once the events file holds a `SessionStart` line and the pane reads ready.
  Gate detection and dismissal are unchanged.
  A timeout with a ready pane and no line names that cause and the events file path.
  An engine without session-start recording keeps pane-only startup.
- **`/clear` and `/compact`**: a send counts as landed on a matching `SessionStart` line past the pre-send events offset; the box read stays the fallback.
- **Dead process**: `checkLivenessTick` ends the run when a process-ending `SessionEnd` lies past this wait's first-tick events-file size and the registry does not read the process alive, or when the registry proves a pid it read alive is gone.
  The outcome is `done` when every output file exists, else `died`.
  An unproven liveness ends nothing.
- **Stray character** (#554): a lone punctuation character in an otherwise idle box, with no operator activity within `operator_quiet_s`, is cleared once behind a full guard reading.
  A second full guard reading follows the redraw settle and is the write's guard reading.
  A character that survives the clear reads as residue and the send proceeds.
  Every draft hold, including the pane-fallback one, quotes the box text, and batten's driver-busy notice carries it.
- **Fork transcripts**: `holdsPendingToolCall` reads from the tail and stops after the newest assistant message with `tool_use` blocks, so its cost no longer grows with the transcript.
- **Corpus**: an `llm`-tier capture scenario in `internal/shuttlecli` records real hook lines, and the trimmed result is the `hook-capture` corpus case under `claudeengine/testdata/corpus/`.
  It shows `SessionStart(startup)` landing on its own before the first prompt.

Docs: `internal/shuttleengine/doc.go`, `internal/shuttleengine/claudeengine/doc.go` and `pattern/PATTERN-pane-text-injection.md`, plus `docs/overview.md` and the `battenshed` and `orchengine` docs.

## Design decisions

- An engine without `SessionSignalParser` behaves as before on every touched path, and an unknown reading still falls back to the pane.
- A `SessionEnd` before this wait's first liveness tick never ends the run.
  This keeps a resumed launch, which shares the run's events file, from failing on an earlier launch's end line.
- The gate's trust-dismiss sequence is unchanged: a gate key may still be sent after `SessionStart` lands.
- A granted permission writes no hook line, so the run reads asking for the rest of that turn; batten or the orch watcher may show it asking for that span.
- A residue send whose post-paste box holds the character beside the sent text fails with `ErrSentTextInDraft`, sends no key and records the pending send.
  The `PATTERN.md` paste line is unchanged; its background entry gains the clear and the residue reading as bounded exceptions.
- Not built: a `PostToolUse` heartbeat for batten's quiet notice, and `idle_prompt` as proof of an empty box.

## Check by hand

- The `hook-capture` corpus case: the capture scenario was written to the corpus format (`case.yaml`, `events.jsonl`, `transcript.jsonl`, with the gate and registry ordering as free `case.yaml` fields); confirm it replays cleanly in `TestCorpus_ReplaysEachTurnEnd`.
- The `InputBoxClearer` doc in `internal/shuttleengine/engine.go` and the `Claude.ClearInputSequence` doc in `claudeengine/session.go`: both assumed the clear goes only into a box holding the sent text, and now also go into a stray-character candidate; confirm both read correctly.
- The `llm`-tier capture scenario in `smoke_turns_test.go` and the real-claude spawn-site count in `modelpin_test.go` were not run in a gate here; run `lyx gate test --tags llm ./internal/shuttlecli`.
- If the registry entry is not yet written when `SessionStart(startup)` lands, the fold reads unknown and the first sends take the pane fallback, as before.
