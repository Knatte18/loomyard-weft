# Card 2 — Attach selects the relevant window by follow state

**Edit:**
- `internal/reedengine#Engine.AttachArgv`
- `internal/reedengine#TestAttachArgv_SelectsTheRelevantWindow`
- `internal/reedengine/doc.go`

**Uses:**
- `plan:internal/reedengine#Engine.followingLocked`
- `internal/reedengine#Engine.attachWindowSelectionLocked`
- `internal/reedengine#relevantWindowFixture`

**Intent:**
D4: in `AttachArgv`, the selection is resolved when no other client is attached or when `followingLocked` reads on, so with follow on an attach appends `; select-window -t <relevant window>` also beside another client, and with follow off it keeps today's rule.
`followingLocked` is read only when another client is attached; attach never writes the option.
With no relevant window, nothing is appended and follow stays as it is, so the run's start (card 4) or its next agent window moves the viewer later.
The comment above the selection, "A second attach never moves a viewer", is rewritten to the new rule.

The bound: with follow on, an attach moves every client of the session to the relevant window, including one on a window it reached outside reed's bindings; a failed follow read counts as off, so it then moves none while another client is attached.

`TestAttachArgv_SelectsTheRelevantWindow` gains a `follow` answer and a `followErr` field, scripted with `answerFormat("#{@lyx_follow}", ...)`.
The row `AnotherClientAppendsNothing` becomes `AnotherClientWithFollowOffAppendsNothing` (follow `off`); new rows: `AnotherClientWithFollowUnsetSelectsTheRelevantWindow` (follow scripted as the empty answer of an unset option, want `@2`), `AnotherClientWithFollowOnSelectsTheRelevantWindow` (follow `on`, want `@2`), and `AnotherClientWithAFailedFollowReadAppendsNothing` (followErr set).
The format must be scripted explicitly in every row that attaches another client, since `newAttachTestEngine`'s fake answers an unscripted `display-message` with the box string.
Every row also asserts that no call carries `@lyx_follow` other than the `display-message` read, so attach never writes the option, and the rows with no other client assert that no `#{@lyx_follow}` read is issued at all.
The test's doc comment drops "A second attach never moves a viewer" for the follow rule.

`doc.go`'s attach sentence in the focus-rules paragraph states the follow rule and its bound.

**ImpactSummary:** One method changes, its selection gate only; the other `AttachArgv` tests attach no other client or script no own window, so their argvs are unchanged.

**Commit:** 2: reed: attach selects the relevant window beside another client while follow is on
