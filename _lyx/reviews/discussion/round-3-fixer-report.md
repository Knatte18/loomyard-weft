# Fixer report: discussion, round 3

## Changes

- **F1** (MEDIUM, fixed as proposed): premise confirmed: D7's read-alive disjunct keys on the session id, so with a resume that keeps the id, an earlier launch's process-ending line in `sessionEndReasonEndsProcess`'s set (`internal/shuttleengine/claudeengine/signals.go`) counts once the resumed process is read alive, and any later not-alive reading before `SessionStart(resume)` ends the run died.
  D7 now counts a process-ending `SessionEnd` only past the first-tick events-file size, taken before that tick's registry read, and the disjunct is gone.
  The bound now states the resumed-launch guarantee on the size alone, and it names the cost: a `SessionEnd` written just before the first tick never counts, so that run ends on registry death or its deadline.
  The Death criterion gains the read-alive-then-unproven case.
  The support log's Rejected alternatives record the dropped disjunct and why.
- **F2** (MEDIUM, fixed as proposed): premise confirmed: `pattern/PATTERN-pane-text-injection.md`'s guard bullet requires "One fresh reading" of three facts, and D8 only said it "reads the box again".
  D8 now takes a second full guard reading after the settle (ready, no operator activity within `operator_quiet_s`, the box), names it the write's guard reading, and holds the send on a not-ready session or operator activity.
  Constraints now amend four places, adding the guard bullet's one-fresh-reading clause.
  The Stray character criterion gains operator activity in the settle.
- **F3** (NIT, fixed as proposed): premise confirmed against `PATTERN.md:73`, which reads "no Enter of lyx's own", and `claudeengine/session.go`, where the `%` reading is the provider's and applies at every reading.
  Constraints now quote the `PATTERN.md` line word for word.
  They state that the residue reading is shuttle's, taken only on the post-clear reading, beside the provider's box reading.
  They also state that the paste-submit's Enter belongs to reed's atomic paste-submit.

## Disputed

None.

## Deferred

None.
