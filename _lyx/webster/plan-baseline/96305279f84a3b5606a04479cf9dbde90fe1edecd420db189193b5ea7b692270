# Card 16 — corpus-states

**Edit:**
- `internal/shuttleengine/claudeengine/corpus_test.go`
- `internal/shuttleengine/claudeengine/testdata/corpus/fork-killed-api-error/case.yaml`
- `internal/shuttleengine/claudeengine/testdata/corpus/fork-killed-api-error/events.jsonl`
- `internal/shuttleengine/claudeengine/testdata/corpus/issue411-bugfix-sessions/case.yaml`
- `internal/shuttleengine/claudeengine/testdata/corpus/issue411-prune-rest/case.yaml`
- `internal/shuttleengine/claudeengine/testdata/corpus/master-fork-running/case.yaml`
- `internal/shuttleengine/claudeengine/testdata/corpus/planwrite-ask-answered/case.yaml`
- `internal/shuttleengine/claudeengine/testdata/corpus/skill-load-turn-end/case.yaml`

**Uses:**
- `internal/shuttleengine/claudeengine#Claude.ParseSessionSignals`
- `internal/shuttleengine#SessionFold.Fold`

**Intent:**
Extend the replay corpus so every case also states the session state its events reach, with every existing reading unchanged.

`case.yaml` gains a `states` list: one entry per events line the signal parser reads, in file order, each naming the state and cause the fold reads after that line.
`turn_ends` keeps one entry per line `ParseEvents` reads an event from, with today's readings; `events.jsonl` may now hold lines `ParseEvents` skips, and the replay appends every line, one per tick, checking a turn end's reading only on the lines that carry one.
The state check folds the case's lines with the real `ParseSessionSignals` into one `SessionFold`, a line at a time, with facts the replay controls: liveness alive, the run autonomous, outputs present exactly when the case's preceding turn end says so, a fixed reading time, and no transcript marker, since the case's transcript is the whole session's final file and would mark every turn end with its end state.

`fork-killed-api-error` gains its API-error turn end: a reconstructed `StopFailure` line between its two Stop lines, for the Master turn that died on the outage (fields `error`, `error_details`, `last_assistant_message` per Claude Code 2.1.294's schema, its text taken from the transcript's API-error entry), and its `events_note` says so.
Its states read `busy` `background`, then `idle-stalled` `api-error` with the error text, then `busy` `background`.
A case with skills folds nothing from the load turn's line Start consumes, which lies before the prompt offset.
Every other case's states follow from its turn ends: a waiting turn end `busy` `background`, a done one `idle-done`, a held one `idle-stalled` `no-output`.

**ImpactSummary:** The corpus replay additionally checks each case's session states and admits non-event lines; one case gains a reconstructed StopFailure line; every existing reading is unchanged.
