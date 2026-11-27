# Machine stops, dead judges and idle seats are not producer failures

Several infrastructure stops were treated as an agent's own failure, or as a reason to leave a session alone.
This change classifies each one for what it is, so a run recovers or stops with the right cause.

- **A cold fixer ended by a machine stop** was reported as `timeout`, and it spent one of the visit's fixers.
  `gatefixer.Await` now reads the strand before the timeout.
  A gone strand whose shuttle run is recorded as interrupted ends dead as `interrupted`.
  The row gate replaces such a fixer, resuming its session where shuttle can.
- **A bouncer judge that died before its verdict** degraded to `Stuck`, which spent the row's bounce budget.
  `Bouncer.Call` now returns an error.
- **The prompt Claude Code leaves after an API error** (`Resend your message to try again`) was read as a typed draft,
  so `lym tender revive` skipped the stalled session. It now reads as an empty input box.
- **An idle seat read as having no input box** could kill a review before its prompt was delivered.
  When the styled capture shows no box and the plain capture shows one, shuttle now reads the plain capture.
- **Two strands in one pane scope** each got an OOM notice for the same kills. A cgroup now gets one notice.

Closes #248
Closes #239
Closes #219

## Design decisions

**Interrupted fixers**

- A replacement runs on the interrupted fixer's report and reply files, which is what lets shuttle resume the session.
- Interrupted records spend none of the `attempts + 1` fixer budget.
  Replacements stop once the set holds more than `attempts + 1` interrupted records,
  and the interruption then stands as the outcome.
- The entry that carries the fixer, its final closure included, may now spawn a replacement.
  Another entry that fails over an interrupted fixer leaves the record live for that entry,
  and `Expired` does not report it.
- A killed pane is recorded the same way as a machine stop, so a fixer whose pane someone killed is replaced too.
- Webster's plan and verify fixers share the new order and can report `interrupted`, but they get no replacement here.
- The orphan sweep now keeps a stopped, interrupted run's directory while a later start of its role may resume it.
  That record is the only evidence of the interruption.

**Bouncer judge**

- A judge that never came up wraps `shuttleengine.ErrNotStarted`, which the driver re-steps as a transient agent start.
- A judge that died after starting wraps `shuttleengine.ErrAgentDied`.
  It is transient only under the existing host spawn-refusal rule; otherwise the call fails with that error.
- A run error, from a spawned or an attached judge, also fails the call and wraps its cause.
  The `RunError` row of `TestBouncer_JudgeCall_Degradations`, which expected `Stuck`, is replaced accordingly.
- A timed-out judge still degrades, and a verdict written before the judge died is still acted on.

**Input box**

- The plain-capture fallback blanks no placeholder, so a placeholder read that way counts as a draft and holds the send.
- The disagreement is warned once per strand per process, with the styled capture's last lines, and logged at debug after.
- An operator draft of exactly the API-error prompt now reads as empty.
  The operator-activity hold still runs before every write.

## To check by hand

- The idle-seat fix is a fallback around the styled reading, and its tests simulate the lost box with a fake engine.
  On a live seat, check that skill load is followed by prompt delivery,
  and read the styled tail of any disagreement warning to find why the styled reading lost the box.
- The interrupted-fixer path is tested with a fake fixer and hand-edited run records.
  A real reboot or killed pane during a row gate, ending in a resumed replacement, is worth one live try.
- The API-error prompt is matched as exact text against constructed captures.
  Check `lym tender revive` against a session stalled on a real API error.
- A shared pane scope's single OOM notice names the first strand in the scope,
  which may not be the one whose process was killed (`planPaneOOMNotices` in `internal/reedcli/oomnotice.go`).

## Merge danger

**Door:** two-way
**Blast radius:** shuttle
