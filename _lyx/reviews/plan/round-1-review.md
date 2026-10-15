---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/06-follow-bindings-button.md
    summary: Card 6's @lyx_follow_command pin adds one set-option call to every geometry pin, which breaks TestAttachArgv_PreflightOnAKnownGoodSession and two TestPinGeometryOptionsLocked subtests that card 6 neither lists nor names
    fix: In card 6, add the Edit bullet `internal/reedengine#TestAttachArgv_PreflightOnAKnownGoodSession` directly after `internal/reedengine#TestPinGeometryOptionsLocked` (checked with validate-plan --overlay, no new finding). In the Intent's Tests paragraph, say that TestAttachArgv_PreflightOnAKnownGoodSession's wantSetOptionCalls rises from 11 to 12 plus the shell pins, for the one set-option carrying @lyx_follow_command (set or unset) that the attach pre-flight's geometry pins now issue. Extend the TestPinGeometryOptionsLocked list with OneOptionFailureDoesNotStopTheRest (wantCalls 8 becomes 9) and WatchdogOnPinsGeometryOptionsOnly (8 set-option calls become 9, with its message naming the follow-command pin).
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/02-attach-follow.md
    summary: Card 2 leaves the failed client listing unaddressed; warnMismatchedClientsLocked reports another client on a listing failure, so with follow on the selection is now kept, while AttachArgv's doc comment, which card 2 does not rewrite, still says a listing failure drops it
    fix: In card 2's Intent, state that a failed client listing counts as another attached client, so the follow read decides it (follow on selects, follow off or a failed follow read drops the selection). Say the card also rewrites AttachArgv's doc comment at internal/reedengine/attach.go:92-93 ("A second attach never moves a viewer, the selection is dropped when another client is attached, when the client listing fails" and "list the clients drops it too") to the follow rule, alongside the inline comment it already rewrites. Add a TestAttachArgv_SelectsTheRelevantWindow row, FailedClientListingWithFollowOnSelectsTheRelevantWindow, whose script answers list-clients with an error and the @lyx_follow format read with on, wanting @2.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/03-follow-run-op.md
    summary: TestFollow_RealTmux attaches a second and a third client with waitForClientAttached, which returns as soon as any client is attached, so reading the newer client's window races its attach
    fix: In card 3's Intent, replace the waitForClientAttached wording for the second and third attach with a wait, through waitUntil, until list-clients -t =<session> reports the expected client count. Name each client from that listing's client_name format before reading its window_id. The first attach may keep waitForClientAttached.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/06-follow-bindings-button.md
    summary: The real-tmux row labelled D7's fallback unsets @lyx_follow_command under a binding pinned with the jump, a state pinBindingsLocked never produces; the fallback pins the click without run-shell, so the row never reaches it
    fix: In card 6's TestBindings_StatusBarClicksSelectAndZoomByRangeType sentence, replace the "@lyx_follow_command unset on the server" step with re-issuing the MouseDown1Status argv from bindingArgvs(prev, next, false) on the fixture's server, follow set off. Then a follow click sets follow on, keeps the current window and runs no recorder, which is D7's fallback as pinBindingsLocked pins it. Or keep the step and drop the "(D7's fallback)" label, leaving the fallback to the Tier-1 rows in TestBindingArgvs and TestPinGeometryOptionsLocked.
---

### [BLOCKING] Card 6 misses three tests its new pin breaks

**Location:** `_lyx/plan/06-follow-bindings-button.md`; the broken assertions are at `internal/reedengine/attach_test.go:291`, `internal/reedengine/windowsize_test.go:313-317` and `internal/reedengine/windowsize_test.go:341`.

**Issue:** `pinBindingsLocked` runs from `pinGeometryOptionsLocked` (`windowsize.go:183`), which boot and the attach pre-flight (`attach.go:167`) both call.
Card 6 makes it issue `set-option -g @lyx_follow_command <line>` or `set-option -gu @lyx_follow_command` on every pin, so every exact count of the pins' `set-option` calls goes up by one.
Card 6 names `AllOptionsIssued`, `ShellPinsFollowTheOptionPins`, `SetHookErrorIsNonFatalWhenWatchdogOff` and `BindingsAreIssuedAfterTheOptionPins`.
It misses three exact counts:

- `OneOptionFailureDoesNotStopTheRest` (`const wantCalls = 8`).
- `WatchdogOnPinsGeometryOptionsOnly` (`setOptionCalls != 8`).
- `TestAttachArgv_PreflightOnAKnownGoodSession` (`wantSetOptionCalls := 11 + wantShellPins`), which runs the same pins through `AttachArgv`.

