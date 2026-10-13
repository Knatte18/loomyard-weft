# Card 4 — time-columns

**Create:**
- `plan:tools/tokencount#timeRule` -> `const timeRule = ""`
- `plan:tools/tokencount#producerElapsedRule` -> `const producerElapsedRule = ""`
- `plan:tools/tokencount#RunTiming.add` -> `func (t *RunTiming) add(o RunTiming)`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, rows []ProducerTime, span time.Duration)`

**Edit:**
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, timing RunTiming)`
- `tools/tokencount#TestCountOrch`
- `plan:tools/tokencount#TestRunTime`

**Intent:**
Render the timing (Decisions 4, 5 and 7): `time` and `time share` columns and an `idle` row in every role table, a Producers table per run with a status, and their sums in "All runs".

`writeTable` gains the run's or the total's `RunTiming`.
Its header becomes `| role | sessions | output | cache write | cache read | input | weight | share | time | time share | est. cost | models (messages) |`; rows keep their weight order and `share` keeps its weight meaning.
A role's `time` is its `RunTiming.Roles` entry rounded with `Round(time.Second)` and printed with `time.Duration`'s `String`, and its `time share` is `100 * time / span` as `%.1f%%`, `0.0%` when the span is zero.
The orch row's two time cells are blank.
Below the role rows, and below the orch row when there is one, comes `| idle | | | | | | | | <idle> | <idle share> | | |`, its token cells blank.

`RunTiming.add` sums another run's span length, role times, idle and producers (by producer name, keeping first-appearance order) into a total, with no union across runs; the total's `Span` is a zero-based interval whose length is the summed span, so `writeTable` divides by the sum of the runs' spans.
`Report.WriteMarkdown` builds that total with `add` over `run.Timing()` of every run and passes it to the "All runs" table, then writes `timeRule` under that table, then, when any run has a status, a `### Producers, all runs` table of the summed producers over the summed span of the runs with a status.
Each run's section passes `run.Timing()` to its table; then, with `HasStatus`, writes `### Producers` and `writeProducers`;
with `run.Status` nil it writes the line `No shed status was read: the span is the transcript window, and idle covers only that window.`;
with a status whose history is empty it writes `The shed status at <tag> holds no history: the span is the transcript window, and idle covers only that window.`
`writeProducers` writes `| producer | entries | elapsed | idle | share |`, one row per `ProducerTime`, durations rounded as above and `share` as `Elapsed` over `span`, then `producerElapsedRule` below the table.

`timeRule` states: time is the union of a role's transcripts' turn time clipped to the run's span; idle is the time no session of the run is in a turn, which holds parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push, and orch sessions count toward idle; role time shares can sum past 100% where sessions overlap, a `+sub` row overlapping its parent; the union of all working time plus idle is the span; "All runs" sums each run's figures and spans, taking no union across runs.
`producerElapsedRule` states that a row's elapsed is the gap between finish times in the run's shed history, so it includes any wait before the row started.

Tests:
- `TestCountOrch`'s full-row assertions on the "All runs" and `run-a` tables gain the two time cells after `share` (the burler rows' turn time and time share, the orch rows' blank cells) and assert the `idle` row under the orch row.
  `run-b`'s transcript gains a user line `[Request interrupted by user]` at 11:10, which carries no usage, so every token and charge assertion stands; the orch main message at 11:30 and the sub-agent `agent-b`, wholly charged to `run-b`, stay charged as before, and `run-b`'s table asserts an `idle` row of `50m0s`, the gap after the interrupt, so orch time enters no union.
- `TestRunTime` gains report rows through `Report.WriteMarkdown`:
  - a run with a status: its table's role rows carry time and time share, its `idle` row has blank token cells, and its `### Producers` table rows carry entries, elapsed, idle and share, with `producerElapsedRule` below;
  - two runs: "All runs" carries the summed role times, summed idle and shares over the summed span, `timeRule` and the summed `### Producers, all runs` table;
  - a run without a status: no Producers table and the no-status line;
  - a run whose status holds no history: no Producers table and the empty-history line naming the tag.

**ImpactSummary:** `writeTable` has one caller, `Report.WriteMarkdown`, on this card; the inserted columns break only `TestCountOrch`'s full-row strings, while `TestCountRun`'s and `TestEstimatedCost`'s substring assertions stay adjacent and keep passing.
