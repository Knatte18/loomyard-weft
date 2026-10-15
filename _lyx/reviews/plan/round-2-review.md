---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/02-attach-follow.md (Edit list); internal/reedengine/attach_test.go:549
    summary: Card 2 omits TestAttachArgv_MultiClientWarning, whose ListClientsError subtest fails once a failed listing with follow on selects the relevant window
    fix: "In 02-attach-follow.md add `internal/reedengine#TestAttachArgv_MultiClientWarning` to **Edit:**, and add to the Intent that its subtest `ListClientsError_WarnsAndAppendsNoSelection` scripts `fake.answerFormat(\"#{@lyx_follow}\", \"off\", nil)` so it keeps its no-selection expectation as the failed-listing-with-follow-off row, while the new `FailedClientListingWithFollowOnSelectsTheRelevantWindow` row covers follow on. The Intent also rewrites the test's doc comment sentence \"an argv byte-identical to what the same script produces with no attached clients at all\" to say so only while follow reads off or no window is relevant. Checked with `lyx loom validate-plan --overlay`: no new blocking finding."
  - id: F2
    severity: LOW
    class: consistency
    location: internal/reedengine#Engine.warnMismatchedClientsLocked
    summary: The doc comment of warnMismatchedClientsLocked says the other-client result is what drops the attach window selection, which card 2 makes false, and the card does not list the member
    fix: "In 02-attach-follow.md add `internal/reedengine#Engine.warnMismatchedClientsLocked` to **Edit:** (no re-sign arrow, the signature is unchanged) and add to the Intent that its doc comment drops \"which is what drops the attach window selection from the argv\" for \"which AttachArgv's selection gate reads\". The glyph encloses the cited doc comment, confirmed with `lyx quarry resolve` (span 276-303). Checked together with F1 by `lyx loom validate-plan --overlay`: no new blocking finding."
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/06-follow-bindings-button.md (Tests, the weird-named session click); internal/reedengine/windowsize.go:162
    summary: The live click on a session named with a space, a quote and a hash cannot reach the follow button, since the status line 2 exists only on sessions reed pinned with status 2
    fix: "In 06-follow-bindings-button.md state in the Intent that the test, after `new-session -d -s <name>` and before the `switch-client`, runs `set-option -t =<name>: status 2` on that session. `pinGeometryOptionsLocked` pins `status 2` per session (`set-option -t <target> status 2`, windowsize.go:162) and boot starts the server with `-f /dev/null`, so a plain new session shows one status line, and the `click` helper's row-1 click at y = rows then lands on line 0, which carries no follow range. No target changes, so no overlay run is needed."
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/06-follow-bindings-button.md (D7's fallback paragraph); internal/reedengine/windowsize_test.go:294
    summary: No planned row reaches the fallback click binding through pinBindingsLocked, so a pinBindingsLocked that always passes true to bindingArgvs passes every planned test
    fix: "In 06-follow-bindings-button.md extend the Intent of `TestPinGeometryOptionsLocked`'s `BindingsAreIssuedAfterTheOptionPins` (fixture tmux binary does not resolve, so `follow` is empty): it also asserts the issued `MouseDown1Status` bind-key argv equals the one from `bindingArgvs(\"\", \"\", false)`, so it holds no `run-shell`. The planned bind-key count `len(bindingArgvs(..., false))` is the same for true and false, since the follow branch lives inside the one click argv, and the real-tmux rows all resolve the paths and run with the jump pinned. No target changes, so no overlay run is needed."
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/03-follow-run-op.md (TestFollow_RealTmux subtests 2 and 4)
    summary: The client counts TestFollow_RealTmux waits for cannot be derived, since subtest 2 leaves two clients attached and subtest 4 then names a second and a third attach
    fix: "In 03-follow-run-op.md state which clients exist at each subtest and the count each later attach waits for. Subtests run in order on one session and the clients of `RunStartSelectsTheOverviewThenEachNewAgentForEveryClient` stay attached, so either make `ASecondAttachMovesTheFirstClientOnlyWhileFollowIsOn` attach a third and a fourth client and wait for counts 3 and 4 with the earlier two counted among the clients it asserts on (put every attached client back on the shell first, and name how), or end subtest 2 by detaching its clients so subtest 4 starts from none. Replace \"the second and third wait\" in the Intent with the counts. No target changes, so no overlay run is needed."
---

### [BLOCKING] Card 2 omits TestAttachArgv_MultiClientWarning, whose ListClientsError subtest fails

**Location:** `_lyx/plan/02-attach-follow.md` Edit list; `internal/reedengine/attach_test.go:549`

**Issue:** Card 2's Intent decides the failed-listing case: a failed `list-clients` counts as another client, so the follow read decides, and follow on selects.
The card's ImpactSummary says the other `AttachArgv` tests "attach no other client or script no own window".
`TestAttachArgv_MultiClientWarning/ListClientsError_WarnsAndAppendsNoSelection` does both: it scripts `relevantWindowFixture(false, false)` (live agent windows) and answers `list-clients` with an error, then calls `assertChainedArgv`, which wants the plain chained argv with no selection.
`newAttachTestEngine` answers every unscripted `display-message` with `$0|4321|1700000000` (`attach_test.go:97`), so `followingLocked` reads a value other than `off`, follow reads on, and `AttachArgv` appends `; select-window -t @2`.
The subtest then fails on the untagged `go test` run in `## verify:`, under a card that does not list it.
The test's doc comment, "an argv byte-identical to what the same script produces with no attached clients at all", also stops holding.
Round 1 saw this subtest as scripting no own window; `attach_test.go:550` shows it scripts `relevantWindowFixture`, so the round 1 fix for its F2 left it broken.

### [LOW] The doc comment of warnMismatchedClientsLocked goes stale

**Location:** `internal/reedengine#Engine.warnMismatchedClientsLocked`

**Issue:** Its doc comment says the reported "another client" result "is what drops the attach window selection from the argv".
After card 2 the result only gates the follow read, and a client alone drops the selection only while follow reads off or its read fails.
The card lists `Engine.AttachArgv` but not this member, so the comment is left wrong.

### [MEDIUM] The live click on a weird-named session cannot reach the follow button

**Location:** `_lyx/plan/06-follow-bindings-button.md` Tests; `internal/reedengine/windowsize.go:162`

**Issue:** The plan's last live row creates a session named with a space, a quote and a `#` with `new-session -d -s <name>`, moves the client there with `switch-client`, sets follow off and clicks the follow button on line 2.
`pinGeometryOptionsLocked` pins `status 2` on the reed session only (`set-option -t <target> status 2`), and the server boots with `-f /dev/null`, so the new session shows tmux's default single status line.
`status-format[1]` is global, so the `click` helper still reads the follow label's text from it, but its click at `y = rows` lands on the one status line, which renders `status-format[0]` and carries no `follow` range.
No recorder runs, and the row times out.
`pinBindingsLocked` produces the bindings, which are server-global, but not the second status row the click needs.
The first two clicks of the follow-range row, in the reed session, are reachable.

### [MEDIUM] No row reaches the fallback binding through pinBindingsLocked

**Location:** `_lyx/plan/06-follow-bindings-button.md` D7's fallback paragraph; `internal/reedengine/windowsize_test.go:294`

**Issue:** The card says the fallback, "the option is unset and the click is bound without `run-shell`", is pinned by the Tier-1 rows.
`TestBindingArgvs_...` with `followJump` false reaches `bindingArgvs`, but not `pinBindingsLocked`'s choice of that value.
`TestPinGeometryOptionsLocked` reaches `pinBindingsLocked` with an unresolved tmux binary, but the planned `BindingsAreIssuedAfterTheOptionPins` only counts `bind-key` calls against `len(bindingArgvs("", "", false))`, and that count is identical for true and false.
The real-tmux rows resolve every path and run with the jump pinned.
A `pinBindingsLocked` that passes `true` unconditionally therefore passes every planned test, and D7's "a click from off only sets follow on" has no test that binds it.

### [MEDIUM] The client counts the real-tmux follow test waits for cannot be derived

**Location:** `_lyx/plan/03-follow-run-op.md` TestFollow_RealTmux

**Issue:** The Intent says the first attach waits with `waitForClientAttached` and "the second and third wait through `waitUntil` until `list-clients` reports the expected client count".
That does answer the wait-for-any-client race, but the expected counts are not derivable from the subtests.
Subtests run in order on one session; subtest 2 attaches two clients and leaves them attached, and subtest 4 then speaks of "a second client's attach" landing "both" on the overview and of "a third attach", which fits a session that holds one client, not two.
Taken literally the test attaches four clients, not three, so "second and third" and the count each `waitUntil` waits for are undetermined, and a wrong count either times out or returns early.

## Rejected

- Focus 1 (card 6's exact `set-option` counts): settled, no defect.
  `TestPinGeometryOptionsLocked`'s `AllOptionsIssued`, `ShellPinsFollowTheOptionPins`, `OneOptionFailureDoesNotStopTheRest`, `WatchdogOnPinsGeometryOptionsOnly` and `SetHookErrorIsNonFatalWhenWatchdogOff`, and `TestAttachArgv_PreflightOnAKnownGoodSession`, are the only tests in `internal/reedengine` that assert an exact count or the exact list of the pins' `set-option` calls, and the card lists each with its new count (9, and 12 plus the shell pins).
  The unset is the last `set-option` of the pins, since `pinBindingsLocked` runs after the shell pins.
- Focus 2, whether the Intent decides the failed-listing case and a row reaches it: settled apart from F1 and F2.
  The Intent decides it, the `AttachArgv` doc comment rewrite quotes the sentences as they stand, and `FailedClientListingWithFollowOnSelectsTheRelevantWindow` reaches the path because the table's script runs after the `list-clients` answer and can replace it with an error.
- Focus 4, whether a row labelled as the fallback reaches it: not settled in the plan's favour, see F4.
- Focus 3, whether each later attach is awaited: the card names the wait, so the race the focus asks about is addressed; F5 is about the counts it waits for.
