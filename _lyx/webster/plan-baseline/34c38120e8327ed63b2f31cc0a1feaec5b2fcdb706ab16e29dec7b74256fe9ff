# Card 5 — measure-before

**Create:**
- `docs/benchmarks/test-suite-prune-core.md`

**Edit:**
- `docs/benchmarks/running-tests.md`

**Intent:**
D8's before half, recorded in the measurement report the "Measurements go in a committed benchmark report" decision names.
This card runs before any test of the six packages changes; cards 1–4 touch only `cmd/testtiming` and docs.

Measure, on this machine, for each of `internal/fabricengine`, `internal/websterengine`, `internal/reedengine`, `internal/shuttleengine`, `internal/loomcli` and `internal/shedadapters`:

- top-level test count, wall time and serial time from `go run ./cmd/testtiming -tags integration,tmux`, three runs, taking the median of each figure per package;
- statement coverage of the package's own code from `go test -tags integration,tmux -cover -count=1 <pkg>`;
- the load average (`cat /proc/loadavg`) at the start of each of the three timing runs and the coverage run.

Write `docs/benchmarks/test-suite-prune-core.md` as a measurement report: a title, a sentence on what was measured with the exact commands, the commit measured, a before table with one row per package (tests, wall, serial, coverage), and the load averages.
Leave an after section the final card fills, described in one sentence so the file reads as complete now.
Also keep, uncommitted under `.scratch/`, the per-package coverage profiles (`-coverprofile`), so the final card can name each lost block; mention in the report that the before profiles are reproducible from the measured commit.

Add one line linking the new report from `docs/benchmarks/running-tests.md`, next to its link to `test-redundancy.md`.
Semantic line breaks throughout; every link resolves (`PATTERN-markdown-link-integrity`).

**ImpactSummary:** Adds one docs page and a link to it; no code changes.
