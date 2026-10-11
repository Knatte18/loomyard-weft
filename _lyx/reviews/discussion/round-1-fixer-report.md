# Fixer report: discussion round 1

Every finding's premise held against the code and the record; all ten are fixed.
`lyx loom validate-discussion` passes on the result.

## Changes

- **F1 (BLOCKING), fixed differently.**
  Not made: the proposed launch-offset boundary.
  `processLiveness` (`internal/shuttleengine/claudeengine/prober.go`) returns dead when an entry naming the id has a gone pid and none leaves doubt, and registry files are per process (`registryEntry` in `resume.go`), so once a resumed launch's `SessionStart(resume)` lands past the offset, the crashed launch's entry still reads dead until the new process registers, and the offset rule would fail the resume.
  Made instead: D7 counts registry death for a session id only once a liveness tick of the same wait has read that id alive, which needs no comparison between procStart ticks and wall-clock times.
  The age rule and the undefined "for a launch" sentence are gone, and D7's bound names what this rule lets through: a process that dies before the wait reads it alive, or a wait attached after the crash, ends only on a `SessionEnd`, else waits out its deadline.
  The Death criterion gains the alive-then-gone case and keeps the resume case in the new terms.
  Both rejected boundaries are recorded in the support log's Rejected alternatives.
- **F2 (BLOCKING), fixed as proposed (first option).**
  The `PATTERN-pane-text-injection` Constraints entry now amends three places: the `%` sentence gains D8's residue reading and its admitted merge, with the bound.
  It states why the `PATTERN.md` line stays unchanged: the provider's box reading defines what a box holds, and the residue reading joins the `%` reading inside it.
- **F3 (MEDIUM), fixed as proposed (release option).**
  In D5, the interrupt marker now releases an ask with cause `ask` to `idle-stalled`/`interrupt`, and D5 states its bound.
  The criterion is extended, and the Open risk now points at D5's bound for the granted-permission case only.
- **F4 (MEDIUM), fixed as proposed (explain option).**
  D7 states that the fold's settle precedence is unchanged and why the two readings may differ.
  The Death criterion now asserts that the fold reads dead beside a live registry entry while the run is not ended.
  The adopt-the-veto option is recorded as rejected.
- **F5 (MEDIUM), fixed as proposed.**
  D1's condition is now what D4 depends on: the `--settings` `SessionStart(startup)` line lands on its own, with no turn typed.
  The "either hook order" claim survives only in the support log (Q13), where it is true of D3.
- **F6 (MEDIUM), fixed as proposed.**
  D1 now says that the blocker withholds only D4, that every other Decision is built, D3's `startup` rule included since it moves nothing without a line, and that the Startup and #549 criteria drop under it.
  Q11 matches.
- **F7 (LOW), fixed as proposed.**
  D8's clear goes out right after the guard reading with no wait or other key between them, and the bound names that latency as the window.
- **F8 (LOW), fixed as proposed.**
  The pre-startup death criterion is narrowed to a process the wait read alive after its `SessionStart` line.
  The Startup criterion gains the earlier-launch-line case.
- **F9 (LOW), fixed as proposed.**
  Q2 now states the trust proof as conditional on the corpus and points the escalation at D1.
- **F10 (NIT), fixed as proposed.**
  Cut Auto-mode assumptions two and three, the third Open risk and the `shuttle-color-verified-send` Constraints line.
  Completeness check: the assumptions' content is in D1 (capture and blocker) and moved to the support log as Q12.
  The Open risk's content is in D4's rationale, and the Constraints line moved to the support log as Q13; neither carried a requirement absent from D1, D3 or D4.

## Disputed

None.

## Deferred

None.
