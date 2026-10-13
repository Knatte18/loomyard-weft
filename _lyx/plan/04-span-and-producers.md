# Card 4 — span-and-producers

**Create:**
- `plan:tools/tokencount#producerTime` -> `type producerTime struct`
- `plan:tools/tokencount#producersOf` -> `func producersOf(record *ShedRecord, span interval, working []interval) []producerTime`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, rows []producerTime, span time.Duration)`
- `plan:tools/tokencount#producerRule` -> `const producerRule = ""`

**Edit:**
- `plan:tools/tokencount#spanOf`
- `plan:tools/tokencount#runTiming`
- `plan:tools/tokencount#timingOf`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#run`
- `plan:tools/tokencount#TestRunTime`
- `tools/tokencount/doc.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`

**Uses:**
- `plan:tools/tokencount#ShedRecord`
- `plan:tools/tokencount#shedEntry`
- `plan:tools/tokencount#covered`

**Intent:**
Take each run's span from its archived shed status and add the Producers tables (Decisions 3, 5 and 6), then rename the report (Decision 10).

Span (Decision 3), in `spanOf`:
- No `Shed` record: the transcript window, as in card 2, with the phrase "the transcript window, first to last line; no shed status was read".
- A record with no entries: the transcript window, with the phrase "the transcript window; the status archived at <tag> has no history entry".
- Otherwise the span ends at the latest `End` among the entries.
  It starts at the record's `Started`, else at `run.First`, else, with neither, at the first entry's `End`; that start is then raised to `NotBefore` when it is earlier, and lowered to the first entry's `End` when it is later.
  The phrase is "from the seed's start to the last history entry, read at <tag>", or "from the earliest transcript line to the last history entry, read at <tag>" when `Started` is zero.
- Card 2's role times and idle already clip to the span, so a reused slug's older sessions before the span start count nowhere.
- No start timestamp is added to the shed history (Decision 6); `Publish` and `Finalize` appear as whole rows.

Producers (Decision 5):
- `producerTime` is one producer name's `entries` count, `elapsed` and `idle` durations.
- `producersOf` walks the entries in recorded order: each owns the interval from the latest `End` among the entries before it (the span's start for the first) to its own `End`, empty when its own `End` is not later.
  It sums per producer name, in first-appearance order, the entry count, the intervals' lengths as `elapsed`, and each interval's length less `covered(working, interval)` as `idle`, where `working` is the run's intervals from card 2.
  The elapsed values sum to the span.
- `runTiming` gains `producers []producerTime`, nil when `spanOf` used the transcript window; `timingOf` fills it.
- `writeProducers` prints `| producer | entries | elapsed | idle | share |` and one row per producer, `share` being `100 × elapsed / span` as `%.1f%%`, 0 when the span is 0.
- `Report.WriteMarkdown` prints, after a run's `Span` line and only when it has producers, `Producers, by shed history entry:` and its table.
  In "All runs", after the time paragraph and only when some run has producers, it prints `Producers, summed over the <n> runs with shed history:` and the sum per producer name, in first-appearance order over the runs as listed, its share dividing by those runs' summed spans.
  `producerRule` follows it: a row's elapsed is the gap between finish times, each history entry owning the time from the latest finish before it, or the span's start, to its own finish, so it includes any wait before the row started; its idle is the part of that time no session of the run is in a turn; elapsed sums to the span.

Report and docs (Decision 10):
- `run` prints the title `# Token usage and time by role`; the output path `.scratch/token-usage-by-role.md` stays.
- `doc.go`'s summary sentence names token use and time; its time paragraph now says the span runs from the seed's `started_at`, or the earliest transcript line when the seed has none, bounded by the slug's next-older archive tag and the first history entry, to the latest history entry's end, a folded entry ending at its `last_at`, and falls back to the transcript window with no archived status; and a new paragraph describes the Producers tables.
- The launchers' description comments in `tokencount.sh` and `tokencount.cmd` say the report holds token use and time per role, the idle time and the Producers tables.

Tests: `TestRunTime` keeps card 2's report, whose runs carry no record, and asserts that its `gap` section holds `no shed status was read` and no `Producers` line.
It adds a second report over five more scenarios, each counted with `CountRun` and given a hand-built `ShedRecord`, with `Producers` table rows asserted as exact lines in the run's section:
- `status`: a `burler` session user 10:10, `end_turn` 10:20, user 10:50, `end_turn` 11:00, and an older `burler` session user 08:00, `end_turn` 08:30; record `Started` 10:00 and entries Discuss 10:30, Plan 10:20, Webster 10:45, Webster 11:00, Publish 11:15.
  The span line is `Span 1h15m0s, from the seed's start to the last history entry, read at archive/status/abc.`; burler `20m0s` `26.7%`, the older session clipped away; idle `55m0s` `73.3%`.
  Producers: `| Discuss | 1 | 30m0s | 20m0s | 40.0% |`, `| Plan | 1 | 0s | 0s | 0.0% |` (an entry ending before an earlier one), `| Webster | 2 | 30m0s | 20m0s | 40.0% |`, `| Publish | 1 | 15m0s | 15m0s | 20.0% |`, whose elapsed values sum to the span.
- `unstamped-early`: sessions user 09:00, `end_turn` 09:10 and user 10:00, `end_turn` 10:30; record with no `Started`, `NotBefore` 09:30, entry Webster 10:40: `Span 1h10m0s, from the earliest transcript line`, the start raised to the older tag's time; burler `30m0s`.
- `unstamped-late`: session user 10:30, `end_turn` 10:50; record with no `Started` and no `NotBefore`, entries Discuss 10:10, Webster 11:00: `Span 50m0s`, the start lowered to the first entry's end.
- `untimed`: one session whose only lines carry no timestamp, so `run.First` is zero; record with no `Started` and no `NotBefore`, entries Discuss 10:00, Webster 10:30: `Span 30m0s`, the start at the first entry's end.
- `empty-history`: one session and a record with no entries: its section holds `the status archived at archive/empty-history/abc has no history entry` and no `Producers` line.
- "All runs" of the second report holds `Producers, summed over the 4 runs with shed history:` (`empty-history` is not among them), `| Webster | 5 | 3h0m0s | 2h0m0s | 80.0% |`, `| Discuss | 3 | 30m0s | 20m0s | 13.3% |` and `producerRule`.

**ImpactSummary:** Local to `tools/tokencount`: `spanOf`, `runTiming` and `timingOf` have one caller chain, `Report.WriteMarkdown`, and the title change is asserted by no test.

**Commit:** 4: tokencount: span from the shed status, Producers tables, report title
