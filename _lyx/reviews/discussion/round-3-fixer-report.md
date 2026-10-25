# Fixer report — discussion, round 3

Every premise was checked against the code at this worktree's HEAD, and each held:
`shedengine.Shed.enter` (`internal/shedengine/entry.go:30`) skips the gate while `EnteredRow` names the row, and `Goto` sets it (`goto.go:123`).
`spawnLoop` (`internal/shedverbs/loopwait.go:158`) is the one non-test `proc.DetachBreakaway` caller, reached from the waiter's default branch (`loopwait.go:111`).
`proc.TerminateNamedJob` runs only from `internal/pairteardown/loop_windows.go`, and `internal/adoptcli/doc.go` shows the adopt acting on this machine's pair alone.

## Changes

All changes are in `_lyx/discussion/decision-record.md` unless named otherwise.

- F1 (BLOCKING), fixed as proposed, with one mechanism of my own for telling the parent's resume.
  D6: under scope `full` the agent session's acceptances are marked `unruled`.
  The row checks at every entry, before any compare, and at the end of an attempt: a record holding an `unruled` acceptance ends it Stuck in a decision halt.
  The row tells the parent's resume from that halt by the run status's last history entry being a Stuck of this row whose latest time is at or after the time the row recorded the acceptances it put.
  Only that resume marks the surviving acceptances `ruled`.
  The review's fix says only that "the resume from that decision halt marks the surviving acceptances `ruled`" and names no test.
  A mark written into the scope record alone would let an error between writing the mark and persisting the halt rule acceptances that no parent saw.
  D7 now carries the ruling on each accepted entry and bounds the agent's acceptances at every entry.
  An acceptance criterion was added: a session that accepts and then ends in an error, transient or interrupted stop does not reach Done on resume until the parent rules.
  Support log: Q20 added.
- F2 (MEDIUM), fixed as proposed.
  D2 now states that the gate is evaluated at entry, through `Shed.enter` and `EnteredRow`, and that `lyx loom goto` sets `EnteredRow`.
  D5's flag-off paragraph is split by when the flag goes off.
  Before the run enters `Drift-Await`, the row is skipped.
  While `Drift-Await` waits or is halted, or `Drift-Verify` is halted, neither row is skipped.
  For giving up at an entered row, the paragraph names `lyx drift defer` at `Drift-Await`, `lyx drift waive` at a `Drift-Verify` halt with no counted capture, and none at any other `Drift-Verify` halt.
  The flag-off criterion is limited to the not-yet-entered case, and a criterion was added for the entered cases.
- F3 (MEDIUM), fixed as proposed.
  D5 defines the counted capture once: a `passed` record that, in loom, has `awaited_at` set.
  D6's halt, `lyx drift waive`'s precondition, the waive and flag criteria, Scope In and D5's HEAD-moved deferral sentence all use it.
  Support log: Q21 added.
- F4 (LOW), fixed as proposed.
  D4: the compare marker is an OS lock taken before the slot wait and probed as the capture lock is.
  The watchdog bound now covers a capture queued for a slot.
- F5 (LOW), fixed as proposed.
  D4 and D5 no longer claim that the adopt ends the other machine's capture.
  A job name is per machine, and a capture still running on the other machine ends at its own timeout or with that machine's teardown.
- F6 (LOW), fixed as proposed.
  The Scope In plumbing bullet now names the `runproc` capture role, `proc.HoldNamedJob`, and `pairteardown` terminating the job through `proc.TerminateNamedJob`.
- F7 (LOW), fixed as proposed.
  D4 settles that no step child runs the `--until-stop` waiter, and that a step's own run holds its loop lock, so a waiter there spawns no loop.
  It also says that a loop is meant to outlive its waiter.
  The plan check was dropped.

## Self-check

- fix-claims, MEDIUM, together with fix-way-forward, BLOCKING, merged: shed folds repeated identical exempt Stucks (`appendOrFold` in `internal/shedengine/run.go`, `At` keeps the first time), so the history check could never see the parent's resume.
  Fixed as proposed: the test reads `LastAt` when a fold set it, and `At` otherwise.
- fix-way-forward, LOW: a history entry holds no Stuck reason (`HistoryEntry.Reason` is the entry gate's text only).
  Fixed as proposed: the test reads "a Stuck of this row".
- fix-proposal, MEDIUM: a resume from a failing-compare, dirty-tree or test-gate decision halt marked acceptances `ruled` that the halt never put to the parent.
  Fixed as proposed: every scope-`full` decision halt reached while an `unruled` acceptance stands names those acceptances, with resume and withdraw, and only such a halt records them.
- fix-proposal, MEDIUM: `lyx drift defer` with a capture still running let the capture become the counted capture after code changed.
  Fixed with the second proposed change: `lyx drift defer` refuses while a capture holds the lock, naming `lyx loom resume`, whose row waits on the capture.
  The defer criterion was extended.
- fix-consistency, MEDIUM: "Each resume … spawns a fresh agent" contradicted the entry halt and the clean compare after ruling.
  Fixed as proposed: "Each resume that reaches a failing compare".
- fix-consistency, LOW: D7's "with the attempt it ran in" had no reader left, and Q11 was not marked superseded.
  Fixed as proposed: the attempt field was dropped, and Q11 now notes "Superseded in review round 3 by Q20".
- fix-consistency, LOW, together with fix-way-forward, LOW, merged: the waive refusal and D5's giving-up sentence named only accept or fix, which missed the stand-by halt's ways forward.
  Fixed as proposed: both now name the ways forward the halt itself names.
- fix-claims, LOW: D4's list of waiters left out batten's driver and the loom run verbs.
  Fixed as proposed: the list is now named as a group ("a driver, a run verb or the arbiter's relaunch line, outside any step").
  fix-proposal confirmed that batten's driver is spawned by the dobby (`beamcli/dobbydeps.go:245`), not by a step.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
