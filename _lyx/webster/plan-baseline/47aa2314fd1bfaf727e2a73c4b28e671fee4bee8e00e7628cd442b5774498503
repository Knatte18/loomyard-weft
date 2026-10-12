# Card 14 — report-weeks-and-markdown

**Create:**
- `plan:internal/reportengine#Week` -> `type Week struct`
- `plan:internal/reportengine#ClassCount` -> `type ClassCount struct`
- `plan:internal/reportengine#RenderMarkdown` -> `func RenderMarkdown(r Report) string`
- `//internal/reportengine/weeks_test.go`

**Edit:**
- `//internal/reportengine/doc.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:**
The report gains its per-week aggregates and per-source absent counts (D5), and the markdown rendering of the same data (D12), in new files `weeks.go` and `markdown.go`.
`Compute` fills `Report.Weeks`, one `Week` per ISO week holding a run, a partial run or a landing, ascending, and `Report.Absent`, the number of selected runs lacking each source.

A `Week` holds, per D5's rule that a week sums over the runs holding a source and states how many lack it:
- `Runs`, `Partial`, and `Lacking`, the number of the week's runs lacking each source;
- `WakesPerRun`, the mean wake total over the week's runs holding every wake source, and `WakeRuns`, how many those are; the `land-ready-tasks` source, absent for every run by D6, takes no part in that test, else no week would ever hold a mean;
- `Wakes`, the total per kind over the runs holding the kind's source, and `Automatic` and `Paused` totals;
- `StopClasses` and `FailureClasses`, each a `ClassCount` per class with its stop count and the number of the week's runs it recurred in, over the runs holding the halt-note source;
- `Landings`, the count per kind of the week's landings (by the landing's own commit date), `Hand`, and the fresh, aged and no-pre-image counts of the `fix` and of the `unmapped` landings;
- `FixShare` and `UnmappedShare`, fresh over fresh plus aged;
- `Rounds`, per segment the mean rounds over the week's runs holding the segment, how many held it, and the count per outcome;
- `Arbiter`, per verdict word the count and applied count, summed over the week's runs holding the arbiter source;
- `LLMSteps`, per source the sum over the week's runs holding that source, `landing` included;
- `DriveReports`, the sum over the week's runs holding the drive-reports source;
- `Decisions`, per `by` the sum over the week's runs holding the decision record;
- `Alignment`, per point and verdict the count and applied count, summed over the week's runs holding the alignment source.
Each of the last five sums over the runs holding its source, with `Lacking` naming the rest, and is `nil` when no run of the week holds the source.
D12's markdown sections are the four board questions, so these five reach the JSON alone.
Every mean and share is a `*float64`, `nil` (`null` in JSON) when no run or no fix carries it, never 0.
`Week` and `ClassCount` carry snake_case JSON tags like every report type before them (Shared Decision `report-json-shape`), so a source's absent figure encodes as `null`.

`RenderMarkdown` renders a `Report` as one section per board question, from the same fields the JSON carries:
`## Do orch wakes per run fall?` (per week: runs, runs counted, wakes per run, wakes by kind, automatic steps, paused),
`## Do the same stop and failure classes recur?` (per week and class: stops and runs recurred in, failure classes likewise),
`## Does the fresh-code fix share fall?` (per week: landings by kind, hand landings, the fix share and the `unmapped` share with their counts) and
`## Do review rounds per segment fall?` (per week and segment: mean rounds, runs, lacking, outcomes).
A `nil` figure renders as `no data`, and the selection opens the document.

`doc.go` describes the weekly rules and both outputs.

Tests in `weeks_test.go`:
- `TestCompute_Weeks` is table-driven over fake sources, each row asserting the exact `Weeks` and `Absent`:
  a week with one run holding every wake source and one lacking the parent-review source has a wake mean over the first alone, `WakeRuns` 1 and `Lacking["parent-review"]` 1;
  a week whose only run lacks the halt-note source has `nil` `StopClasses` and `FailureClasses`, encoding as `null`, and counts it lacking;
  a week with feature landings and no fix reports `FixShare` `nil`, not 0, and a week with one fresh and one aged fix reports 0.5;
  one stop class seen in two runs of a week shows two stops and two runs, and once in each of two runs of another week shows in both;
  runs bucket by their `RunRow.Week` and landings by their own commit's week, so a run and its landing on either side of a week boundary count in different weeks, and a partial run counts in its week's `Partial` alone;
  a review segment two runs hold with 2 and 4 rounds has mean 3 and one run lacking it;
  a week with one run holding the arbiter source and one lacking it sums the first's arbiter outcomes alone and counts the second in `Lacking["arbiter"]`;
  a week whose two runs hold the `loom` LLM-step source with 2 and 3 steps, one of them landed by hand, sums `loom` to 5 and `landing` to 1.
  Its last row marshals a computed report and asserts the envelope's top-level keys (`selection`, `runs`, `partial`, `landings`, `weeks`, `absent`, `findings`) and that an absent wake source encodes as `null`.
- `TestRenderMarkdown` renders the report of a fake source with two weeks and asserts the four section headings in order, that each section's cells equal the report's own fields for both weeks, and that a `nil` share renders `no data`.

`cmd/lyx/testdata/test-budget.yaml` raises `internal/reportengine` to 8.

**ImpactSummary:** Extends the unreleased report engine; nothing outside `internal/reportengine` and the budget file changes.

**Commit:** `14: report engine: weekly aggregates and markdown`
