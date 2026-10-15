# Card 12 — batten-hand-back-notice

**Create:**
- `internal/battenshed#HandBackNotice`
- `internal/battenshed#TestHandBackNotice`

**Uses:**
- `internal/battenshed#rowNotifier.send`
- `internal/battenshed#rowNotifier.setMarkerLine`
- `internal/battenshed#rowNotifier.readMarker`

**Intent:**
Give batten's driver hand-back one orch notice per stop episode, sharing the rows' per-episode marker so a stop a row already announced is not announced twice (D4, the open risk on batten's notices).
Card 13 wires it as batten's hand-back `Notify`.

`HandBackNotice`, in `internal/battenshed/rownotice.go`, sends `line` for the stop at the batten row `producer`, whose stop key's history length is `historyLen`, through the row's own notifier over `deps`:
- It reads the row's marker, `<producer>-row-notice` under `scratchDir`, the file `rowNotifier.failed` keeps.
  A row's `failed` notice records `failure <n>` with the history length it read during its step, before shedengine appended the halting entry, so the same episode's stop key carries `n + 1`.
  `HandBackNotice` therefore sends nothing when the marker holds `failure <historyLen-1>`, the row's own notice for this stop, or `handback <historyLen>`, an earlier hand-back of the same stop by a driver that restarted.
- Otherwise it sends through `rowNotifier.send`, so the line reaches the hub notice log, waits for a recorded orch strand and retries within `noticeMaxAttempts`, as a row notice does.
  A queued line records `handback <historyLen>` through `setMarkerLine`.
- A nil `deps.Notify` or a cancelled context sends nothing.
- It reports whether the line was queued.
`ClearRunMarkers` already clears the marker where batten seeds a fresh status, so a rerun of the slug starts with no episode recorded.

`TestHandBackNotice`, over a fake `Notify`, a fixed notice log and a temp scratch directory as `TestRowNotifierSend_DeliveryBound` builds them:
- a stop with no marker queues once and records `handback <n>`, and a second call at the same length queues nothing;
- a marker holding `failure <n-1>`, the row's own notice of this stop, queues nothing;
- a marker holding `failure <n-3>`, an earlier episode, queues the hand-back;
- with no orch strand recorded nothing is queued and no `handback` line is written, so a later call can still send;
- a nil `Notify` queues nothing.
battenshed stays inside its row in `cmd/lyx/testdata/test-budget.yaml` (113 of 114 after card 8's test).
