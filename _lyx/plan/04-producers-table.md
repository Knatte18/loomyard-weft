# Card 4 — producers-table

**Create:**
- `plan:tools/tokencount#ProducerTime` -> `type ProducerTime struct`
- `plan:tools/tokencount#RunTally.Producers` -> `func (run RunTally) Producers() []ProducerTime`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, producers []ProducerTime, span time.Duration)`

**Edit:**
- `tools/tokencount#Report.WriteMarkdown`
- `plan:tools/tokencount#TestReadArchivedRun`

**Uses:**
- `plan:tools/tokencount#ArchivedRun`
- `plan:tools/tokencount#RunTally.useArchive`
- `plan:tools/tokencount#union`
- `plan:tools/tokencount#clip`
- `plan:tools/tokencount#length`

**Intent:** Split each archived run's span by shed history entry into a per-run "Producers" table and its sum in "All runs" (Decision 5 of the decision record).

In `tools/tokencount/archive.go`:
- `ProducerTime` is one producer name's row: `Producer`, `Entries`, `Elapsed` and `Idle`.
- `Producers` returns nil for a run with no `Archive`.
  Otherwise each history entry, in recorded order, owns the interval from the latest end among the entries before it (the span's start for the first entry) to its own end, empty when its own end is not later.
  Per producer name it counts the entries and sums the intervals' lengths as `Elapsed` and, as `Idle`, the part of each interval no working interval of the run covers (the interval's length minus `length(clip(union(<every role's Working>), <interval>))`).
  Rows sort by `Elapsed` descending, then by name, so the slowest producer leads.
  The intervals tile the span, so `Elapsed` sums to the span's length.
- `writeProducers` writes a `### Producers` heading and the table `| producer | entries | elapsed | idle | share |`, durations rounded to the second, share the elapsed over `span` as `%.1f%%`.

`WriteMarkdown` writes each archived run's Producers table after its role table; a run with no `Archive` gets none, its span line already saying no shed status was read.
"All runs" gets a Producers table after its role table when at least one run has an `Archive`: per producer name the runs' entries, elapsed and idle summed, sorted as above, with share over the summed spans of those runs, so the shares sum to 100%.
The note paragraph below the "All runs" table gains one sentence: a Producers row's elapsed is the gap between shed finish times, so it includes any wait before the row started, and its idle is the part of that gap no session of the run is in a turn.

`TestReadArchivedRun` gains assertions:
- a history with entries recorded out of finish order, one folded, and two entries of one producer gives exact `Producers` rows whose `Elapsed` sums to the span, with an entry whose end is not later than an earlier one's owning nothing;
- each row's `Idle` is the part of its intervals the run's synthetic working intervals leave uncovered;
- the report of two archived runs and one without an archive carries each archived run's Producers table, none for the third, and an "All runs" Producers table whose cells sum the two runs.

**ImpactSummary:** `WriteMarkdown` gains the Producers tables and one note sentence; with no caller of `useArchive` until card 5, no run has an archive, so the note sentence is the report's only visible change.

**Commit:** 4: tokencount: Producers table from the shed history
