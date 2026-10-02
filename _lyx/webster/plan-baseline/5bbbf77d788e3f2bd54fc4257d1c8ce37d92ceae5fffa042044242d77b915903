# Card 11 — describe-every-generation

**Edit:**
- `internal/landingshed/describe.go`
- `internal/landingshed/describe_test.go`
- `internal/loomshed/generation.go`
- `internal/loomshed/generation_test.go`
- `internal/loomcli/wiring.go`

**Prosa:**
- `contracts/stencils/landing/landing-template-describe.md`

**Intent:**
Tell Describe every generation's run record, oldest first (D11), so the PR description keeps manual-check items from generations a rework round retired.

- `loomshed` gains a lister returning each round's archived webster directory (`round-<N>/prior-generation/webster`), ascending by `N`, for every round that has one; the layout constants stay the ones `prrework.go` declares.
- `DescribeInputs` gains `PriorRunRecordPaths []string`; `RunRecordPath` stays the live record and stays required.
  The stencil's `run_record_path` marker becomes `run_record_paths`, rendered as one bullet per path, every prior generation oldest first and the live record last, and the "What to read" item says so.
- `wire()`'s `DescribeSpec` closure fills `PriorRunRecordPaths` with `summaryparser.Path` of each listed directory whose summary file exists, so `summaryparser` stays the sole declarer of the filename.

Tests: the lister returns `round-1` before `round-2` before `round-10` and skips a round with no archive; `describe_test.go` asserts the rendered prompt lists two prior records then the live one in that order, and only the live one when none is told.

**ImpactSummary:** Describe's inputs and stencil marker change; the only caller is `wire()`'s `DescribeSpec` closure.