The last is a test member no card lists, so the card has no target covering a test its own change breaks.
The proposed target addition passes `lyx loom validate-plan --overlay`, which reports no finding the `{}` baseline lacks.

### [MEDIUM] Card 2 leaves the failed client listing undecided and AttachArgv's doc stale

**Location:** `_lyx/plan/02-attach-follow.md`; `internal/reedengine/attach.go:92-93` and `:288-293`.

**Issue:** `warnMismatchedClientsLocked` returns `anotherClient = true` when `list-clients` fails.
Under card 2's gate, "resolved when no other client is attached or when `followingLocked` reads on", a failed listing with follow on now selects the relevant window.
Today it drops the selection.
D4 does not settle the listing-failure case, and card 2's Intent never mentions it.
`AttachArgv`'s doc comment says "the selection is dropped when another client is attached, when the client listing fails", and "list the clients drops it too".
Card 2 rewrites only the inline comment at `attach.go:131`, so the doc comment ends up contradicting the code.
No row pins the new behavior: `TestAttachArgv_MultiClientWarning`'s failing-listing case scripts no own window, so its argv does not change either way.

### [LOW] TestFollow_RealTmux's later attaches are not waited for

**Location:** `_lyx/plan/03-follow-run-op.md`; `internal/reedengine/attachgeometry_integration_test.go:156-162`.

**Issue:** `waitForClientAttached` polls until `list-clients` is non-empty.
Once the first client is attached it returns immediately for the second and third.
`RunStartSelectsTheOverviewThenEachNewAgentForEveryClient` and `ASecondAttachMovesTheFirstClientOnlyWhileFollowIsOn` then read each client's window with `display-message -c <client>`, which can run before the newer client exists.
That makes the test timing-dependent, which `PATTERN-test-speed` and the testing rules forbid.

### [LOW] Card 6's live fallback row tests a state the code never produces

**Location:** `_lyx/plan/06-follow-bindings-button.md`, the `TestBindings_StatusBarClicksSelectAndZoomByRangeType` sentence.

**Issue:** `pinBindingsLocked` writes the option and the binding together.
It sets `@lyx_follow_command` and binds the click with the jump, or it unsets the option and binds the click without `run-shell`.
The row unsets the option under a binding pinned with the jump, so the click runs `run-shell -b ""`.
No pin produces that pairing, so the row never reaches D7's fallback binding, which it is labelled as.

## Rejected

- Card order (focus 2): `driverWindows`' only caller is `newAgentWindowSelected` (`ownwindow.go:338`), and no test references it.
  Both re-signed members' callers (`spawn.go:344`, `ownwindow_test.go:116`) sit on card 1.
  Cards 2–6 use only what card 1, card 3 or card 5 creates, and `followTargetWindow` is self-contained on card 5. Not a defect.
- Re-sign arrows and member glyphs (focus 4): `newAgentWindowSelected`, `newAgentWindowTakesViewerLocked`, `bindingArgvs` and `switchBindingCommandsLocked` carry arrows.
  Every other changed symbol is listed as a member glyph. Not a defect.
- Attack surface and refusal reachability (focus 3): cards 3, 4, 5 and 6 each state their bound.
  Three tests reach the unknown-session envelope: `TestFollowSessionVia`'s failing `list-panes` row, `TestRunCLI_ColdHub`'s new subtest and `TestFollow_RealTmux`.
  Card 1's row and card 2's `AnotherClientWithAFailedFollowReadAppendsNothing` reach failed-read-counts-as-off.
  Card 5's space-and-`#` row and card 6's live click row reach the quoting of a session name.
  `validateFollowFlags` rejects only empty values, so no upstream check rejects these inputs.
- Test economy and budget (focus 5): `internal/reedengine` holds 266 top-level tests against a 266 row, and cards 3 and 5 each raise it by one for their one new test.
  No other card adds a top-level test.
- Verify coverage (focus 6): the section covers reedengine, reedcli, loomcli and cmd/lyx, runs `-tags integration` and compiles `tmux` and `llm`, in the stencil's order.
  `pattern/PATTERN-pane-binary-resolution.md` is read only by `internal/pattern`'s orphan check, which a content edit cannot fail.
- CLI shape (focus 7): `lyx reed follow` is a reedcli leaf with `Use: "follow"`, flags checked by a validator rather than cobra's required mark (as `switch` does), `AudienceRole` and an `output.Err` envelope.
  The audience bullet allows that.
- Fidelity (focus 1): D1–D9 and every Constraint trace to a card.
  D6 is covered by card 1's readbacks and doc rewrite, D9 by cards 1 and 6, the background-file list by card 6, and the reedengine, reedcli and loomcli doc rewrites by cards 1–6, 5 and 4.
  No card adds scope the record does not license.
