# Plan round 1 — fixer report

## Changes

- **F1** (fixed as proposed): card 6 lists `internal/reedengine#TestAttachArgv_PreflightOnAKnownGoodSession` directly after `TestPinGeometryOptionsLocked` under Edit.
  Its Tests paragraph raises that test's `wantSetOptionCalls` from `11 + wantShellPins` to `12 + wantShellPins` for the `@lyx_follow_command` pin, set or unset.
  The `TestPinGeometryOptionsLocked` list gains `OneOptionFailureDoesNotStopTheRest` (`wantCalls` 8 to 9) and `WatchdogOnPinsGeometryOptionsOnly` (9 `set-option` calls, its message naming the follow-command pin).
- **F2** (fixed as proposed): card 2's Intent states that a failed client listing counts as another client, so follow on selects and follow off or a failed follow read drops the selection.
  The card also rewrites `AttachArgv`'s doc comment (`attach.go:92-93`) to that rule, keeping the lock, session and state failures as drops.
  It adds the `TestAttachArgv_SelectsTheRelevantWindow` row `FailedClientListingWithFollowOnSelectsTheRelevantWindow`, with `list-clients` erroring and follow `on`, want `@2`.
- **F3** (fixed as proposed): card 3's `TestFollow_RealTmux` keeps `waitForClientAttached` for the first attach only.
  The second and third attach wait through `waitUntil` until `list-clients -t =<session>` reports the expected count, and each client is named from that listing's `#{client_name}` before its `window_id` is read.
- **F4** (fixed by the fix key's second option): card 6's live `TestBindings_StatusBarClicksSelectAndZoomByRangeType` row drops the unset-`@lyx_follow_command` step and its "(D7's fallback)" label.
  A sentence now names the Tier-1 rows that pin the fallback: `TestBindingArgvs_...` with `followJump` false and `TestPinGeometryOptionsLocked`, whose tmux binary does not resolve.
  I did not take the first option, re-binding `MouseDown1Status` from `bindingArgvs(prev, next, false)` mid-test, because the row's later quoted-session click needs the jump binding, so that option would also need a second re-bind.

`lyx loom validate-plan` reports `ok: true` after the edits.

## Disputed

None.

## Deferred

None.
