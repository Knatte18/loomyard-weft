# Card 10 — The note gate passes at a turn end with the note written, no reply needed

**Edit:**
- `internal/orchengine#Watcher.readReplies`
- `internal/orchengine#Watcher.tickHandoff`
- `internal/orchengine#replyWritten`
- `internal/orchengine#watchEnv.finishNote`
- `internal/orchengine#TestWatcher_NoteReplyGate`
- `internal/orchengine#TestWatcher_SoftDeferWithHandoffWrittenProceedsToClearing`
- `internal/orchengine#TestCompact_RenderFailureAfterNoteChangesNothing`
- `internal/orchengine#TestWatcher_NoteRequestRendersTemplateFileBeforeTyping`
- `internal/orchengine/doc.go`
- `contracts/stencils/orch/orch-template-handoff.md`
- `contracts/stencils/orch/orch-template-handoff-soft.md`
- `contracts/stencils/orch/orch-template-role.md`

**Delete:**
- `internal/orchengine#Watcher.isRequestedNote`

**Create:**
- `internal/orchengine#Watcher.notePassed`

**Uses:**
- `internal/orchengine#handoffWritten`
- `internal/orchengine#Watcher.startClearing`
- `internal/orchengine#Watcher.startCompacting`
- `internal/orchengine#Watcher.tickClearing`
- `internal/orchengine#newWatchEnv`
- `internal/orchengine#watchEnv.injectHandoff`
- `internal/orchengine#watchEnv.writeHandoff`
- `internal/orchengine#watchEnv.endTurn`
- `internal/orchengine#watchEnv.reply`
- `internal/orchengine#newSoftEnv`
- `internal/orchengine#watchEnv.injectSoft`
- `internal/orchengine#TestWatcher_HandoffTurnEndWithoutFileNoClear`
- `internal/orchengine#TestWatcher_SoftDeferWithoutFileReturnsIdleAndBlocksNextAttempt`

**Intent:**
D4: in phase handoff-requested the watcher ends the note phase at a tick where the newest session event read since the phase began is a turn end and the requested note file exists and is not empty; the orch posts no `written` reply.

`Watcher.notePassed` answers true when `w.newest` is set and `isTurnEnd(*w.newest)`, and `handoffWritten(st.PendingHandoff)` is true.
`w.newest` holds only events read since the phase began: `tickIdle` sets it to nil after entering the phase, and a restarted watcher's `initCursor` resets it and re-reads from the persisted `PhaseEventsOffset`, so it finds a turn end read before the restart.
`tickHandoff`, after its ask and timeout checks and before its early return for a request already posted, calls `notePassed` and, when it passes, moves on with `startCompacting` in compact mode and `startClearing` otherwise, as `readReplies` did.
Today the move ran in the mailbox handler, before `tick`'s phase switch, so the same tick then ran `tickClearing` or `tickCompacting` and sent `/clear` or `/compact`.
`tickHandoff` keeps that one-tick sequence: after a start that entered its phase, it loads the state and runs `tickClearing` or `tickCompacting` on it in the same call; a start that returned to idle (a render failure) or failed returns at once.
So `reachClearing` ("clearing with the clear sent"), `reachResuming`, `reachReloadStep` and `reachCompacting` keep their tick counts.
A turn end with the note missing or empty keeps the phase waiting until a later turn end finds it written or the timeout ends the phase; the guarantee becomes: nothing reaches `/clear` or `/compact` before the named note exists, non-empty, at a turn end of the note phase.
Bound: a note the orch extends in a later turn is cycled on the version present at the first qualifying turn end.

`readReplies` keeps the `defer` handling unchanged: a `defer` to a soft request with the note unwritten returns to idle with `deferred`, and is ignored in a hard or requested cycle or once the note is written.
A `defer` read in the same tick as a passing gate is therefore ignored, since the gate needs the note written, and the mailbox is read before `tickHandoff` runs.
A body starting with `replyWritten` is logged at Info as `orch: a written reply is ignored, the note gate reads the note itself` and dropped; every other body is logged and dropped as today.
`Watcher.isRequestedNote`, whose one caller was the `written` branch, is deleted, and the `replyWritten` comment says the reply is only recognised to be ignored.

The handoff stencils stop asking for the reply: `orch-template-handoff.md` asks the orch to write the note to the path following the template and then end its turn; `orch-template-handoff-soft.md` keeps the `defer` reply for a cycle it declines and says that otherwise the orch writes the note and ends its turn, with no reply.
Each stencil's leading comment drops the sentence about the mail's id where the text no longer names one; the soft stencil keeps it for the `defer` reply.
`orch-template-role.md`'s cycle paragraph says the note request expects no reply, that the watcher cycles once the note exists at the end of the turn, and that a soft request may be declined with `defer`.
`internal/orchengine/doc.go`'s handoff-requested bullet and its guarantee sentence are rewritten to the turn-end gate, the restart re-read, the ignored `written` reply and the kept `defer`.

Tests:
- `watchEnv.finishNote` writes the note, then ends a turn (`endTurn`), which ticks; every caller of it (the clear and compact paths) keeps passing.
- `TestWatcher_NoteRequestRendersTemplateFileBeforeTyping` drops every assertion on a stencil literal, the `"written <path>"` text in both requests, `lyx creel post --reply` and `[creel]` in the hard request and `lyx creel post --reply <id> defer` in the soft one, and adds none, since `PATTERN-test-economy` forbids pinning a stencil's static prose.
  It keeps the Go-filled values, the note template path and the `PendingHandoff` path in both requests, and the family rule, a fragment Go chooses, both ways: the hard request offers no `defer` and the soft one does.
  Review, not a test, guards that the reworded stencils ask for no written reply.
- `TestWatcher_NoteReplyGate` becomes a table of gate rows: a note written with no turn end read keeps the phase; a turn end with an empty note keeps it; a note written, a turn end read, then a turn-start event read on a later tick keeps it, since the newest event is no turn end; a `written` reply with the note written and no turn end keeps it and reads the reply; a note written then a turn end moves on to clearing; and a turn end read while the note was missing, then the note written and a new watcher built, moves on to clearing at the new watcher's first tick, which re-reads that turn end from the phase's events offset.
  Its subtests about a reply naming the note relative to the prime root and a failed save after the reply is read are removed, since the `written` reply no longer moves the phase.
- `TestWatcher_SoftDeferWithHandoffWrittenProceedsToClearing` writes the note, posts `defer` and ends a turn in the same tick: the defer is ignored and the phase moves on to clearing with no deferral recorded.
- `TestCompact_RenderFailureAfterNoteChangesNothing` writes the note and ends a turn instead of replying, expects the tick error and the phase kept, and drops its unread-reply assertion.
- `TestWatcher_SoftDeferWithoutFileReturnsIdleAndBlocksNextAttempt` keeps pinning a `defer` with the note unwritten returning to idle with `deferred`.

**ImpactSummary:** The note gate moves from the mailbox handler to the handoff tick inside orchengine, with the two handoff stencils and the role stencil reworded; the only reply the watcher still acts on is the soft `defer`, and no caller outside orchengine reads the gate.

**Commit:** 10: the orch note gate needs no reply
