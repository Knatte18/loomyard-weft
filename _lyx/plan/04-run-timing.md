# Card 4 — run-timing

**Edit:**
- `tools/tokencount#RunTally`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:tools/tokencount#RunTiming` -> `type RunTiming struct`
- `plan:tools/tokencount#ProducerRow` -> `type ProducerRow struct`
- `plan:tools/tokencount#entryEnd` -> `func entryEnd(e shedengine.HistoryEntry) (time.Time, error)`
- `plan:tools/tokencount#RunTally.resolveTiming` -> `func (run *RunTally) resolveTiming(archived *ArchivedRun) error`
- `plan:tools/tokencount#TestRunTiming` -> `func TestRunTiming(t *testing.T)`

**Uses:**
- `plan:tools/tokencount#ArchivedRun`
- `plan:tools/tokencount#Interval`
- `plan:tools/tokencount#clip`
- `plan:tools/tokencount#total`
- `internal/shedengine#HistoryEntry`

**Intent:** Turn a run's working intervals and its archived status into the run's span, per-role time, idle and Producers rows, Decisions 2 to 5, in `timing.go`.
`RunTiming` holds `Span Interval`; `HasStatus`, with `NoStatus` the one-line reason its section prints when false; `Roles map[string]time.Duration`, each role's `Working` clipped to the span and totalled; `Idle`, the span's length minus the total of the run's `Working` clipped to the span; and `Producers []ProducerRow`, empty without status.
`ProducerRow` is one producer name's `Entries`, `Elapsed` and `Idle`.
`RunTally` gains `Timing RunTiming`.
`entryEnd` parses an entry's `LastAt` when `Repeats` is over zero, else its `At`, as RFC 3339.
`resolveTiming` sets `run.Timing`.
With `archived` nil or `HasStatus` false, the span is the transcript window `First`..`Last`, `NoStatus` says no shed status was read, and no rows are built; a status whose history is empty is treated the same, with `NoStatus` saying the archived status holds no history entry.
Else the span end is the latest `entryEnd` over the history; the span start is the seed's `StartedAt` when set, else `First`, then raised to `OlderTagCommitted` when that is later and lowered to the first entry's end when that is earlier.
The Producers rows walk the history in recorded order with a cursor starting at the span start: each entry owns the interval from the cursor to its own end, empty when its end is not later, and the cursor becomes the later of the two, so the intervals tile the span and `Elapsed` sums to its length whatever the entries' order; per producer name, `Entries` counts entries, `Elapsed` sums intervals, and `Idle` is `Elapsed` minus the total of the run's `Working` clipped to those intervals.
Rows are ordered by `Elapsed` descending, then name, so the slowest producer reads first.
A `started_at`, `at` or `last_at` that fails to parse is an error naming the slug, the archived tag and the file, the same shape as card 3's decode error, which is the branch an archived file with a malformed timestamp reaches once it has decoded.
`TestRunTiming`, in a new `timing_test.go`, builds `RunTally` values with `Working` intervals and `ArchivedRun` values in memory and asserts exact durations and rows: a role's time is its clipped union and two roles' times may sum past the span; an orch-shaped interval absent from `Working` leaves its time in `Idle`; the span runs from `started_at` to the latest end, a folded entry ending at `last_at`; a seed without `started_at` starts at `First`, raised to the older tag's time and lowered to the first entry's end; out-of-order entries still sum `Elapsed` to the span and each row's `Idle` is the uncovered part; a nil `archived` and an empty history take the transcript window with their `NoStatus` text and no rows; a malformed `at` returns the error naming slug, tag and file.
One new top-level test, so the `tools/tokencount` row in `cmd/lyx/testdata/test-budget.yaml` rises by one.

**ImpactSummary:** `RunTally` gains one field; nothing calls `resolveTiming` yet, so the report is unchanged until card 6 wires it.
