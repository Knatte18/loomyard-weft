# Card 11 — timing-markdown

**Create:**
- `plan:internal/reportengine#HeadingTimingRuns` -> `const HeadingTimingRuns = "## Runs"`
- `plan:internal/reportengine#HeadingTimingPhases` -> `const HeadingTimingPhases = "## Phase medians"`
- `plan:internal/reportengine#HeadingTimingBuckets` -> `const HeadingTimingBuckets = "## Time per bucket"`
- `plan:internal/reportengine#HeadingTimingSettlers` -> `const HeadingTimingSettlers = "## Stops by settler"`
- `plan:internal/reportengine#RenderTimingMarkdown` -> `func RenderTimingMarkdown(r TimingReport) string`
- `plan:internal/reportengine#formatSeconds` -> `func formatSeconds(seconds *int64) string`
- `plan:internal/reportengine#TestRenderTimingMarkdown` -> `func TestRenderTimingMarkdown(t *testing.T)`

**Edit:**
- `internal/reportengine#allowedImports`
- `internal/reportengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Render a `TimingReport` as markdown, every duration through `durfmt.FormatSeconds` (card 1), the status pane's format for closed spans: with seconds below a day (`45s`, `12m05s`, `1h05m12s`) and as `2d03h` from a day on.

`RenderTimingMarkdown` writes `# Timing report`, the selection (records branch and tip, since, bucket kind, snapshot time, run, in-flight and finding counts) and the sleep note, then four sections under their heading constants, each a table through the existing `table` helper, which prints `no data` for a table with no rows:
the run table (slug, state, start, span, active, each wait kind, sleep, each phase, `other` included, and the attempt counts with their time);
the phase-median table (phase, median, runs);
the bucket table (bucket, runs, each phase, each wait kind, active, sleep);
and the stops-by-settler table per bucket (bucket, stop kind, settler, count).
`formatSeconds` spells a duration through `durfmt.FormatSeconds`, and a nil figure as `no data`, as `formatInt` does for counts.
`allowedImports` gains `internal/durfmt`, the one import this task adds to `reportengine`.

`TestRenderTimingMarkdown` in `internal/reportengine/timing_test.go` renders a `TimingReport` built in the test and asserts: the four headings in order; a run row's span of 3725 seconds rendered `1h02m05s` and one of 183600 seconds rendered `2d03h`; a nil `land` phase rendered `no data`; the run and bucket tables' sleep columns; a stops-by-settler row; and an empty report rendering `no data` under each heading.
`internal/reportengine/doc.go`'s Outputs section names `RenderTimingMarkdown` and its tables.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/reportengine` to 10.

**ImpactSummary:** the import allowlist gains one stdlib-only leaf; RenderMarkdown and convergence's output are untouched.

**Commit:** 11: Render the timing report as markdown
