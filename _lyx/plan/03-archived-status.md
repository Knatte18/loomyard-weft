# Card 3 — Archived shed status sets the span and the Producers table

**Create:**
- `plan:tools/tokencount#ArchiveTag` -> `type ArchiveTag struct`
- `plan:tools/tokencount#StatusRecords` -> `type StatusRecords interface`
- `plan:tools/tokencount#ProducerTime` -> `type ProducerTime struct`
- `plan:tools/tokencount#attachStatus` -> `func attachStatus(run *RunTally, hub string, tags []ArchiveTag, records StatusRecords) error`
- `plan:tools/tokencount#historyEnds` -> `func historyEnds(history []shedengine.HistoryEntry) ([]time.Time, error)`
- `plan:tools/tokencount#producerTimes` -> `func producerTimes(history []shedengine.HistoryEntry, ends []time.Time, span Interval, working []Interval) []ProducerTime`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, producers []ProducerTime, span time.Duration)`
- `plan:tools/tokencount#producerRule` -> `const producerRule = "A producer's elapsed time is the gap between its history entries' finish times and the finish before each, so it includes any wait before the row started; the elapsed column sums to the span, and a producer's idle is the part of its elapsed time no session of the run works in."`
- `plan:tools/tokencount#noStatusNote` -> `const noStatusNote = "No shed status was read: the span is the transcript window, so idle covers only that window."`
- `plan:tools/tokencount#TestAttachStatus` -> `func TestAttachStatus(t *testing.T)`

**Edit:**
- `tools/tokencount#RunTally`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#archivedSlugs` -> `func archivedSlugs(refs string) map[string][]ArchiveTag`
- `tools/tokencount#finishedRuns` -> `func finishedRuns(weft string) (map[string][]ArchiveTag, error)`
- `tools/tokencount#recentRuns` -> `func recentRuns(projects, hub, prime string, finished map[string][]ArchiveTag, last int) ([]string, error)`
- `tools/tokencount#run`
- `tools/tokencount#TestArchivedSlugs`
- `tools/tokencount#TestRecentRuns`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shedrun#SeedTreeRel`
- `plan:internal/shedrun#StatusTreeRel`
- `plan:internal/shedrun#DecodeSeed`
- `plan:tools/tokencount#Interval`
- `plan:tools/tokencount#coveredWithin`
- `plan:tools/tokencount#RunTally.settleTimes`
- `plan:tools/tokencount#userAt`
- `plan:tools/tokencount#assistantAt`
- `internal/shedengine#Status`
- `internal/shedengine#HistoryEntry`
- `internal/gitrepo#ErrPathNotAtRevision`
- `tools/tokencount#fakeHistory`
- `tools/tokencount#CountRun`
- `tools/tokencount#writeLines`

**Intent:**
Decisions 3, 5, 6 and 8's tag rules, and the in-flight bound: a finished run's span comes from its archived shed status, and the Producers table splits it by history entry.
The new members live in a new file `tools/tokencount/status.go` with a one-line file comment; `archivedSlugs`, `finishedRuns` and `recentRuns` stay in `count.go`.
No shed history shape changes (Decision 6): each entry's finish time is the only time read, and `Publish` and `Finalize` appear as their own whole intervals.

