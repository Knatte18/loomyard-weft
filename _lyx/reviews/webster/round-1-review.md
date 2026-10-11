---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/shuttleengine/send.go:340
    summary: retryLanded reads any later turn start as the send going out, so a retried slash command with a stale pending record is silently dropped
  - id: F2
    severity: LOW
    class: design
    location: internal/reedengine/io.go:154
    summary: pasteSubmitArgv deletes its named buffer only through paste-buffer -d, so a failed paste-buffer leaves the lyx-inject buffer behind
---

Range: `main..HEAD`, the 18 card commits (status `product.parent` is `main`, no rework round exists).
`lyx gate test` over the touched packages (`shuttleengine`, `reedengine`, `orchengine`, `loomcli`, `battenshed`, `battencli`, `shedrun`, `testkit`, `contracts`, `cmd/lyx`) passed, including the test-budget check.

### [LOW] A retried slash command can be dropped by an unrelated turn start

**Location:** `internal/shuttleengine/send.go:340` (`retryLanded`), reached for slash commands through `sendSequence` and `sendVerified`.
**Issue:** `retryLanded` reads a record with the same text and any turn start past `EventsOffset` as "the text went out".
For a prose send that holds, since the text is the turn.
A slash command (`/clear`, `/compact <focus>`, `/reload-plugins`, `/color <name>`) starts no turn, so a turn start past the offset is some other turn, such as one the operator submitted after the failed attempt.
A failed `/clear` that left a record, retried after the operator emptied the box and ran a turn, returns nil and types nothing, and the orch watcher then confirms a clear that never happened.
Card 7 states "a slash command starts no turn, so its retry always types afresh (R8)", which holds only while no other turn intervenes; the code does not enforce it.
**Fix:** Do not save a pending record for a text the provider treats as a command, or have `retryLanded` require the record's text to be one a turn start can evidence.
Alternatively, record the turn start's identity at save time and compare it, so only a turn that began from this text counts.

### [LOW] Named paste buffer outlives a failed paste-buffer

**Location:** `internal/reedengine/io.go:154` (`pasteSubmitArgv`).
**Issue:** The chain is `set-buffer -b <name> -- <text>`, `paste-buffer -p -d -b <name> -t <pane>`, optionally `send-keys`.
The `-d` that deletes the buffer runs only when `paste-buffer` succeeds.
A pane that vanished between pane resolution and the command leaves `lyx-inject-<hex>` in the server's buffer list, since tmux stops the list at the failing command.
Card 1 and `PATTERN-pane-text-injection` say the named buffer is cleaned up.
The operator's automatic buffer stack stays untouched, so the harm is a leaked buffer per failed write, small and unlikely.
**Fix:** On a non-nil error from the invocation in `pasteSubmitLocked`, run a best-effort `delete-buffer -b <name>` and ignore its error.

## Judgments on the focus questions

- **Guarded path (cards 5-8).**
  `ClearSession`, `CompactSession`, `ReloadPlugins` and `TypeColor` all go through `sendSequence`, and a paste-path engine reaches `sendVerified`, whose first step is `awaitIdleSession`.
  `busyReading` takes the pane capture, the readiness reading with the new no-input-box hold, and the operator-activity query on every poll, so one fresh reading precedes the paste.
  The remaining raw writers (`Interrupt`, `Inject`, `ModelSwitchSequence`, trust-prompt dismissal) are the explicit key deliveries the decision record and the PATTERN background name as outside the rule.
- **Enter and retry (cards 6-7).**
  `confirmPasted` sends the Enter only when the normalized box equals the sent text or the provider's own placeholder, behind `guardedEnter`'s activity check.
  The sent text beside a draft fails with `ErrSentTextInDraft` and any other text with `ErrSessionBusy`, and neither sends a key.
  The pending record is written after the confirm window ends, not before the paste.
  A crash in between leaves our text in the box, and the retry's `awaitIdleSession` holds on the draft, so nothing is typed twice; this is not a defect.
- **Reed (cards 1-3).**
  The paste and its Enter are one `tmux.run` invocation using `set-buffer -b <name> --`, with `;` handled for a trailing semicolon.
  The `/rename` repair asks `operatorActiveLocked` after the idle reading and pastes through `pasteSubmitLocked`.
  Only F2 qualifies the buffer cleanup.
- **Claude (card 9).**
  No sequence opens with an Escape; the one remaining Escape is `InterruptSequence`.
  `awaitSettledBox` runs only in `settleAndConfirmPasted` when `PasteSubmits()` is false, and on the typing engine's box path, as the shared decision says.
- **Held write as "later" (cards 11-15).**
  Every typed step in the orch watcher routes through `heldTyping`.
  In `tickIdle`, `startClearing` and `startCompacting` the phase is entered before the send, so a held send is retyped by the next tick.
  The resume-report path is reused only while the marker content matches, and it is removed on a landed resume and on the spawn path.
  Batten maps `driver_busy` to `ErrChildDriverBusy`, retries once per `noticeProbe` and queues one notice that carries no pane text.
- **Cards 16-17.**
  The flag fires only for a failed, blocked or paused batten over a child whose status reads done, and every read failure answers not found.
  The stencil's `## After landing` section names `lyx batten status <slug>` then `lyx batten run <slug>`, and the claim in `claims_test.go` covers it.
- **Test economy.**
  The new top-level tests are the ones each card prescribes, and I found no new test that duplicates an existing one.
  `TestHaltedOverDoneChild` (the state matrix) and the lifecycle step (the real envelope) cover different layers.
- **Per-card mechanical checks.**
  The `Delete` groups (`isOwnText`, `colorSettleAttempts`, `colorSettleInterval`, `composeSendSettleMS`, `Claude.PastePlaceholder`, `childNotParked`) leave no caller or reference in non-plan files.
  The re-signed helpers compile and have no stale callers.
