```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough lens. I ran each decision's mechanism end to end against the code and the 2.1.312 source in context.

**Walkthrough list**

- **D1, one mail #13 posted mid-turn and claimed at the turn's Stop: purpose met.**
  - The Stop runs with `stop_hook_active` false, so the merged command pipes to `hook-read` and `ClaimAtBoundary` resets the count to 0, hands out #13 and raises the count to 1.
  - `BoundaryBlock` prints `additionalContext` only, `hookReadExit` returns 0 and the merged command records no turn end.
  - Claude Code pushes `hook_additional_context` into `He` and returns `{blockingErrors:He}`. The turn continues with `stopHookActive:true`, and the operator gets one `Stop hook feedback:` entry through `Czs(...oo)`.
  - The next Stop has nothing to claim, so `hook-read` exits 1 and the turn end is recorded.
- **D2, a mail claimed after a Bash tool call: purpose met.** The JSON carries `additionalContext` (`PostToolUse`) and `systemMessage`, the exit is 0 and no block is written.
- **D3, strand idle with a marked waiter, #14 posted: purpose met on the main path, but one release path breaks (F1).**
  - `wakeHookStrand` sees ids [14], a waiter and `NoTurn`, so `ringHook` claims #14 in its locked step and the outcome carries it.
  - The client handler writes stderr, then acks. The daemon journals the read, confirms, and `hook-wait` exits 2.
  - Claude Code checks `Ae.code===2` and builds `` `${Qe} ${lt}` `` with `lt=De||Oe` (stderr) and `priority:"next"`.
  - While #14 is reserved, `unreadLocked` leaves it out, so `hookWakeState` returns no ids and no bell can fire.
  - Release path: the connection drops before the ack, the claim is released, ids are back to [14] and equal `mark.ids`, so the ring is not ringable. With no waiter, an unanswered mark and `NoTurn`, the bell fires after `ringAnswerWindow`, as D3 states.
  - Woken turn: the notification sets `stopHookActive:!0`, so its Stop arrives with `stop_hook_active` true. The ring's reset left the count known at 0, so a Stop claim in that turn still works.
- **D4, the waiter entry rendered with both keys: purpose met.** `_tt` passes `e.rewakeMessage` and `e.rewakeSummary` through, giving model text `[creel] Mail that arrived while you were idle: [creel] #14 …` and summary `creel mail` (`We` applies to plugin hooks only).
- **D5, one claim per outcome kind: mostly met.**
  - Tool-boundary claim acknowledged → `acknowledged` at Info.
  - Client write fails → no ack → `handOut` releases → `released`.
  - Pause, cap or unknown count → nothing handed out, at Debug.
  - One outcome is not covered (F2).

**Findings**

**F1 — LOW, design.** Location: D3 and the Notes bullet "The Go client today acknowledges an AckRequired response by itself when no handler is given; D3's client passes one".
- D3 says `hook-wait` never resumes once a ring carrying mail has arrived.
- The existing client resumes exactly that case. `CallWatching` reconnects whenever the error is a `connLostError`, the connection was established and `req.Wait` is true, and `hook-wait` sends `Wait: true`. `resume` re-sends the same request for `VerbHookWait`.
- After the handler has written the lines, both the ack write failing and `awaitAckConfirmation` seeing the connection close return `connLostError`. The client then re-registers the waiter with the lines already on stderr.
- At the next ring, Claude Code reads the whole stderr, so the model gets the first ring's lines a second time while the released mail is also handed out again. That is the duplicate D3's rule exists to prevent.
- The record states the rule, and the D3 criterion ("exits 0 without resuming") would catch it. The Note still suggests a handler is the only client change needed.
- Fix: D3 or its Note says the hook-wait call must not reconnect once a ring response with lines has reached the handler.
- Evidence: `internal/creelengine/client.go` `CallWatching` (the `!errors.As(err, &lost) || !established || !req.Wait` check, then `c.resume`), `resume` (`case req.Verb == VerbRead, req.Verb == VerbHookWait: return req, nil`), the ack write and `awaitAckConfirmation` returning `&connLostError{...}`, and `internal/creelcli/hookwait.go` (`Request{Verb: creelengine.VerbHookWait, Wait: true}`).

**F2 — NIT, consistency.** Location: D5's outcome list (`acknowledged`, `released`, nothing handed out).
- In `handOut`, a client can acknowledge while `reading.Ack()` fails to journal. The daemon then logs only its existing `Warn` and returns an error response.
- That attempt is neither `acknowledged` nor `released`, so D5 names no line for it with strand, path and ids.
- Fix: D5 names that outcome, such as `ack-failed`, or says the existing Warn carries the strand, path and ids.
- Evidence: `internal/creelengine/server.go` `handOut` (`if err := reading.Ack(); err != nil { logger.Warn("creel daemon: could not journal an acknowledged read", ...)`).

I made no tool calls; everything above comes from the inherited context.