Archive tags.
`ArchiveTag` is `Name` (`archive/<slug>/<sha12>`), `SHA` (the full commit SHA) and `Committed` (the commit's committer time).
`finishedRuns` runs `gitexec.Run` with `for-each-ref --format=%(refname) %(objectname) %(committerdate:unix) refs/tags/archive/` in the weft (archive tags are lightweight tags on commits, `internal/fabricengine/archive.go`), and `archivedSlugs` parses each line into a tag under its slug, skipping a line that does not have the three fields, a `refs/tags/archive/<slug>/<sha>` name or an integer time, and orders each slug's tags newest committer time first, ties by name.
`recentRuns` takes that map and keeps a slug with at least one tag; `run` passes `finishedRuns`'s result through unchanged, so with no slugs named it behaves as before.

`attachStatus` sets a run's span from its archived status:
1. When the directory `<hub>/<run.Slug>` exists the run is in flight (teardown removes the worktree), and it returns with no status read.
2. With no tags it returns with no status read.
3. It reads `tags[0]`, the newest, at `shedrun.StatusTreeRel(run.Slug)` through `records.FileAtRevision(tag.SHA, rel)`; `StatusRecords` is that one method, which `*gitrepo.Repo` and the test's `fakeHistory` both have.
   `gitrepo.ErrPathNotAtRevision` means no status was read.
4. It decodes the bytes into `shedengine.Status` with `json.Unmarshal`, and `historyEnds` parses each entry's end, its `LastAt` when `Repeats > 0`, else its `At`, as RFC 3339; an empty history means no status was read.
5. It reads the seed at `shedrun.SeedTreeRel(run.Slug)` the same way; an absent seed has no start, and a present one goes through `shedrun.DecodeSeed`, its non-empty `StartedAt` parsed as RFC 3339.
6. The span ends at the latest entry end.
   It starts at the seed's `StartedAt`, else at `run.First`, and the start is then raised to the committer time of `tags[1]`, the slug's next-older tag, when there is one, and lowered to the first entry's end when later; a zero start (no seed start, no transcript timestamp) becomes the first entry's end.
7. It sets `run.Span`, `run.StatusTag` to the tag's name and `run.Producers` to `producerTimes`, then calls `run.settleTimes()` so the role times and idle are clipped to the new span.

Every failure past step 2 that is not "no status read", a run id `StatusTreeRel` or `SeedTreeRel` refuses, a read error, a status or seed that fails to decode, or an unparseable time, returns an error naming the slug, the tag's name and the file, with the file from the tree path or, on a refused run id, from `shedrun`'s error; `run` returns it and the report is aborted.
A run with no status read keeps the transcript window `CountRun` set as its span.

`RunTally` gains `StatusTag string`, empty when no status was read, and `Producers []ProducerTime`.
`ProducerTime` is `Producer`, `Entries int`, `Elapsed` and `Idle time.Duration`.
`producerTimes` walks the history in recorded order: entry i owns the interval from the latest end among entries before it (the span start for the first) to its own end, empty when its end is not later; per producer name it sums the entries, the intervals' durations as `Elapsed`, and as `Idle` each interval's duration minus `coveredWithin(working, interval)`, with `working` the run's `Working`.
Rows are ordered by `Elapsed` descending, then name.
`Elapsed` sums to the span, out-of-order entries included, since the latest end before an entry never decreases and the last reaches the span end.

`writeProducers` writes `| producer | entries | elapsed | idle | share |` rows, durations rounded to the second, share `100 * Elapsed / span` as `%.1f%%`.
`WriteMarkdown` prints, in each run's section after its role table, `noStatusNote` when `StatusTag` is empty, else a line `Span <duration> from shed status at <tag>.` and a `### Producers` table.
"All runs" gains, after the role table and `timeRule`, a `### Producers` table summing each producer's entries, elapsed and idle over the runs with a status, its share over the sum of those runs' spans, followed by `producerRule`; it is left out when no run has a status.

Tests:
- `TestAttachStatus` is table-driven over one run counted by `CountRun` from a burler transcript of two turns, 10:00 to 10:10 and 10:40 to 10:50 (`userAt`, `assistantAt` with `end_turn`), and a `fakeHistory` holding the files at each tag's SHA, keyed by `shedrun.StatusTreeRel`'s and `SeedTreeRel`'s paths.
  Rows:
  - the base row: seed `started_at` 09:50 and history Plan at 10:20, Discussion at 10:05 (out of order, so empty), Webster at 10:30 folded with `repeats` 2 and `last_at` 11:00: span 70m, burler 20m, idle 50m, Producers Webster 40m elapsed with 30m idle, Plan 30m with 20m idle, Discussion 1 entry and 0s, elapsed summing to the span, and the rendered run section holds the span line and the `### Producers` rows;
  - no seed file and no older tag: the span starts at the transcript's 10:00;
  - a seed `started_at` of 09:00 and a next-older tag committed at 10:02: the span starts at 10:02;
  - a seed `started_at` of 10:30 and a first entry ending 10:20: the span starts at 10:20;
  - a seed without `started_at` and a first entry ending 09:55: the span starts at 09:55, not the transcript's 10:00;
  - two tags whose older one holds a status naming another producer: only the newest tag's producers appear and `StatusTag` names it;
  - the directory `<hub>/<slug>` exists, with tags present: no status read, span 10:00 to 10:50;
  - no tags: no status read and burler's 20m kept;
  - a tag whose tree has no status, and a status whose history is empty: no status read;
  - a status that is not JSON, a status whose entry's `at` does not parse, and the base row's status beside a seed with an unknown key: an error holding the slug, the tag name and the file's tree path;
  - a slug `a/b`: an error holding the slug, the tag name and `status.json`.

  Every no-status row also asserts its rendered run section holds `noStatusNote` and no `### Producers` table.
  After the rows, one report over the base row's run and the no-tags row's run asserts the "All runs" `### Producers` table holds the base row's producers with shares over the base run's 70m span alone, and the report holds `producerRule`; a report over the no-tags run alone has no `### Producers` table anywhere.
- `TestArchivedSlugs` feeds three-field lines, two tags of one slug out of time order and malformed lines, and asserts each slug's tags newest first with their SHA and time, and the malformed lines skipped.
- `TestRecentRuns` builds its `finished` set as `map[string][]ArchiveTag` with one tag per finished slug; its assertions are unchanged.
- `cmd/lyx/testdata/test-budget.yaml`'s `tools/tokencount` row goes from 13 to 14 for `TestAttachStatus`.

**ImpactSummary:** Local to `tools/tokencount`: the three re-signed functions' callers are `run`, `finishedRuns` and their two tests, all on this card; `attachStatus` has no production caller until card 4, and the budget row rises by one.

**Commit:** 3: tokencount reads a run's archived shed status for its span and Producers table
