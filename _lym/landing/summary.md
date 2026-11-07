# Batching hook: wait for the call to reach the transcript, then judge

The batching hook fired on 2 of 156 edit follows in the 2026-11-05 re-measure, although a replay of a recorded edit follow fired.
This change finds the cause and fixes it, so the hook can be measured again before it is kept or removed.

## Cause

Claude Code writes an assistant message's lines to the transcript only after the message's tool calls have started.
A call that finishes fast, such as Read, Grep, Glob or Edit, runs its PostToolUse hook before its own message is in the transcript.
The judgment then found no message holding the call's `tool_use_id`, and `messageHolding` returned -1, so the hook stayed silent.
A replay reads the finished transcript, which is why it fired.

Probes from inside a live session show the lag.
A Bash command polled its own transcript for a marker in its own command line.
The line appeared 5 to 40 ms after the command started when the call was the last of its message.
It appeared about 4.1 s later when the call was the first of three parallel calls, since its line waits until the model has finished writing the whole message.
This matches the report: all 8 live lookup firings were on Bash calls, which outlive the write, and edit chains almost never fired.

## Change

- `BatchingNudge` in `internal/shuttleengine/claudeengine/batchingnudge.go` now waits for the call before it judges.
  `awaitCallInTranscript` rereads the transcript tail every `batchingTranscriptTick` (25 ms), at most `batchingTranscriptAttempts` (80) times, until a message holds the call.
  It rereads only when the file has grown.
- The state file is read before the wait, and the first tail's context reading is checked before any sleep.
  So a run with no state file, a chain past its nudge cap and a chain below `context_tokens` never wait.
  In a chain above the threshold, the wait adds latency to each judged call until the call's line is written.
- The judgment under the state lock uses the tail the wait found, and its other checks are unchanged.
- `Claude` carries the wait as the `transcriptWait` field, set by `New` and `NewFromConfig`.
  Tests pass a sleep function that writes the missing message instead of waiting.
- The package doc describes the wait.

## Design decisions

- The wait is a bounded poll on the file's size, not an `fswatch` watch: the hook is a short-lived process per call, and a bounded wait under a minute is outside `PATTERN-daemon-wakeups`.
- The two-second bound covers the write delay after a message ends.
  The early calls of a parallel message still being written can outlast it and stay silent.
  Such a message holds several calls, so it is never a single-lookup breach.
  For an edit message, the later edits' hooks see the whole message.
- The bound is a constant, not a `claude.yaml` key: no run needs to tune it, and a new key would need a config meaning change.
- The task's other way forward, removing the hook, is left to a new measurement.
  Note `batching-hook-remeasure-wait` (R7) asks to re-run the report's scripts once this has landed.

## Tests

`TestBatchingNudge` gains a row for an edit chain whose last message reaches the transcript during the wait, and fires.
It also gains rows for a late call below the context threshold and a late call past the cap, which stay silent without waiting.
Every row now asserts how many times the engine waited.
The absent-call row asserts that the wait stops at its bound.
