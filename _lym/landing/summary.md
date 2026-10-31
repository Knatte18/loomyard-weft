# The adopt check shows its progress and result on the prime's status bar

`lym adopt` starts the adopt check, a gate run of several tiers, detached in the prime.
Until now nothing the operator looks at showed that it ran or how far it had come:
the envelope carried `suite` among other JSON, each tier went to the orch as mail, and `lym beam status` showed it on request.

## What changes

- **The status bar.**
  `reedengine.Engine.SetCheckMark` sets three session user options on its session, `@lym_check` (the text), `@lym_check_state` (`running`, `green` or `failed`) and `@lym_check_start` (epoch seconds), or clears them.
  Line 2 of reed's status bar renders the current session's mark right-aligned before the clock, styled by state: yellow on grey while running, black on green, white on red.
  While a start is set, tmux appends the seconds since it at each refresh, so the check writes nothing per second.
  The options are display only, like the bar's other marks.
- **The check.**
  `runCheck` sets the mark on the prime's reed session as it starts, as each tier starts and ends, and at its end, through a new `Mark` seam on `checkDeps`.
  `checkMarkOf` builds the text from the record, for example `adopt check: tier0 green 40s · tier1 37s` while tier1 runs, `adopt check green: tier0 40s · tier1 104s · tier2 53s` at the end, or `adopt check failed: …` with every tier and its outcome.
  A failure before any tier ended says so and points at `lym beam status`.
  A mark that fails, for example when the prime has no reed session, is logged and changes nothing.
- **The adopt envelope.**
  When it starts or finds the check, `lym adopt` adds a `message` such as `adopt check started in the prime, tiers 0, 1, 2; the prime's reed status bar shows its progress and result`, or `already running`.
  `suite` gains `tiers`, from the hub's `adopt` gate key, and `already_running`.
  Step 8 of the help names the status bar.

## Design decisions

- The plain line goes into the envelope's `message` field, not after the JSON.
  stdout and stderr share one writer, and callers decode the output as one JSON line.
- "Time spent" is the running time of the tier in flight, with each finished tier's wall time beside it.
  The board entry's own figures, 40 s to 104 s per tier, would round to 0 or 1 in minutes, so the bar shows seconds.
- The mark shows on the prime session's bar only, as the board entry asks, so other sessions' bars stay the same.
- The tiers in `message` are read from the gate config before the check's own hub reconcile runs, so a reconcile that reseeds `adopt` could run other tiers.
  The status bar shows the tiers that actually ran.
  When the config does not read, the tiers are left out and the check still starts.

## Bounds

- If the check is killed before its end, it leaves its last running mark, with growing seconds, until the next check sets a new one or the prime's session ends.
  `lym beam status` still reads that record as `ended without a verdict`.

## Tests

- `TestRunCheckMarks` pins the order and text of the marks for a green run, a failed tier and a failure before any tier.
- `TestRunCheck` also checks that a failing mark changes nothing.
- `TestSetCheckMark` pins the tmux argv for setting a mark with a start, setting one without a start, and clearing it.
- `TestBar_LineTwoShowsTheCheckMarkBeforeTheClock` renders line 2 on a real tmux: no mark, running with its seconds, green, failed and cleared.
- `TestAdopt` and `TestEnsureCheck` cover `message`, `tiers` and `already_running`.
