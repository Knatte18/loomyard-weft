# Card 5 — report time columns

**Create:**
- `plan:tools/tokencount#timeColumnsNote` -> `const timeColumnsNote`
- `plan:tools/tokencount#producersNote` -> `const producersNote`
- `plan:tools/tokencount#noShedNote` -> `const noShedNote`
- `plan:tools/tokencount#sumTimings` -> `func sumTimings(timings []RunTiming) RunTiming`
- `plan:tools/tokencount#writeProducers` -> `func writeProducers(w io.Writer, timing RunTiming)`
- `plan:tools/tokencount#TestReportTimeColumns` -> `func TestReportTimeColumns(t *testing.T)`

**Edit:**
- `tools/tokencount#Report`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#writeTable` -> `func writeTable(w io.Writer, roles map[string]*RoleTally, orch *OrchCharge, timing RunTiming)`
- `tools/tokencount#TestCountOrch`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** The role tables gain the run's time, beside tokens, and the report gains the Producers table.
The doc comments of `Report` and `WriteMarkdown` are updated to name the time columns, the notes and the Producers tables.
`Report` gains `Shed map[string]*ShedRecord`, keyed by slug, a struct field, so it takes no arrow; a slug with no entry reads as no record.
`writeTable` takes the table's `RunTiming` and writes the header `| role | sessions | output | cache write | cache read | input | weight | share | time | time share | est. cost | models (messages) |`, with `time` and `time share` after `share`.
`share` keeps its weight meaning and the rows keep their weight order; a role's `time` is `timing.Roles[role]` rounded to the second, printed as `time.Duration`, and its `time share` is that over `timing.Length` as `%.1f%%`, `0.0%` when the length is zero.
The orch row keeps its weight, share and cost, and its two time cells are blank.
After the orch row, an `idle` row gives `timing.Idle` and its share of the length in the two time columns, with every token cell blank.
`WriteMarkdown` computes each run's `RunTiming` with `run.Timing(r.Shed[run.Slug])`, passes it to that run's `writeTable`, and under it writes `noShedNote` when `ShedRead` is false, else `writeProducers`: a table `| producer | entries | elapsed | idle | share |`, `share` being `Elapsed` over `Length`.
The "All runs" table takes `sumTimings` of the runs' timings: role times, idle and lengths summed, each computed within its run with no union across runs, and producer rows summed by name over the runs that read a shed status; `ShedRead` is true when any run read one.
Under the "All runs" table it writes `timeColumnsNote`, then `producersNote` and the summed Producers table when any run read one, else `noShedNote`.
The three constants hold the report's definitions: `timeColumnsNote` says a role's time is the union of its sessions' turn time, that role shares can sum past 100% where sessions overlap and a `+sub` row overlaps its parent, that the union of working time plus idle is the span, and that idle is time no session of the run is in a turn, parent review, PR-Gate, gate-slot waits, a parked driver and Go-only work such as verify and push included; `producersNote` says a row's elapsed is the gap between finish times, so it includes any wait before the row started, and that `Publish` and `Finalize` show their whole interval; `noShedNote` says no shed status was read, so the span is the transcript window.
`TestCountOrch` asserts exact role and orch row strings, which the two new columns and the blank orch time cells change, so the card recomputes those expected cells; those runs carry no `ShedRecord`, so their time cells follow the transcript-window fallback from the test's `timedAssistant` timestamps, and a run with one line prints `0s` and `0.0%`; every other assertion of the report in `TestCountRun` and `TestEstimatedCost` matches a prefix or an inner cell and stays valid.
`TestReportTimeColumns` is one new top-level test building a `Report` of two runs from synthetic transcripts and `ShedRecord` literals, one with a record and one without, and its assertions are:
- the header and an exact role row carry `time` and `time share`, and an exact `idle` row follows the roles;
- an orch session in a prime project directory, counted with `CountOrch` and charged wholly to a run, leaves that run's `idle` row at the span minus the worktree transcripts' union, so parent review counts as idle;
- the run with a record prints the Producers table with exact rows, `elapsed` summing to the span and `share` as elapsed over the span, and the run without prints `noShedNote` and no Producers table;
- "All runs" sums the role times, the idle and the lengths of both runs, and its Producers table sums the entries by name;
- each note appears once where described, found by its key definition (`past 100%`, `gap between finish times`, `No shed status was read`).
The test lives in `count_test.go` and runs with `t.Parallel()`; the orch row's blank time cells are pinned by `TestCountOrch`'s recomputed exact orch row.
The new top-level test takes the `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` from 16 to 17.

**ImpactSummary:** `writeTable` has one caller, `Report.WriteMarkdown`, and `Report` literals in `TestCountRun`, `TestCountOrch` and `TestEstimatedCost` compile unchanged; only `TestCountOrch`'s exact row strings need the new cells.
