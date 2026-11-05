# Read the load turn's end when no signal lies past the prompt offset

A seat start refused its first prompt with "the pane is not idle" although the pane showed an empty prompt after the skill-load turns.
The start moves the run's prompt offset past the events the load turns wrote, and the readiness reading folded only the signals past that offset.
Right after the load turns nothing lay past it, so the session state read unknown, and the reading fell back to the pane, which can lag the turn's end by a moment.
The retry hit the same state and failed the same way, which consumed a review round.

`readinessState` in `internal/shuttleengine/readiness.go` now folds the whole events file when no signal lies past a non-zero offset, so the send reads the last load turn's end instead of unknown.
`sessionReadiness` calls it in place of `readRunSessionState`; `doc.go` describes the rule.

Closes #50

## Design decisions

- The fallback applies only to the exact state "unknown with cause no-signal" at a non-zero offset.
  A zero offset, a different unknown cause and any state with signals past the offset read as before.
- A stale unmatched turn start in the events file stays held, since the whole-file fold sees it.
  The change does not release a turn that never ended.
- The pane stays a required check in the full readiness reading.
  The change only replaces the unknown state, so the box and hold checks still run.
- No retry with backoff was added to the seat start; the issue offered it as an option, and the state fix removes the cause.

## To check

- `readiness-fallback-sessionidle-test-gap`: the `TestRunner_SessionIdle_ReturnsScriptedClassification` rows for a seeded events file cover the fallback through `Runner.SessionIdle`.
  Check that they pin the three cases above: the load turn's end, a stale turn start and a zero offset.
- `hookStateSkillEngine` in `fakes_test.go` no longer treats a launch-carried load as idle.
  Check that the start tests still exercise the pane-busy case through the hook fold alone.
- Not run by hand: a real seat start against a live provider, where the post-turn settle occurs.
