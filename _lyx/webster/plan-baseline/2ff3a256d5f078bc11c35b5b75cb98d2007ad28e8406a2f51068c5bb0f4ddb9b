# Card 4 — A run's span and Producers table from its archived shed status

**Create:**
- `tools/tokencount#ArchiveTrees`
- `tools/tokencount#ShedEntry`
- `tools/tokencount#ShedRecord`
- `tools/tokencount#readShedRecord`
- `tools/tokencount#RunTally.useShedRecord`
- `tools/tokencount#ProducerRow`
- `tools/tokencount#RunTally.Producers`
- `tools/tokencount#writeProducers`

**Edit:**
- `tools/tokencount#RunTally`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount/count_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shedrun#TreeRunFileRels`
- `internal/shedrun#ParseSeed`
- `internal/shedengine#Status`
- `internal/shedengine#HistoryEntry`
- `internal/gitrepo#Repo.FileAtRevision`
- `internal/gitrepo#ErrPathNotAtRevision`
- `tools/tokencount#covered`

**Intent:**
Read a finished run's seed and shed status from its newest archive tag, set the run's span from them, and split the span by history entry into a Producers table (decision record, Decisions 3, 5, 6 and 9).
Card 5 calls `readShedRecord` and `useShedRecord` from `run`; this card adds them and the report output, tested over an in-memory tree.

`ArchiveTrees` declares the one method `FileAtRevision(rev, relPath string) ([]byte, error)`, which `*gitrepo.Repo` satisfies.
`ShedEntry` is one history entry's `Producer` and parsed `End`: its `LastAt` when `Repeats` > 0, else its `At`.
`ShedRecord` holds the `Tag` read, `Older` (the next-older tag's `Committed`, zero when the slug has one tag), the seed's parsed `StartedAt` (zero when the seed or its field is absent) and the `Entries` in recorded order.

`readShedRecord` takes a slug's tags newest first, as `archivedSlugs` orders them, and reads `tags[0].Commit`.
It returns nil and no error when `tags` is empty, when `status.json` is absent at the tag (`gitrepo.ErrPathNotAtRevision`), or when the status's history is empty, since a run with no history entry has no span end; each takes the transcript-window span card 2 set.
It names its two files through `shedrun.TreeRunFileRels(slug)`.
That call's run-id error is wrapped with the slug, the tag's `Name` and both files, `seed.json` and `status.json`, in the shape of the decode errors below (Decision 9), and is a fail-loud wiring guard no test reaches: `archivedSlugs` keys tags by the segment between two slashes, so no tagged slug carries a separator.
It decodes `status.json` into `shedengine.Status` with `encoding/json` and the seed, when present, through `shedrun.ParseSeed`, and parses as RFC 3339 the seed's `started_at` when present, every entry's `at`, and an entry's `last_at` only when its `Repeats` > 0, since `last_at` is omitted on an unfolded entry.
A `seed.json` absent at the tag (`gitrepo.ErrPathNotAtRevision`) is no seed, not an abort: the record keeps a zero `StartedAt`.
A seed or status that fails to decode, or a timestamp in either that fails to parse, aborts with an error naming the slug, the tag's `Name` and the file, `seed.json` or `status.json`.

`RunTally` gains `Shed *ShedRecord`, nil when no shed status was read.
`useShedRecord` stores the record and sets `Span`: `To` is the latest `End` among the entries; `From` is `StartedAt`, or the run's `First` when that is zero, raised to `Older` when it is earlier, then lowered to the first entry's `End` when it is later (Decision 3).
`Producers` returns nil for a run with no record.
Otherwise each entry, in recorded order, owns the interval from the latest `End` among the entries before it (the span's `From` for the first) to its own `End`, empty when its own `End` is not later; per producer name it sums the entries, the elapsed time of those intervals and their idle time, each interval's length minus `covered(run.Working, interval)`.
Rows sort by elapsed, longest first, ties by name.
Elapsed over every row sums to the span.

`ProducerRow` holds `Producer`, `Entries`, `Elapsed` and `Idle`.
`writeProducers` prints a `### Producers` table, `| producer | entries | elapsed | idle | share |`, durations rounded to the second, share as elapsed over `span` in the `%.1f%%` form, followed by one line saying a row's elapsed is the gap between finish times, so it includes any wait before the row started, and `Publish` and `Finalize` record no inner breakdown.
`WriteMarkdown` prints it after each run's role table for a run with a record; a run without one gets the line `No shed status was read, so the span is the transcript window and no Producers table is printed.` instead.
The "All runs" section prints the Producers table summed per producer name over the runs with a record, its share over the sum of those runs' spans, and prints none when no run has a record.

Tests: a new table test `TestShedRecordSpanAndProducers` in `count_test.go` over `readShedRecord`, `useShedRecord`, `Producers` and `WriteMarkdown`, with a map-backed fake `ArchiveTrees` keyed by commit and path and runs built by `CountRun` over synthetic, timestamped transcripts:
- a seed with `started_at`, an unfolded entry with no `last_at`, which reads without error, a folded entry ending at its `last_at` and an entry that finishes earlier than one before it: the span runs from `started_at` to the latest end, the out-of-order entry's interval is empty, elapsed sums to the span, each producer's idle is exact and `Idle()` is the span minus the working union;
- a seed without `started_at`: the span starts at the earliest transcript line; with an older tag committed after that line it starts at the older tag's date; with a first entry ending before that line it starts at that end;
- a tag holding `status.json` and no `seed.json` gives a record whose span starts at the earliest transcript line;
- a slug with two tags whose commits hold different statuses reads the newest tag's;
- a status and a seed that fail to decode, a seed whose `started_at` is not RFC 3339, a status entry whose `at` is not and a folded entry (`repeats` > 0) whose `last_at` is not each abort with an error naming the slug, the tag's `Name` and the file;
- no tags, a tag without `status.json` and a status with an empty history each return no record;
- the report prints the Producers table for a run with a record, the no-status line for a run without one, and the summed Producers table in "All runs".

Raise `tools/tokencount`'s row in `cmd/lyx/testdata/test-budget.yaml` by the new top-level test.

**ImpactSummary:** Runs gain an optional shed record that, once card 5 reads it, sets the span the time columns divide by and adds a Producers table per run and in "All runs"; with no record the report is card 2's plus the no-status line.
