# Card 2 — role-working-time

**Create:**
- `tools/tokencount/turns.go`
- `plan:tools/tokencount#interval` -> `type interval struct`
- `plan:tools/tokencount#unionOf` -> `func unionOf(spans []interval) []interval`
- `plan:tools/tokencount#coveredWithin` -> `func coveredWithin(union []interval, from, to time.Time) time.Duration`
- `plan:tools/tokencount#turnLine` -> `type turnLine struct`
- `plan:tools/tokencount#endsTurn` -> `func endsTurn(l line) bool`
- `plan:tools/tokencount#turnIntervals` -> `func turnIntervals(lines []turnLine) []interval`
- `plan:tools/tokencount#RunTally.roleTimes` -> `func (run RunTally) roleTimes() (times map[string]time.Duration, idle, span time.Duration)`
- `tools/tokencount/turns_test.go`
- `plan:tools/tokencount#TestRunTime` -> `func TestRunTime(t *testing.T)`

**Edit:**
- `tools/tokencount#line`
- `tools/tokencount#RoleTally`
- `tools/tokencount#RunTally`
- `tools/tokencount#RunTally.countFile`
- `tools/tokencount#CountRun`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, times map[string]time.Duration, idle, span time.Duration)`
- `tools/tokencount#run`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `tools/tokencount#userTexts`
- `tools/tokencount#eachLine`
- `tools/tokencount#RunTally.observeTime`

**Intent:**
Measure each role's working time and each run's idle time from the transcripts the tool already walks, and print them as new role-table columns and an `idle` row (decision record, Decisions 1, 2, 4 and 7).
In this card every run's span is its transcript window, `First`..`Last`; card 3 replaces it with the archived status span.

`turns.go` holds the time arithmetic:
`interval` is a `From`/`To` pair of `time.Time`.
`unionOf` sorts and merges overlapping or touching intervals, dropping empty ones.
`coveredWithin` is the total length of a union's intervals clipped to `[from, to]`, zero when `to` is not after `from`.
`turnLine` is one user or assistant line's parsed timestamp and whether it ends a turn.
`endsTurn` reports a turn-ending line: an assistant line whose message `StopReason` is non-empty and not `tool_use`, or a user line one of whose `userTexts` starts with `[Request interrupted by user` or with `<local-command-stdout>`.
`turnIntervals` sorts its lines by timestamp, stably, and returns the interval between each pair of consecutive lines unless the first of the pair ends a turn; a gap after a turn-ending line is idle until the next line, and a session that crashes mid-turn counts its gap as working (the record's auto-mode assumption).

`line`'s message struct gains `StopReason string` with JSON key `stop_reason`, so a null decodes as empty.
`RoleTally` gains `Working []interval`, the union of its transcripts' working intervals; `RunTally` gains `Working []interval`, the union over every transcript of the run, and `Span interval`.
`countFile` collects a `turnLine` for each user or assistant line with a parseable timestamp in the same `eachLine` pass, then appends that transcript's `turnIntervals` to the role tally's and the run's `Working`.
A sub-agent transcript is counted under `<role>+sub` already, so its time lands in that row, and its intervals join the run's `Working`.
`CountRun` unions each role's and the run's `Working` once all transcripts are read, and sets `Span` to `First`..`Last`.
Orch transcripts are read only by `CountOrch` and never reach a run's `Working`, so orch time, an orch sub-agent charged wholly to a run included, counts as the run's idle.

`roleTimes` returns, within `Span`, each role's `coveredWithin` of its `Working`, the idle time (span minus the run's covered `Working`) and the span length.
`writeTable` takes those figures and adds `time` and `time share` columns after `share`: a duration rounded to the second, and the duration as a percentage of `span` (`n/a` for a zero span).
Rows keep their weight order.
Below the role rows it writes an `idle` row with time and time share and blank token, weight, share, cost and models cells, then the orch row, whose time cells are blank.
`Report.WriteMarkdown` calls `roleTimes` per run, and for "All runs" sums each run's role times, idle and span, taking no union across runs.
Each role table is followed by its span, and "All runs" states that idle is time no session of the run is in a turn (parent review, PR-Gate, gate-slot waits, a parked driver, Go-only work), that role time shares can sum past 100% where sessions overlap (a `+sub` row overlaps its parent), and that the union of working time plus idle is the span.
`run`'s report title becomes `# Token usage and time by role`; the old title is asserted by no test (grepped), and the output path stays.
The package doc in `main.go` gains a paragraph defining working time as turn time, the union per role and per run, idle, the two new columns and the `idle` row, as this card measures them.

`TestRunTime`, in the new `turns_test.go`, is a unit-tier table over synthetic transcripts written with the existing `writeLines` helper and `CountRun`, then `Report.WriteMarkdown`, asserting exact durations and exact table rows:
two turns split by an `end_turn` assistant line and a long gap count both turns and leave the gap out;
a user line holding the interrupt marker makes the gap after it idle;
a `<local-command-stdout>` user line between turns adds no working time after it;
a `tool_use` assistant line followed later by its tool result counts that wait as working;
two overlapping sessions of one role count their union, not their sum, and a sub-agent transcript counts under `<role>+sub`;
the run's idle row equals the span minus the union of all working intervals, and a report with an `OrchTally` charging the run leaves the idle row unchanged and the orch row's time cells blank;
"All runs" sums two runs' times, idle and spans.
The new top-level test pushes `tools/tokencount` past its `test-budget.yaml` row, so the row rises to the package's new count.

**ImpactSummary:** Adds working-time and idle measurement to `tools/tokencount`'s transcript pass and two columns plus an `idle` row to every role table; token figures, the orch section and the calibration sections are unchanged.

**Commit:** 2: tokencount: working time and idle per role
