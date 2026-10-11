# Webster round 1 — fixer report

All three findings were validated against the code and the plan and fixed; none disputed, none deferred.
`lyx gate test ./internal/shuttleengine/...` passes after each fix, and `lyx loom lint-comments` is clean.

## Changes

### F1 — fixed as proposed (`8aa3a2159`)

Premise holds: the plan's Shared Decisions say an engine without `SessionSignalParser` behaves exactly as today on every path the plan touches, and `confirmPasted` set `awaitsLanding` from `sc.landingStart` alone, so a parser-less engine waited out the whole submit window on `/clear` and `/compact`.
Card 6's "falls back to the box read at its end" stays true for an engine that parses signals but records no session start; the Shared Decision governs the parser-less engine.

- `internal/shuttleengine/send.go`: `awaitsLanding` now also needs `sc.engine` to implement `SessionSignalParser`; `confirmPasted`'s doc says so.
- `internal/shuttleengine/doc.go`: the `ClearSession`/`CompactSession` paragraph says a parser-less engine confirms on the empty box at once.
- `internal/shuttleengine/session_test.go`: `checkLandingStartConfirmation` gains the regression row "an engine that parses no session signals confirms on the empty box at once", over a new `parserlessPasteEngine` that exposes only `Engine`, `TextPaster`, `InputBoxReader` and `SessionCycler`; `newSlashPasteRunnerAppending` takes a `parserless` flag.
  The row failed before the fix (waited 30s) and passes after it.
  The existing "no session start confirms on the empty box once the window closes" row stays, as the review asks, for an engine that parses signals and records none.

### F2 — fixed as proposed (`23d03a0f8`)

Premise holds: every row appended its session-start line on the paste, so none had a matching line before the pre-send offset.
`checkLandingStartConfirmation` gains the row "a session start of the matching source before the send is waited out", which appends the landing source's line to the events file before the call, with no paste line, and expects the full-window wait.
It runs under both `TestRunner_ClearSession_PlaysScriptedSequence` and `TestRunner_CompactSession_PlaysSequenceForFocus`.

### F3 — fixed as proposed (`6d063e51f`)

Premise holds: every fork row fit in one `initialReadChunk` window, so the unsettled path that widens the window never ran.
`TestTurnEndActivity`'s fork rows gain "a pending call in a batch entry before the first read's window reads now": a pending `batchFirst`, then a `msg_batch` entry longer than `initialReadChunk`, then the answered `batchLast`.
A temporary mutation making the read stop after its first window failed exactly this row; the mutation was reverted before the commit.

No new top-level test was added, so `cmd/lyx/testdata/test-budget.yaml` is unchanged.

## Disputed

None.

## Deferred

None.
