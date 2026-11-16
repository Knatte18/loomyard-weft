# Round 2 fixer-report

## Changes

- **F1 (BLOCKING), fixed as proposed**, commit `9029f217`.
  `RemoteControlBarVerb` and `RemoteControlOnVerb` are declared once in `internal/reedengine/bar.go`.
  `composeRemoteControlClickCommand` and `composeRemoteControlStateCommand` in `internal/reedengine/panebin.go` use them.
  `internal/orchcli/remotecontrol.go` sets `remoteControlBarVerb` and `remoteControlOnVerb` from them.
- **F2 (MEDIUM), fixed as proposed**, no commit.
  `_lym/landing/summary.md` is under `_lym`, where git is off limits, so the edit stays in the working tree.
  The "click always sends" bullet now says a click on a lit button refuses and sends nothing, with `/remote-control` typed by hand as the way forward.
  A new bullet says a start adopting an unrecorded running strand records it off.
  The RC-button bullet names the record-names-the-strand condition.
  The Codex Open-risks bullet now says no record is written, so no RC button is drawn.
- **F3 (MEDIUM), fixed as proposed**, commit `2091a367`.
  `TestStart_LiveStrand` in `internal/orchcli/start_test.go` asserts the RC word for every row that reaches the success path.
  A new row, an already-running strand the record names, pins the keep-the-record path as lit.

`lym gate test ./internal/orchcli ./internal/reedengine` passed at tier0 with 0 failed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
