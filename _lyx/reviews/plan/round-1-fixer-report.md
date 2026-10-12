# Plan round 1 — fixer report

Every finding's premise held against the code; all seven are fixed.
`lyx loom validate-plan` passes on the result.

## Changes

- **F1** (card 7): fixed as proposed, and extended.
  `TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv` and `TestAttachArgv_PreflightOnAKnownGoodSession` join `**Edit:**`, each with a Tests bullet carrying the proposed assertion changes.
  The same premise also breaks `internal/reedengine#TestPinGeometryOptionsLocked`, which calls `pinGeometryOptionsLocked` directly (`windowsize_test.go:316-345` asserts no `set-hook` call with the watchdog on, and `windowsize_test.go:396-410` wants exactly one `set-hook` with it off).
  The review did not name it.
  It joins `**Edit:**` too, with a Tests bullet: `WatchdogOnPinsGeometryOptionsOnly` asserts no resize-hook call and one unseen-clear call, and `SetHookErrorIsNonFatalWhenWatchdogOff` wants two `set-hook` calls.
- **F2** (card 1): fixed as proposed.
  `TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv` joins `**Edit:**`, its row becomes `PlanError_TopAnchorWithoutContentSizeRejected` with a top/positional display (`render/policy.go:24-25` still refuses it), and the ImpactSummary names the reedengine row.
- **F3** (cards 4 and 5): fixed as proposed.
  Card 4 states the relaunch kill as a deliberate extension of D6, bounds it to the strand's own recorded pane, and runs `selectStrandsWindowIfCurrentLocked` before the kill when that pane is its window's last; `selectStrandsWindowIfCurrentLocked` joins `**Uses:**`.
  Card 5 re-lists the session's panes through `tmux.listWindowPanes` before `groupLiveCount`, adds that method to `**Uses:**`, and adds the `RelaunchOfADeadGroupMemberRejoinsItsWindow` subtest.
- **F4** (card 4): fixed as proposed, as a subtest of `TestAddStrandLocked` on its own engine and fake tmux, since the test's existing rejected rows run without a fake.
  It is a subtest, so the test budget is unchanged.
- **F5** (card 12): fixed as proposed, with a Bound sentence after the fallback explanation.
- **F6** (cards 1 and 7): fixed as proposed.
  Card 1's three `types.go` glyphs become `internal/reedengine/render/types.go`, and card 7's `statusFormatButtons` becomes `internal/reedengine/bar.go`.
- **F7** (card 13): fixed as proposed.
  The card names the second verb list by its bullet, the one giving `reed`, `shuttle` and `loom` each a CLI, not by line number, so the reference holds as the file changes.

## Disputed

None.

## Deferred

None.
