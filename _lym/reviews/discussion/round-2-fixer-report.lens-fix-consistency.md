```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's changes agree with the sibling decisions, criteria and cross-references around them (fix-consistency lens).

Findings:

1. **LOW**: D4's new sentence ("A session reopened outside the row while the run is halted … is admitted only if it keeps the recorded strand name; otherwise its `decision add` is refused until `lym loom resume` re-enters the row and re-records it") has no matching acceptance criterion.
   - The fix added a new admission and refusal rule to D4, but the D4 acceptance bullet was left unchanged.
   - Evidence: the D4 criterion covers only "after the session was resumed under a new strand", another label, the strand after the row ended, and another task strand. It has no case for a session reopened outside the row under a different strand name that is refused until `lym loom resume`.
   - The F1 twin was handled: the fixer extended the D6 criterion, but not D4's.

2. **LOW**: the way forward in D4's new sentence depends on D5's ensure order, and D5 does not cover a session that is live under an unrecorded strand.
   - Evidence: D5's first bullet says "A recorded strand that is live is reused, a recorded session id whose strand is not live is resumed".
   - If a reopened session runs under a different strand name, `lym loom resume` finds the recorded strand not live and resumes the recorded session id. That session id is already running in the reopened pane.
   - D4 now promises that this resume "re-records it", meaning the reopened session. D5 instead describes resuming the session id again and does not say what happens to the session already running it.
   - It is unclear whether this case can occur; it depends on whether `lym reed resume` keeps the strand name. Either way, the changed D4 text and the unchanged D5 bullet do not agree on it.

Other twins checked and consistent:
- D6's Bound ("ends the row Done at once … without starting a session") matches the new entry stop.
- The D6 criterion ("on a clean checkout it ends the row Done and the session is stopped") matches.
- D8's `sampler-done` bullet matches D5's new pointer to it.
- No other text in the record or the support log restated the old publish steer, "always the live one" or "its own diff".

Tool calls:
- grep of the record and support log for steer, publish, push, "always the live" and "own diff": to find twins of the changed text outside D4 to D6, which my context had not shown.
- Read of the record's Constraints, Open risks and Notes for the plan writer, plus the support log's entry and stop lines: to check the sections my context lacked for statements the fix touches.
