# Plan round 3 — fixer report

## Changes

- **F1** (fixed as proposed): `_lyx/plan/04-shuttle-conditioned-send.md`.
  Premise confirmed: `clearableBoxEngine.ParseSessionSignals` (`internal/shuttleengine/run_test.go:1373`) parses only `START`, `STOP:` and `SESSIONSTART`, and `pasteEngine` embeds it through `tuiEngine`.
  Added the Edit target `internal/shuttleengine#clearableBoxEngine.ParseSessionSignals`.
  The Intent now states that the parser gains the case `trimmed == "APIERR"`, mapped to `SessionSignalAPIErrorTurnEnd`, and why the pending-record row needs it.
  That row now asserts the line goes out through `PasteSubmit`, not typed, so it pins that the paste path is taken.
- **F2** (fixed as proposed): `_lyx/plan/03-github-per-request-deadlines.md`.
  The held-page row now uses a 400 ms per-request budget, a 250 ms first-page handler (150 ms of slack for page 1), the same 150 ms gate `Sleep` and a 250 ms page-2 handler.
  The derived sentences now say that a fresh deadline leaves page 2 150 ms of margin and the shared deadline fails it 100 ms short.

`lyx loom validate-plan` passes after both edits, with informational `create-new-unit` findings only.

## Disputed

None.

## Deferred

None.
