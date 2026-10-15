# Card 2 — Attach selects the relevant window by follow state

**Edit:**
- `internal/reedengine#Engine.AttachArgv`
- `internal/reedengine#Engine.warnMismatchedClientsLocked`
- `internal/reedengine#TestAttachArgv_SelectsTheRelevantWindow`
- `internal/reedengine#TestAttachArgv_MultiClientWarning`
- `internal/reedengine/doc.go`

**Uses:**
- `internal/reedengine#Engine.followingLocked`
- `internal/reedengine#Engine.attachWindowSelectionLocked`
- `internal/reedengine#relevantWindowFixture`

**Intent:**
D4: in `AttachArgv`, the selection is resolved when no other client is attached or when `followingLocked` reads on, so with follow on an attach appends `; select-window -t <relevant window>` also beside another client, and with follow off it keeps today's rule.
`followingLocked` is read only when another client is attached; attach never writes the option.
With no relevant window, nothing is appended and follow stays as it is, so the run's start (card 4) or its next agent window moves the viewer later.
A failed client listing counts as another attached client, as `warnMismatchedClientsLocked` reports it, so the follow read decides it too: follow on selects, follow off or a failed follow read drops the selection.
The comment above the selection, "A second attach never moves a viewer", is rewritten to the new rule.
`AttachArgv`'s doc comment is rewritten to the same rule: its sentences "A second attach never moves a viewer: the selection is dropped when another client is attached, when the client listing fails, or when no window is relevant." and "Failing to take the lock, read the session or the state, or list the clients drops it too" now say that another client or a failed listing drops the selection only while follow reads off or its read fails, and that failing to take the lock or read the session or the state still drops it.
`warnMismatchedClientsLocked`'s doc comment drops "which is what drops the attach window selection from the argv" for "which AttachArgv's selection gate reads"; its signature is unchanged.

The bound: with follow on, an attach moves every client of the session to the relevant window, including one on a window it reached outside reed's bindings; a failed follow read counts as off, so it then moves none while another client is attached.

`TestAttachArgv_SelectsTheRelevantWindow` gains a `follow` answer and a `followErr` field, scripted with `answerFormat("#{@lyx_follow}", ...)`.
The row `AnotherClientAppendsNothing` becomes `AnotherClientWithFollowOffAppendsNothing` (follow `off`); new rows: `AnotherClientWithFollowUnsetSelectsTheRelevantWindow` (follow scripted as the empty answer of an unset option, want `@2`), `AnotherClientWithFollowOnSelectsTheRelevantWindow` (follow `on`, want `@2`), and `AnotherClientWithAFailedFollowReadAppendsNothing` (followErr set), and `FailedClientListingWithFollowOnSelectsTheRelevantWindow`, whose script answers `list-clients` with an error and the `#{@lyx_follow}` read with `on`, want `@2`.
The format must be scripted explicitly in every row that attaches another client, since `newAttachTestEngine`'s fake answers an unscripted `display-message` with the box string.
Every row also asserts that no call carries `@lyx_follow` other than the `display-message` read, so attach never writes the option, and the rows with no other client assert that no `#{@lyx_follow}` read is issued at all.
The test's doc comment drops "A second attach never moves a viewer" for the follow rule.

`TestAttachArgv_MultiClientWarning`'s subtest `ListClientsError_WarnsAndAppendsNoSelection` scripts live agent windows and a failed `list-clients`, so with the fixture's unscripted `display-message` answer follow would read on and the argv would gain a selection.
It now scripts `fake.answerFormat("#{@lyx_follow}", "off", nil)` and keeps its no-selection expectation as the failed-listing-with-follow-off row, while `FailedClientListingWithFollowOnSelectsTheRelevantWindow` above covers follow on.
The test's doc comment sentence promising "an argv byte-identical to what the same script produces with no attached clients at all" is rewritten to hold only while follow reads off or no window is relevant.

`doc.go`'s attach sentence in the focus-rules paragraph states the follow rule and its bound.

**ImpactSummary:** One method changes, its selection gate only; apart from `TestAttachArgv_MultiClientWarning`'s failed-listing subtest, which now scripts follow off, the other `AttachArgv` tests attach no other client or script no own window, so their argvs are unchanged.

**Commit:** 2: reed: attach selects the relevant window beside another client while follow is on
