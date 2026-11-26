```yaml
lens: fix-way-forward
outcome: reported
```

Task: check the ways forward this fix introduced (refusals, waits, skips and hand-offs) can be reached and lead to progress.

Ways forward checked:
1. **Config refusal of a negative `gate_hold_factor` or a `gate_hold_min` below one (D6).** It is reachable: `LoadConfig` already refuses keys with a `lym config beam --set <key>=<value>` way forward (`internal/dobbyengine/config.go:120-122`). No finding.
2. **Failed `Pool.Holders` read skips the tick with one Warn per episode (D6).** It recovers on the next tick's read. No finding.
3. **A never-read `run_watch` is not judged until a read succeeds (D2).** See finding 2.
4. **A cancel that removes its records is reported by the cancel's own notice (D3).** See finding 1.
5. **The watch drops an entry for an unlisted run that owes no line (D4).** Owed lines still leave through D5's three-failure and one-hour drops. No finding.

Findings:

1. **LOW**: D3, "The cancel's own notice reports that cancel, and the watch posts no line for it."
   - Summary: the hand-off has no retry. If the cancel notice's post fails, the orch hears nothing about a records-removing cancel, and the record presents the cancel notice as the guaranteed reporter.
   - Evidence: `postNotice` only warns on a failed post and keeps no owed line (`internal/battencli/cancel.go:829-836`). The run has already left `dobbyParentRuns`, so no later step reports it. The sentence should name this bound: a failed post is only a Warn in the cancel's output.

2. **LOW**: D2, "A run whose `run_watch` has never read is not judged: its reading neither posts nor becomes the memory until a read succeeds."
   - Summary: if the read keeps failing, the run stays silent with no signal to the orch or the log, and the record names none.
   - Evidence: the landing notice it cites logs each failed read once per episode (`internal/dobbyengine/landnotice.go:84`, `d.landNoticeLookup.Fail`). D2 states no such Warn for the `run_watch` read, so a persistently unreadable setting leaves the run unwatched with nothing pointing at the cause.

Tool calls:
- Ran `sed` on `internal/battencli/cancel.go:825-860` and `internal/dobbyengine/config.go:95-140` to see `postNotice`'s failure handling and `LoadConfig`'s existing refusal way forward. My context held only those functions' names, not their bodies.
