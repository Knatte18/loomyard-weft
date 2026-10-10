# Card 4 — weight-report-before

**Create:**
- `docs/benchmarks/test-suite-weight.md`

**Prosa:**
- `docs/benchmarks/running-tests.md`

**Intent:** The before-state of D3, measured on the suite as it stands after cards 1 to 3, which changed no test's cost.
The implementer runs `go run ./cmd/testtiming -resources` for the untagged tier, `-resources -tags integration` and `-resources -tags tmux`, each once, and writes `docs/benchmarks/test-suite-weight.md`: a header naming the commit, the machine, the date and that it is a measurement report that may go stale; then a `## Before` section with, per tier, the load averages, the per-package resource table and the git census table, pasted from the harness output.
A `## After` section is left as a heading with one line saying the last card fills it.
`running-tests.md` gains a paragraph under the timing-harness section naming the three commands that regenerate the report, the CPU and memory sources and the "quiet machine" precondition, and links the report.
The harness takes its own gate slot, so the runs need no `lyx gate test` wrapper.

