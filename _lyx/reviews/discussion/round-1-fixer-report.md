# Fixer report — discussion round 1

Every finding's premise was checked against the code and held; all six are fixed as proposed, in `_lyx/discussion/decision-record.md`.

## Changes

- **F1** (MEDIUM, design) — fixed as proposed.
  Premise confirmed: `loadSkillsThenPrompt` in `internal/shuttleengine/run.go` calls `run.abandonStartup(OutcomeDied, nil)` when the pane dies during the skill-load turn, after ready, and the cause-carrying teardowns in `wait.go` and `run.go` carry no pane tail.
  D4 now keys the tail on `awaitStartup`'s not-ready exits, says the died-during-skill-load teardown gets no tail and why, and says the non-busy causes get no tail on purpose.
  The matching Acceptance criterion now names the skill-load case.
- **F2** (MEDIUM, consistency) — fixed as proposed.
  D2's background line now also states the let-through: a lone-`%` draft typed outside `operator_quiet_s` reads as empty and a write can merge with it.
- **F3** (MEDIUM, design) — fixed as proposed, first option.
  D5's Rationale and the `PATTERN-daemon-wakeups` Constraints line now rest on the background's definition, "no deadline of its own", which `awaitStartup` does not meet, name that its 90 s window exceeds the minute the background names, and say the plan follows that reading with no backoff.
- **F4** (LOW, design) — fixed as proposed.
  D6 now states the multiple is floored to 1 as `Wait` floors it, that a probe at or past one gap may dismiss, and that only a non-empty play sent without error starts the gap.
  The premise sentence about `Wait`'s probe now names the floored multiple and the at-or-past rule it depends on.
- **F5** (LOW, consistency) — fixed as proposed.
  D1's bound now says the operator-activity hold runs before every write behind the pre-write guard, and names the session idle probe's no-run fallback (`internal/shuttleengine/session.go`) as a reading with no hold.
- **F6** (LOW, design) — fixed as proposed.
  Premise confirmed in `internal/seatengine/work.go`: `outstanding` also holds true for an advisor whose up line the sender has not landed.
  D7 now says the cap governs any reading `outstanding` keeps true on a live advisor, naming both cases.

## Disputed

None.

## Deferred

None.
