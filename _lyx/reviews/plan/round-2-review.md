---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/plan/04-reed-own-window-launch.md (Intent, launchStrandLocked bullets)
    summary: Card 4 kills a relaunching strand's dead own-window pane, which D6 and D5 forbid, and justifies it with a premise card 3 contradicts
    fix: In card 4's launchStrandLocked bullets, either (a) drop the kill of the relaunching strand's recorded dead pane and its select-window step, so the old dead pane is left unbound and card 3's reconcile ends its window once none of its panes is bound; then update card 5's re-list sentence (no kill to follow) and its RelaunchOfADeadGroupMemberRejoinsItsWindow subtest to the resulting arrangement, or (b) keep the kill only once the operator has amended D6 and D5 in the decision record to allow a kill on relaunch, and replace the card's self-licensed deliberate-extension sentence with a citation of that amended decision. In either case, delete the claim that the dead pane would never be killed or ended.
---

### [MEDIUM] Card 4's relaunch kill departs from D6's own-window kill rule without a license

**Location:** `_lyx/plan/04-reed-own-window-launch.md`, Intent, the `launchStrandLocked` bullets (the "When `s.PaneID` names a listed pane …" bullet and the "deliberate extension of D6" sentence after it).

**Issue:** D6 says: "In own windows reed kills a pane only when it is a recorded strand's pane being removed".
D5 says that a dead own-window pane "keeps its strand binding and is killed only by the strand's removal".
D5's acceptance criterion repeats it: the dead pane stays "until its strand is removed, including across later reed ops in the worktree".
Card 4 kills the pane on a relaunch, not on a removal, and calls this "a deliberate extension of D6".
The relaunch is reachable in a live session, not only after a server restart.
`planResumeLaunches` (`internal/reedengine/lifecycle.go:175`) relaunches every non-alive, non-hidden strand whose done-when paths are not all present, and `AddStrandUnless`'s `ifAbsentRelaunch` branch (`internal/reedengine/strand.go:535`) does the same by name.
So a `lyx reed resume` or an `--if-absent` add kills a dead own-window pane that the decision record says stays until removal.
The rubric admits no card scope the answer key does not license, and a card cannot grant its own license.
The wording comes from round 1's F3 fix, which round 1's reviewer proposed and its fixer applied as proposed.
A review round cannot license scope either: only the decision record can, and it still forbids the kill.

The card's justification, carried over from round 1's F3, also rests on a false premise.
It says that without the kill "a dead pane left in a shared window would never be killed or ended".
Card 3's `planReconcile` lists in `ownWindowsToEnd` every window that carries `@lyx_own` and none of whose panes is bound.
After the relaunch rebinds the strand to its new pane, the old pane is unbound, so its window ends at the first reconcile after the group's last bound pane in it goes.
The dead pane outlives the relaunch only while the group's other strands stay in that window.

On the focus questions this finding touches:
the select before the kill does keep D5's viewer bound, since it runs only when the dead pane is the last listed pane of its window;
and card 5 places from a listing taken after the kill (its re-list sentence orders `tmux.listWindowPanes` after card 4's kill and after the head reconcile).
The defect is the kill's license, not its ordering.

## Rejected

- Focus 1, the `session-window-changed` pin in `pinGeometryOptionsLocked`: not a defect.
  `AttachArgv` (`internal/reedengine/attach.go:137`) calls the pin before every suppressing guard except the non-positive size and `requireSessionLocked`.
  So the degraded rows `FewerThanTwoLivePanes`, `NoStrandOwnsAPresentPane`, `ListPanesErrors` and the plan-error row of `TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv` would issue the new hook and trip its `mustNotCall("set-hook")`.
  `TestAttachArgv_PreflightOnAKnownGoodSession` would see the new hook as its first `set-hook`, before `list-panes`, and fail its index and `-u` checks.
  Card 7 lists both tests under `**Edit:**` and states the change to each: the resize-hook-name filter and the one `session-window-changed` assertion.
  The other tests that assert on this function's `set-hook` calls are on card 7 too: `TestPinBootOptionsLocked_InstallsTheSignalEntry` (`lifecycle_test.go`) and `TestPinGeometryOptionsLocked` (`windowsize_test.go`).
  `TestAttachArgv_ChainCarriesTheAdjustedPinsAfterSelectLayout` asserts with `slices.Contains` on the hook bodies, and `TestPinGeometryOptionsLocked`'s `WatchdogOffUnsetsHookAndRemovesSignalFile` reads the last `set-hook` call.
  With the pin placed after the navigation bindings, as card 7 says, the watchdog unset stays last, so neither test breaks.
- Focus 2, the `PlanError_DeferredAnchorRejected` row: not a defect.
  Card 1 itself rewrites the row to `PlanError_TopAnchorWithoutContentSizeRejected`, with a top-anchored positional strand that `render.ValidateDisplay` still refuses (`render/policy.go:24`).
  `anyPlacedStrand` (`apply.go:133`) counts that strand as placed, so the row still reaches `planLayout`'s error between card 1 and card 3, and the test is on card 1's `**Edit:**`.
  No other `internal/reedengine` test relies on own-window being refused at the render layer.
  `TestAddStrandLocked` and `TestUpdateStrandLocked` rely on reedengine's own `validateDisplay`, which keeps refusing it until card 4 edits both.
- Card 13 does not wire `lyx shuttle open` through `internal/cliwire`, although the decision record's constraint names it.
  This is not a PATTERN violation: `PATTERN-cliwire-sole-wiring` binds standalone-capable CLIs, `internal/shuttlecli` imports no `cliwire`, and the card records the deviation and its reason.
