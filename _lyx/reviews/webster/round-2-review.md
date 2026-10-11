---
verdict: APPROVED
---

No findings.

The review range is `git diff $(git merge-base main HEAD)..HEAD`: `_lyx/shed/shuttle-hook-first/status.json` names `main` as `product.parent`, and `_lyx/loom/rework/` does not exist.
`lyx gate test ./internal/shuttleengine/...` passes for `shuttleengine` and `claudeengine`.

## Focus questions settled

- `internal/shuttleengine/send.go`, `confirmPasted`: `awaitsLanding` is `sc.landingStart != "" && parsesSignals`, with `parsesSignals` the `SessionSignalParser` assertion on the engine.
  An engine without the parser therefore confirms `/clear` and `/compact` on the first empty-box read, as any other send does.
  An engine that parses signals but records no matching line keeps reading while `windowClosed` and `maxReads` allow, then the empty box confirms, so it waits out the submit window.
  The doc comments on `confirmPasted` and in `shuttleengine/doc.go` state both.
- `internal/shuttleengine/session_test.go`, `checkLandingStartConfirmation`: the row "a session start of the matching source before the send is waited out" seeds `SESSIONSTART:<landing>` through `appendEventsLine` before the call, so it lies before the pre-send offset `eventsSize` takes inside `pasteVerified`.
  The row has no `pasteLine`, so nothing is written on the paste, and `wantWait` requires `waited >= window`.
  The row "an engine that parses no session signals confirms on the empty box at once" runs `parserlessPasteEngine`, which promotes only `Engine`, `TextPaster`, `InputBoxReader` and `SessionCycler`, so it does not satisfy `SessionSignalParser`, and requires `waited < window`.
  Both run for `ClearSession` and `CompactSession` through the one table-driven helper, so there is no duplicated test.
- `internal/shuttleengine/claudeengine/activity_test.go`, `TestTurnEndActivity`: the row "a pending call in a batch entry before the first read's window reads now" puts `batchPadding`, an entry of `msg_batch` longer than `initialReadChunk`, between `batchFirst` and `batchLast`.
  `readBackward` drops the first partial line of its window, so the first 64 KiB window holds only `batchLast` and its result, and `newestToolCallingMessagePending` returns `settled == false`.
  The window then doubles to reach `batchFirst`, whose call is still pending, so the row reads `now`.
  A first window that sufficed would read not pending, so the row pins the widening.

## Remainder of the diff

Read for correctness and against the plan's cards 1 to 11: the session-start sources and the idle-fresh fold, the `PermissionRequest` ask and its interrupt release, startup waiting on a session-start line, `sessionGone`, the stray-character clear, the batten notice quoting the box, and the `PATTERN-pane-text-injection` update.
Nothing in them departs from the cards, a listed invariant or the comment conventions, and no new test duplicates an existing one.
