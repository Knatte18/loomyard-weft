# Round 2 fixer-report: discussion

## Changes

- **F1** (BLOCKING), fixed as proposed, extended.
  Premise checked against `internal/reedengine/attach.go`: `AttachArgv` returns the bare argv on a non-positive client size before taking the lock, on a window-size or status readback failure, on `len(live) < 2` and on no placed strand, and logs every suppression at Warn.
  D5's Attach bullet now appends a `select-window` on the relevant window to every argv `AttachArgv` returns, the bare argv of every degrade path included, resolved under the op lock ahead of the layout chain's guards.
  Beyond the `fix` key, which listed only the in-lock guards: the selection also runs when no client size is known, since that early return is the one the review's prose names as skipping D10 too.
  Only a failure to take the lock or read reed state drops it, at Warn.
  Also from the review's prose: the chain's skip for fewer than two panes or no placed strand now logs at Debug, since it is the normal state under D3 and D4; the other suppressions keep Warn.
  D10 states its rename and move run in the attach pre-flight ahead of those guards and with no client size known.
  An acceptance criterion now requires the attach selection when Selvage is the only pane in the shell window and when no client size is known.
  The Constraints' `PATTERN-log-levels` line now includes D5.
- **F2** (LOW), fixed as proposed, extended.
  D10 logs a failed rename or move at Warn once per session and at Debug while it persists, and at Debug on psmux, as D2 does; the retry at the next open or attach stays.
  Added that a session option records the warned failure, since each window open and attach runs in its own process and an in-process `logger.Episode` would Warn every time.
- **F3** (LOW), fixed as proposed.
  D5's relevant-window paragraph now says that on psmux, where D2's order is best-effort, the relevant window may be an older live agent's.

`support-log.md` is unchanged.

## Disputed

None.

## Deferred

None.
