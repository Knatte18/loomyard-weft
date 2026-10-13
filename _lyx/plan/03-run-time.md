# Card 3 — Run span, role times, idle and producer times

**Create:**
- `plan:tools/tokencount#ArchivedRun` -> `type ArchivedRun struct`
- `plan:tools/tokencount#RunTime` -> `type RunTime struct`
- `plan:tools/tokencount#ProducerTime` -> `type ProducerTime struct`
- `plan:tools/tokencount#MeasureRunTime` -> `func MeasureRunTime(run RunTally, archived *ArchivedRun) (RunTime, error)`
- `plan:tools/tokencount#TestMeasureRunTime` -> `func TestMeasureRunTime(t *testing.T)`

**Edit:**
- `tools/tokencount#RunTally`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Decisions 2 to 6: from a counted run and its archived shed records, compute the run's span, each role's time, the run's idle time and the Producers rows.
The code lives in `tools/tokencount/span.go`, beside card 2's interval helpers.

`ArchivedRun` is what card 5 reads from the weft: `Tag string` (the archive tag name), `Seed shedrun.Seed`, `Status shedengine.Status`, and `Floor time.Time`, the committer date of the slug's next-older archive tag, zero when there is none.
`RunTime` holds `From, To time.Time`, `HasStatus bool`, `NoHistory bool`, `Roles map[string]time.Duration`, `Idle time.Duration` and `Producers []ProducerTime`.
`ProducerTime` holds `Producer string`, `Entries int`, `Elapsed, Idle time.Duration`.
`RunTally` gains `Time RunTime`; `run` (card 5) stores `MeasureRunTime`'s result there and `Report.WriteMarkdown` (card 4) reads it.

`MeasureRunTime`:
- An entry's end is its `LastAt` when `Repeats > 0`, else its `At`, both parsed as RFC 3339.
- With `archived` non-nil and at least one history entry: `To` is the latest entry end; `From` is the seed's `StartedAt` parsed, or `run.First` when `StartedAt` is empty; `From` is then raised to `Floor` when `Floor` is later and lowered to the first history entry's end when that is earlier; `HasStatus` is true.
- With `archived` nil, or a status with no history entry: `From, To` are `run.First, run.Last`, `HasStatus` is false and `Producers` is empty (Decision 3's no-status fallback).
  `NoHistory` is true only for an archived status with no history entry, so card 4 says a status was read but held no history rather than that none was read.
- `Roles[role]` is `covered(union(run.Working[role]), From, To)` for every role in `run.Working`; `Idle` is `To - From` minus `covered(union(run.Busy), From, To)`.
- Producers (Decision 5): entries are walked in recorded order with `latest` starting at `From`; an entry whose end is after `latest` owns `[latest, end)` and moves `latest` to its end, any other owns an empty interval.
  Per producer name, in order of first appearance: `Entries` counts its entries, `Elapsed` sums its intervals and `Idle` sums the part of them `union(run.Busy)` does not cover.
  `Elapsed` over all producers sums to `To - From`.
- An entry time or a `StartedAt` that does not parse returns an error naming the run's slug, `archived.Tag` and the file, `status.json` or `seed.json` (Decision 9's abort on a record that fails to decode).
  It is reached by a test row whose archived status carries `"at": "yesterday"` and by one whose seed carries `"started_at": "yesterday"`.

New top-level `TestMeasureRunTime`, table-driven over `RunTally` values built in the test with `Working`, `Busy`, `First` and `Last` set directly and `ArchivedRun` literals, asserts exact `From`, `To`, `Roles`, `Idle`, `HasStatus`, `NoHistory` and `Producers`:
- a seed `started_at` and a folded entry (`repeats` 2, `last_at` later than the next entry's `at`): the span runs from `started_at` to the folded entry's `last_at`; idle is the span minus the union of `Busy`;
- entries out of order (a later entry's end before an earlier one's): that entry's elapsed is zero and `Elapsed` sums to the span; each producer's idle is the part of its intervals no busy interval covers;
- a role whose working intervals extend before `From` and after `To` is clipped to the span;
- a seed without `started_at` starts at `run.First`; with `Floor` after `run.First` it starts at `Floor`; with the first entry's end before `run.First` it starts there;
- a seed with `started_at`: with `Floor` after `started_at` it starts at `Floor`; with the first entry's end before `started_at` it starts there, so both bounds hold for the seeded start too;
- `archived` nil: the span is `run.First..run.Last`, `HasStatus` false, `NoHistory` false, no producers, role times kept;
- an archived status with an empty history: the span is `run.First..run.Last`, `HasStatus` false, `NoHistory` true, no producers;
- an unparseable `at`: an error whose text holds the slug, the tag and `status.json`;
- an unparseable `started_at`: an error whose text holds the slug, the tag and `seed.json`.
The `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` rises by one.

**ImpactSummary:** one added field on `RunTally` and a pure function over it; no existing caller or output changes until card 4 renders it.

**Commit:** 3: tokencount: run span, role times, idle and producer times
