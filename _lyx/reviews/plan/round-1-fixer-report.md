# Plan round 1 — fixer report

Every finding's premise was checked against the code and the decision record, and all six hold.
`lyx loom validate-plan` exits clean after the edits.

## Changes

- **F1 (MEDIUM), fixed as proposed.**
  Card 3's `guardBootSessionLocked` now takes `identityBefore` in place of the `candidate` flag, reads the identity again itself and derives candidacy.
  `ensureServerAndSessionLocked` keeps only the pre-spawn read.
  `TestTmuxConfBootGuard` gains the candidacy rows: differing identities claim, no pre-spawn identity claims, the same server on both sides does not.
  The seen-server-ended criterion now names the differing-identity row.
- **F2 (LOW), fixed as proposed.**
  Card 4's `TestListSessions` asserts the call's argv against a literal per host GOOS and gains literal `linux` and `windows` rows over `listSessionsArgv`, in the shape of `TestServerSpawnArgv`'s `goos` rows.
  `TestSwitchClientVia_IssuesOnlyTheThreeCommands`'s rows are stated to be literals.
- **F3 (LOW), fixed differently.**
  The proposal put the Windows pass-through on F4's surviving `TmuxConfRelative` row.
  A `TestUp_BootValidation` row carries one static `wantErr` or `notErr`, so one row cannot both refuse off Windows and pass on Windows.
  Card 2 instead adds a second row, `TmuxConfUnvalidatedOnWindows`, which skips off Windows and asserts through `notErr` that the boot is not refused for `tmux_conf`, as `ValidWatchdog_On` does.
  Card 3 states that this row and `TestServerSpawnArgv`'s unchanged Windows row pin the criterion, and that the skipped guard call is reachable only through a real psmux boot.
- **F4 (LOW), fixed as proposed.**
  Card 2 keeps `TmuxConfRelative` only, with no fixture file, and drops `TmuxConfMissing` and `TmuxConfGlobChar` along with the shared fixture variable; `TestTmuxConfPath` stays the covering test for the refusal classes.
- **F5 (LOW), fixed as proposed.**
  Card 1 drops the `TestFill` row and `internal/configreg_test#TestFill` from `Edit:`, and names `TestFill`'s `reed dropped top-level key` row plus the `TestLoadConfig_EnvOverride` assertion as the reconcile criterion's coverage.
- **F6 (NIT), fixed as proposed.**
  Card 4's `ImpactSummary` drops the tally and says "every `list-sessions` call in one package".

## Disputed

None.

## Deferred

None.
