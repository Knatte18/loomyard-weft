# Card 13 — wait-shadow-log

**Create:**
- `internal/shuttleengine/waitshadow.go`
- `internal/shuttleengine/waitshadow_test.go`

**Edit:**
- `internal/shuttleengine/wait.go`
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/fakes_test.go`

**Uses:**
- `plan:internal/shuttleengine#SessionFold`
- `plan:internal/shuttleengine#SessionFold.Fold`
- `internal/shuttleengine/sessionread.go`

**Intent:**
Log the session state beside the wait loop's own classification (D5, first bullet), in shadow mode: the loop never branches on it.

`Run` gains a `SessionFold`, a signal cursor separate from `run.offset`, and the last logged state and disagreement.
`waitshadow.go` holds one method `Wait` calls once per tick after `pollEventsTick` (and once more just before each return of a classified outcome):
- It reads the events file from the signal cursor, parses it with the engine's `SessionSignalParser`, and advances the cursor by the parser's `consumed`, so a trailing stamp is re-read with its payload next tick (the added decision on incremental readers); `run.offset` and `ParseEvents`' input are untouched.
- It folds the new signals with the facts of that tick: the spec's `Interactive`, one `allOutputFilesExist` over the spec's output files, and the liveness and transcript facts card 8's helper reads, refreshed at a tick that folds new signals and at each liveness tick and reused between.
- It logs every state change at Info as `shuttle: session state`, with the state, cause, since and the loop's current classification: `running`, `waiting` (a recorded waiting turn end or a pending gate entry), `held` (the held wait card 10 shows), `done` or `died`.
- It logs `shuttle: session state disagrees` at Warn once per disagreement when the pair is outside the fixed mapping waiting↔`busy`, held↔`idle-stalled` or `asking`, done↔`idle-done`, died↔`dead`, running↔`busy`, naming both sides, and again only after either side has changed.
  Three pairings are expected and log no disagreement: a gate wait with every output present beside `idle-done`; a held turn end after transcript-reported shells expired beside `busy` `background`; `done` beside `busy` `background`, gated or not.
- An engine without the parser, an unreadable events file or a fact read failure logs once at Debug and changes nothing.

Bound: the method returns nothing `Wait` reads; every verdict, notice, wait mark and return stays as before this card.

Tests, untagged, in `waitshadow_test.go`, driving `Wait` with the fake clock and reed and an engine double from `fakes_test.go` that answers signals, liveness and transcript facts, with log capture: a run moving through turn start, waiting turn end, held turn end and done logs one `shuttle: session state` per change with its classification; a state outside the mapping (a `dead` reading while the loop reads running) logs one disagreement across several ticks; each of the three expected pairings logs none; and a table row per scenario asserts the same `Result` and notifier calls as the run without the parser capability.

**ImpactSummary:** The wait loop gains shadow-mode session-state logging on a separate cursor; no wait-loop verdict, notice, mark or return changes.
