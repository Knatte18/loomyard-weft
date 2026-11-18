```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out as proposed (F1, F2, F4–F10; the proposals for F4, F5, F7, F8, F9, F10 are test or text only), with no tool calls made.

**1. MEDIUM. `run.go#sendStartPrompt` and `sendThroughPause`: F6's `sc.deadline = run.deadline` makes a start fail after a pause.**
- `observePause` (wait.go:474) is the only code that moves `run.deadline` later by paused time, and it runs only inside `Wait`.
- During `Start` the deadline is the initial `createdAt + Timeout`, 5 minutes by default.
- Case: a hub pause covers the strand while the start prompt is sent, and it outlasts the timeout.
- `sendThroughPause` waits out the pause and rebuilds the context with the stale `deadline` the fix now keeps.
- The resend then fails: `awaitIdleSession` sets `limit = sc.deadline`, which is already past, so the first busy reading returns `SessionBusyError`. If the box reads idle, `pasteVerified` returns "submit window … had closed before the paste; nothing was pasted" (`ErrSubmissionNotLanded`).
- Either way the start is torn down as died. Before F6 the pause was waited out and the line was typed.
- The `PATTERN-reed-panes` rule quoted in the review's Rejected section says the deferred prompt waits out the pause and resends, and this breaks it for the start path.
- The test rows set no pause, so none catches it.

**2. LOW. F1's offset is read before the send's idle waits, so an operator turn during them still cancels the retype.**
- Affected text: `promptSendOffset = sentAt` in `loadSkillsThenPrompt`, and `sendOffset := eventsSize(...)` in `typePendingPrompt`.
- Both offsets are taken before `sendStartPrompt` or `sendThroughPause`, which can spend up to `(startPromptBusyRetries+1)` idle windows, or a pause wait, before anything is typed.
- A hold by an active operator is the usual reason for those waits.
- Case: the operator submits their own turn during the hold. That turn start lands past the stored offset, so `retypeLostPrompt` reads it as the line's turn and never retypes a dropped line.
- This is the premise of F1 in a narrower form. The proposal's wording ("just before the watched typing") is met only for a send that does not wait.
- The row added for F1 seeds the operator turn before the attach, so it does not cover this.

**3. LOW. F2's unlatched busy retry blocks `Wait` for a full idle window on each tick.**
- Affected text: `retypeLostPrompt`, which returns early on `*SessionBusyError` without latching.
- It holds `sendMu` and `Wait`'s goroutine through each wait, up to `send_ready_timeout_s` (60 s by default).
- Case: a persistently held reading, such as the no-input-box hold from #137, repeats a full wait plus a Warn on every tick until the run deadline.
- That blocks event polling, liveness checks and any done/stop classification for those 60 s spans.
- The proposal names no cap, unlike the start path's `startPromptBusyRetries`.
- The consequence is bounded by the deadline, so this is a residual risk, not a wrong result.

Every other proposal I weighed (F9, F10, and the test-only changes F4, F5, F7, F8) is sound as proposed.

Further tool calls: none. Context already held everything this lens needed.
