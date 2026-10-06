# Card 1 — measure-before

**Create:**
- `docs/benchmarks/test-suite-prune-rest.md`

**Edit:**
- `docs/benchmarks/running-tests.md`

**Intent:**
The before half of Decisions 11 and 12, recorded in the measurement report the parent's "Measurements go in a committed benchmark report" decision names.
This card runs on the untouched tree: no test file changes before it commits.

For every in-scope package (the decision record's Scope: every package with a `## ` section in `docs/benchmarks/test-redundancy.md` except `internal/fabricengine`, `internal/websterengine`, `internal/reedengine`, `internal/shuttleengine`, `internal/loomcli`, `internal/shedadapters` and everything under `internal/testkit/`), measure on this machine:

- top-level test count, wall time and serial time from `go run ./cmd/testtiming -tags integration,tmux`, three runs, taking the median of each figure per package (Decision 11);
- non-test statement coverage exactly as Decision 12 defines it: the package's tests run from the repository root under `-tags integration,tmux -covermode=set -coverpkg=./... -coverprofile=<file>`, and covered statements over total statements summed from each block's statement field, over the non-test files directly in the package's own directory;
- the load average (`cat /proc/loadavg`) at the start of each timing run and of the coverage run.

Record the commit measured, which is this branch's base commit.
A small helper that turns a profile into the per-package figure lives under `.scratch/`, never in the repo.
Also run each in-scope test the report lists as `failed` alone, as Decision 8 prescribes, and keep the verdicts for the package cards.
Keep the before coverage profiles under `.scratch/` so card 18 can name a lost block, and say in the report that they are reproducible from the measured commit.

Write `docs/benchmarks/test-suite-prune-rest.md` as a measurement report: a title, one sentence on what was measured with the exact commands, the commit, a before table with one row per in-scope package (tests, wall, serial, coverage), and the load averages.
An after section that card 18 fills is described in one sentence, so the file reads as complete now.

In `docs/benchmarks/running-tests.md`, add one line linking the new report, as its own line directly after the line linking `test-redundancy.md`, so `test-suite-prune-core`'s own link line merges beside it.
Semantic line breaks throughout, and every link resolves (`PATTERN-markdown-link-integrity`).

**ImpactSummary:** Adds one docs page and one link line; no code or test changes.

**Commit:** 1: Record the before measurement of the rest of the test suite
