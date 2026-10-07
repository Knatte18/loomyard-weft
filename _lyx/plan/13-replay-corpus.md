# Card 13 — replay-corpus

**Create:**
- `internal/shuttleengine/claudeengine/corpus_test.go`
- `internal/shuttleengine/claudeengine/testdata/corpus/issue411-prune-rest/case.yaml`
- `internal/shuttleengine/claudeengine/testdata/corpus/issue411-prune-rest/transcript.jsonl`
- `internal/shuttleengine/claudeengine/testdata/corpus/issue411-prune-rest/events.jsonl`
- `internal/shuttleengine/claudeengine/testdata/corpus/issue411-bugfix-sessions/case.yaml`
- `internal/shuttleengine/claudeengine/testdata/corpus/issue411-bugfix-sessions/transcript.jsonl`
- `internal/shuttleengine/claudeengine/testdata/corpus/issue411-bugfix-sessions/events.jsonl`
- `internal/shuttleengine/claudeengine/testdata/corpus/planwrite-ask-answered/case.yaml`
- `internal/shuttleengine/claudeengine/testdata/corpus/planwrite-ask-answered/transcript.jsonl`
- `internal/shuttleengine/claudeengine/testdata/corpus/planwrite-ask-answered/events.jsonl`
- `internal/shuttleengine/claudeengine/testdata/corpus/master-fork-running/case.yaml`
- `internal/shuttleengine/claudeengine/testdata/corpus/master-fork-running/transcript.jsonl`
- `internal/shuttleengine/claudeengine/testdata/corpus/master-fork-running/events.jsonl`
- `internal/shuttleengine/claudeengine/testdata/corpus/fork-killed-api-error/case.yaml`
- `internal/shuttleengine/claudeengine/testdata/corpus/fork-killed-api-error/transcript.jsonl`
- `internal/shuttleengine/claudeengine/testdata/corpus/fork-killed-api-error/events.jsonl`
- `internal/shuttleengine/claudeengine/testdata/corpus/skill-load-turn-end/case.yaml`
- `internal/shuttleengine/claudeengine/testdata/corpus/skill-load-turn-end/transcript.jsonl`
- `internal/shuttleengine/claudeengine/testdata/corpus/skill-load-turn-end/events.jsonl`
- `internal/shuttleengine/claudeengine/testdata/corpus/session-continued-in/case.yaml`
- `internal/shuttleengine/claudeengine/testdata/corpus/session-continued-in/transcript.jsonl`
- `internal/shuttleengine/claudeengine/testdata/corpus/session-continued-in/events.jsonl`

**Edit:**
- `internal/shuttleengine/wait.go`
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/testkit/shuttlefake/shuttlefake.go`

**Intent:**
D10, the offline half: a replay corpus of real misreads, tested by one untagged table test through public surfaces.

The clock seam goes public so a test outside the package can drive Wait's time: the unexported `clock` interface in `wait.go` becomes the exported `Clock` (its uses renamed), and `Runner` gains `SetClock(c Clock)`, documented as the time seam for a caller that replays a run; production wiring never calls it.

The corpus lives under `claudeengine/testdata/corpus/`, one directory per case, each holding:
`transcript.jsonl`, the real transcript lines that matter, trimmed to the lines and fields the parser reads, carrying no secrets;
`events.jsonl`, the run's events file, or, when the real one is gone, one reconstructed from the transcript's turn ends;
`case.yaml`, giving the source run (transcript path and session id), the date, the Claude Code version, what went wrong, whether `events.jsonl` is reconstructed, whether the run is gated, and the expected reading of each turn end in order: `done`, `held`, or `waiting` with each outstanding task's kind, id and signal, plus whether every output file exists at that turn end.
Each `transcript_path` in `events.jsonl` is rewritten to the case's own `transcript.jsonl` at load time.

The first cases, sourced from real transcripts under `~/.claude/projects/`:
the two #411 absorbed-notification runs (paths and line numbers in the decision record's Notes), each gated, whose Stop with `background_tasks: []` reads `done`;
the `bugfix-sessions` Plan-Write ask answered in its own session, reading `held` and then `done`;
a Webster Master turn end with a fork still running, reading `waiting` with the fork;
a fork killed by an API error;
a skill-load turn end;
a session `continued-in` another.
Each case's expected reading is the one the fixed code must give for what really happened in that run, stated beside what the old code read.

`corpus_test.go` is one table test over every directory under `testdata/corpus/`, so a new case directory joins without a code change, and a case whose reading differs fails naming the case and the turn end.
Per case it builds a real `shuttleengine.Runner` over `shuttlefake.Reed` and an engine whose `ParseEvents` is the real `Claude.ParseEvents` (the rest from `shuttlefake.Engine`), with an injected `Clock` and a recording notifier.
It starts the run (gated with a passing gate entry when the case says gated), then lets the injected clock's `Sleep` append the next turn end to the run's events file, creating the output files when the case says they exist at that turn end, so each poll tick reads exactly one turn end.
It asserts, per turn end: `done` finalizes Done on the tick that reads it; `held` calls the notifier exactly once and does not finalize; `waiting` neither notifies nor finalizes, and `ParseEvents` on that line reports exactly the listed tasks.
It spawns nothing and reads no real clock (`PATTERN-test-speed`); fixtures are files under `testdata`, so `t.Parallel` holds.

Add a short corpus section to `claudeengine/doc.go`: where the corpus lives, what a case holds, and that promoting a failing live scenario (card 14) is a copy and a trim.

**ImpactSummary:** Exports shuttleengine's clock seam as Clock with Runner.SetClock; adds test data and one untagged test.
