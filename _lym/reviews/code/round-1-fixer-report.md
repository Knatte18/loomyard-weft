# Burler fixer-report — code review round 1

Every finding is fixed, F2 by the parent's ruling.
`lym gate test` is green at tiers 0 and 1 for `internal/orchengine`, `internal/orchcli`, `internal/shuttleengine` and `internal/reedengine`.
The orchcli commits cover several findings each, since F2 to F8 share `remotecontrol.go` and `start.go`.

## Changes

- **F1** (`95ad87c1`): `slashPasteEngine` gains `RemoteControlSequence`, and both paste-path wrappers embed `RemoteController`.
  `TestRunner_TurnOnRemoteControl_PastesTheNamedCommandThroughTheGuardedSend` runs `checkPastedSessionCommand` with `/remote-control <name>`.
  It also covers the multi-line refusal, an engine without the capability typing nothing, and the told-path error.
  `dispatchEngine` gains `RemoteControlSequence`, and `dispatchOps` a `Runner.TurnOnRemoteControl` row.
- **F2** (ruling, see Rulings): `turnOnRemoteControl` refuses while the record reads on.
- **F3**: `TestStart_LaunchPassesAndRecordsRemoteControl` gains rows for an unlisted strand, a failing machine name and a resumed session.
  - The unit harness hub is no git repository, so the unlisted row pins the fall-back to a launch without remote control.
  - The formed fixed name is pinned by the tier-1 `TestLaunchRemoteControlName_UnlistedStrandTakesTheFixedStrandName` in `remotecontrol_integration_test.go`.
  - The rows also assert the spec's `ResumeSessionID` and the RC button's state word.
- **F4**: `TestRemoteControlBarWordAt` gains an `UnreadableRecord` row.
  `TestRemoteControlBarVerb_PrintsNothingForNoWorktree` runs the orch command tree with `rc-bar` and with `rc-bar --dir <tmp>`, and expects an empty line and exit 0.
- **F5**: `turnOnRemoteControl` refuses several orch strands, naming their guids and `lym reed remove`.
  A `TwoOrchStrands` row pins it.
- **F6**: fixed differently, see below.
- **F7**: fixed differently, see below.
- **F8**: `recordRemoteControl` writes nothing for an engine with no remote control.
  - New `orchengine.ReadRemoteControl` tells a missing record from an off one, and `RemoteControlOn` is built on it.
  - `remoteControlBarWordAt` returns no word when no record names the strand, and `rc-off` when the record cannot be read.
  - The `NoRecord` and `RecordedOnForAnOldStrand` rows expect an empty word.
  - The `rc-bar` and `rc-on` help and the orchcli doc are updated.
- **F9**: the orch group's Long text exempts the hidden `rc-bar` from the prime-or-beam rule.
- **F10**: `//testtiming:keep` marks above both named tests.

### Fixed differently

- **F6**: the proposed carry of the old strand's record to the adopted strand is not made.
  - The `fix-proposal` fork built the case against it.
    The state names a strand recorded on and a hand-started live orch is adopted.
    The carry lights the button for a session that never had remote control, and the F2 refusal then blocks the click.
  - A launch by lym already records its own strand, so the proposal's premise holds only when nothing is known of the adopted strand.
  - Own change: `recordUnrecordedRemoteControl` records the live strand off when the record does not name it.
    It runs in `startOrch`'s spawn-watcher and already-running branches.
    The button stays drawn and a click can turn remote control on.
  - Rows cover no carry from the old strand's record, and a record that already names the adopted strand staying lit.
- **F7**: the proposed plain success after a failed record write is not made as such.
  - The `fix-proposal` fork showed a plain success hides the failure, and a second click would queue the command again.
  - Own change: the click still succeeds and logs at Warn, and it adds a `warning` field to the success fields.
    The `FailedRecordWriteStillSucceeds` row asserts the field.

### Gate repair

The gate refused wrapped and over-long comment lines the round added.
They are re-broken one sentence per line in `internal/orchcli/doc.go`, `remotecontrol.go`, `remotecontrol_test.go`, `remotecontrol_integration_test.go` and `internal/shuttleengine/session_test.go`.
No behaviour changed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal, MEDIUM, F8 button lost for an orch with no record: fixed with an own change.
  `recordUnrecordedRemoteControl` records such a live strand off when a start finds it running, so the button is drawn again.
  An orch that no start touches stays without a button until the next `lym beam start`.
- fix-proposal, MEDIUM, F6 carry of an "on" record onto an adopted strand: fixed with an own change, as under F6 above.
- fix-proposal, LOW, F7 hidden failure and a repeated send: fixed with an own change, as under F7 above.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

- **F2**: a ruling from `ly:LOOM`, "Added after Discussion (parent, 2026-11-16): A click on a lit RC button sends nothing".
  - It asks `turnOnRemoteControl` to refuse while `orchengine.RemoteControlOn` reads on for the live strand and send nothing.
    The refusal's way forward names `/remote-control` typed by hand in the orch pane.
  - Done.
    The refusal comes after the several-strand and no-live-strand checks.
    The `AlreadyOn` row asserts it, with nothing injected and the record still on.
    The `rc-on` help, `orchcli/doc.go` and the function comment state it.
  - The ruling was neither mistaken nor redundant, so no flag went to the parent.
