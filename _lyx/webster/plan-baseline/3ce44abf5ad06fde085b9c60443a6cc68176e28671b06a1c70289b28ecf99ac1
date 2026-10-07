# Card 15 — replay-corpus

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
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/testkit/shuttlefake/shuttlefake.go`
- `plan:internal/shuttleengine#Runner.SetNotifier`
- `plan:internal/shuttleengine#Runner.SetClock`

**Intent:**
D10, the offline half: a replay corpus of real misreads, tested by one untagged table test through public surfaces.

The test drives `Wait`'s time through the `Clock` seam and `Runner.SetClock` that card 14 adds.

The corpus lives under `claudeengine/testdata/corpus/`, one directory per case, each holding:
`transcript.jsonl`, the real transcript lines that matter, trimmed to the lines and fields the parser reads, carrying no secrets;
`events.jsonl`, the run's events file, or, when the real one is gone, one reconstructed from the transcript's turn ends;
`case.yaml`, giving the source run (transcript path and session id), the date, the Claude Code version, what went wrong, whether `events.jsonl` is reconstructed, whether the run is gated, and the expected reading of each turn end in order: `done`, `held`, or `waiting` with each outstanding task's kind, id and signal, plus whether every output file exists at that turn end.
Each `transcript_path` in `events.jsonl` is rewritten to the case's own `transcript.jsonl` at load time.

The first cases, sourced from real transcripts under `~/.claude/projects/`, each case's `case.yaml` naming its source run:
the two #411 absorbed-notification runs, each gated, whose Stop with `background_tasks: []` reads `done` — `issue411-prune-rest` from the `test-suite-prune-rest` transcript (launch `toolu_0156x6kuzhRxYK53hcQhn2Lk`, task `bn68egcp9`), `issue411-bugfix-sessions` from the `bugfix-sessions` transcript (launch `toolu_01CpWHu7fKM8iH7NoWVUBVPD`, task `b5o9v6f2u`), both paths in the decision record's Notes;
the `bugfix-sessions` Plan-Write ask answered in its own session, reading `held` and then `done`;
a Webster Master turn end with a fork still running, reading `waiting` with the fork, and, where no Master run with a running fork survives on disk, taken from the hub orch run `6c8e3b9253ddaa0334c35176af6225eb`, whose events file carries Stop payloads listing running subagents, with `case.yaml` saying so;
a fork killed by an API error, reading `done` or `held` (never `waiting`) because the payload no longer lists the fork as `running`;
a skill-load turn end, which `Start` consumes, so the first turn end the run reads is the one after the prompt, taken from the first turn end of the hub orch run `6c8e3b9253ddaa0334c35176af6225eb` (its `promptOffset` is 649);
a session `continued-in` another, taken from the `test-suite-prune-rest` transcript `97d818aa-8fb0-4103-9b56-0ae4da2a75b7`, which ends with a `continued-in` entry naming session `8e5a1d83-f2e5-4809-a7ea-8b056c5c6258`: the Stop lines written after the switch carry the continuing session's transcript path and read as the run's own turn ends, never `waiting` on a launch recorded in the predecessor's transcript.
Each case's expected reading is the one the fixed code must give for what really happened in that run, stated beside what the old code read, and each case adds a reading that `background_test.go`'s synthetic rows and `skillload_test.go` do not: the real line shapes, replayed through `Wait`.
A case whose real run reads differently from the line above is corrected in `case.yaml` and reported to the parent, never synthesized to fit.

`corpus_test.go` is one table test over every directory under `testdata/corpus/`, so a new case directory joins without a code change, and a case whose reading differs fails naming the case and the turn end.
Per case it builds a real `shuttleengine.Runner` over `shuttlefake.Reed` and an engine whose `ParseEvents` is the real `Claude.ParseEvents` (the rest from `shuttlefake.Engine`), with an injected `Clock` and a recording notifier set through `SetNotifier`;
a case that names `skills` in `case.yaml` also runs the real `Claude.SkillLoadMessage` and `Claude.ClassifySkillLoad` through the fake's overrides, so the skill-load turn end is classified as `Start` would.
It starts the run (gated with a passing gate entry when the case says gated), then lets the injected clock's `Sleep` append the next turn end to the run's events file, creating the output files when the case says they exist at that turn end, so each poll tick reads exactly one turn end.
It asserts, per turn end: `done` finalizes Done on the tick that reads it; `held` calls the notifier exactly once and does not finalize; `waiting` neither notifies nor finalizes, and `ParseEvents` on that line reports exactly the listed tasks.
It spawns nothing and reads no real clock (`PATTERN-test-speed`); fixtures are files under `testdata`, so `t.Parallel` holds.

Add a short corpus section to `claudeengine/doc.go`: where the corpus lives, what a case holds, and that promoting a failing live scenario (card 16) is a copy and a trim.

**ImpactSummary:** Adds test data and one untagged test, and one doc section; no production code changes.
