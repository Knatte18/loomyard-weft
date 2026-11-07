# Batching hook: wait for the call to reach the transcript

The batching nudge hook almost never fired live on edit follows, though a replay of it on a recorded edit follow fired.
The cause is write order:
Claude Code writes a message's lines to the transcript only after its calls start.
A fast call such as a Read or an Edit therefore ran its after-tool-call hook before its own message was there to judge,
and the judgment found no message holding the call and stayed silent.

`BatchingNudge` in `claudeengine` now rereads the calling chain's transcript tail until a message holds the call.
It rereads every 25 ms, at most 80 times, so a call waits up to about two seconds.
A call still missing at the bound gets no nudge, as before.
The wait is the `batchingTranscriptWait` field of `Claude`, set by `New` and `NewFromConfig`;
tests replace its `sleep` to change the transcript instead of waiting.

## Design decisions

- The state file is read before the wait, so a run with no state file and a chain past its nudge cap never wait.
  The judgment still re-checks both under the state file's lock, so the parallel calls of one message nudge at most once.
- The first tail's latest context reading is checked against `context_tokens` before waiting,
  so a chain below the threshold never waits.
  While the call's message is missing, that reading is the previous message's,
  so the one call whose message first reaches the threshold gets no nudge.
  This trades one missed nudge per chain for no wait on chains that would stay silent.
- The tail is reread only when the transcript file's size changed since the last read.
- A transcript that cannot be read still gets no nudge and no error.
- `doc.go` and the `BatchingNudge` comment state the wait and its silences.
- The `hookbatching_test.go` corrupt-state row now matches the shorter `the batching nudge state` error text,
  since the state is read directly before the update.

## Tests

`TestBatchingNudge` gains rows for a call that reaches the transcript during the wait, for main sessions and subagents,
after one reread and after several.
It also gains rows for the bounded wait ending silent and for each case that never waits.
`TestEngineConstructorsSetTheTranscriptWait` pins that both constructors set the production wait.

## To check by hand

- `constructors-transcript-wait-unpinned` (`claudeengine.go`, `New` and `NewFromConfig`):
  a review left this finding open; confirm `TestEngineConstructorsSetTheTranscriptWait` covers both constructors.
- The fix is only unit-tested against a simulated late write.
  Run the batching hook live on edit follows, as the remeasure report did, and confirm edit firings now meet edit follows.
  The report is `docs/research/turns/batching-hook-remeasure-2026-11-05`.
- Confirm the wait of up to two seconds per judged call fits the hook's timeout where Prepare installs it.
