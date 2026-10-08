# Card 6 — state-reducer

**Create:**
- `internal/shuttleengine/sessionstate.go`
- `internal/shuttleengine/sessionstate_test.go`
- `internal/shuttleengine#SessionState`
- `internal/shuttleengine#SessionFacts`
- `internal/shuttleengine#SessionFold`
- `internal/shuttleengine#SessionFold.Fold`
- `internal/shuttleengine#Liveness`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#SessionSignal`
- `internal/shuttleengine#SessionSignalKind`

**Intent:**
Add the provider-neutral session-state reducer (D3) as a pure fold in `sessionstate.go`; nothing calls it yet.

Types:
- `SessionStateName` with `busy`, `idle-done`, `idle-stalled`, `asking`, `dead`, `unknown`.
- `SessionState`: name, cause, detail (an API error's text), the outstanding tasks of a `busy` `background`, and `Since`, zero when the causing line has no time.
- `Liveness`: unproven (the zero value), alive, dead.
- `SessionFacts`, what the caller read for this fold: `Interactive`; `OutputsExist`; `Liveness`; `EventsUnreadable`; the transcript API-error marker of the fold's newest turn end, with its text; the interrupt marker of the fold's newest turn start, with the transcript entry's time; and `ReadAt`, the reading's time from the caller's clock.
- `SessionFold`, the fold's memory: the newest signal, the newest turn end's judgment, the current state and the history.
  `Fold(signals, facts)` folds new signals in file order, then settles the current state; `State()` and `History()` read it.
  A files-only reader makes a zero fold and folds every signal past the prompt offset once; the wait loop keeps one fold and calls `Fold` at every poll, with an empty slice when nothing new arrived.

Rules, exactly as D3 states them:
- Signal-derived states: a turn start is `busy` `turn`, and a second one records nothing; a turn end with outstanding tasks is `busy` `background` with its tasks; a turn end with none is `idle-done` when `OutputsExist`, else `idle-stalled` `no-output` in an autonomous run and `asking` in an interactive one; an API-error turn end is `idle-stalled` `api-error` with its text, and so is the newest turn end when the transcript marker says so; an ask is `asking`; an idle notice changes nothing unless the newest signal is a turn start, which is `unknown` `contradiction`; a `clear` or `resume` session end is `unknown` with that reason until the next signal; the interrupt marker turns the newest turn start's `busy` into `idle-stalled` `interrupt`.
- The output check is applied to every turn end of one fold with that fold's `OutputsExist`; at a fold with no new turn end, only the newest turn end is re-judged, and only when it reads `no-output` or interactive `asking`, so outputs that appeared since make it `idle-done`, never a negative state.
- Precedence, first match wins: a process-ending session end as the newest signal is `dead` with its reason; `Liveness` dead is `dead` `process-gone`; `Liveness` unproven is `unknown` `liveness-unproven`; `EventsUnreadable`, then no signal at all, is `unknown` with that cause; then the signal-derived state.
- History records an entry only when the state or the cause changes, in signal order.
  `Since` is the causing signal's `At` for a turn start, a waiting turn end, an API-error turn end, an ask, a session end and the contradiction's idle notice; the turn end's `At` for `idle-done`, `no-output`, interactive `asking` and a transcript-marked `api-error`; the transcript entry's time for `interrupt`; `ReadAt` for `dead` `process-gone`, `unknown` `liveness-unproven`, `unknown` with no signal or an unreadable file, and for `idle-done` reached by re-judging at a later fold with no new signal.
- Signals are ordered by file order only; `At` is read for `Since` alone, never compared.

`SessionFold`'s doc comment carries the handover the added decision "Output files first covers output-derived states only" requires: the output files are consulted before the output-derived states (`no-output`, interactive `asking`), while `dead` and `unknown` outrank `idle-done`, so a finished run can read `dead`, and the first consumer that acts on `dead` applies `PATTERN-completion-signal`'s output check before acting.

Tests, untagged, in one table-driven top-level function in `sessionstate_test.go`, over hand-built `SessionSignal` slices and facts with a fixed `ReadAt`, one row per acceptance reading of the reducer: every state and cause listed under the acceptance criteria's reducer row, D3's precedence rows, the history row (one entry per change, none for a repeated signal, each `Since` source), a two-fold row where outputs appear at the second fold with no new signal (`idle-done` stamped `ReadAt`) against a one-fold row over the same signals (`idle-done` stamped with the turn end's `At`), the transcript-marker rows (only the newest turn end or turn start of a fold is marked), and the interrupt row where a turn start's later or missing `At` changes nothing.
`cmd/lyx/testdata/test-budget.yaml`: the `internal/shuttleengine` row rises by 1, for that function.

**ImpactSummary:** Adds a pure reducer no code calls yet; the only existing file touched is the test budget row.
