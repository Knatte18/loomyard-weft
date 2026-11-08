# Coverage study of fabricengine tier 1 tests that others already cover

Adds a measurement report and its scripts under `docs/research/speed/fabric-test-redundancy-2026-11-08/`.
No test and no production code changes.

## What the study found

Each of the 590 top-level tier 1 tests of `internal/fabricengine` ran once on its own under a coverage binary scoped to that package.
370 of them cover only blocks that another tier 1 test also covers; 68 of those already carry `//testtiming:keep`.
329 of the 370 can go together without uncovering a block, and the 267 unmarked ones cost 78 s standalone.
Block coverage is coarse here, because most tests share a large fixture path, so the report treats these as reading candidates and not as cuts.

The report also lists tests that start git or build a hub while asserting logic that needs neither,
and for the top candidates of both lists says what the test asserts that block coverage does not show.

## Design decisions

- The committed `docs/benchmarks/test-redundancy.md` lists no candidates for this package, because `cmd/testtiming -redundancy` measures coverage over the whole module.
  This study scopes coverage to `internal/fabricengine` as the task asked, and the report names the difference so a later task confirms a cut with the module-wide tool first.
- Raw `go test -c` is refused, so `coverage-runner.sh` runs as the `-exec` program of one `lym gate test --tier 1` call and runs the compiled binary once per test.
  The gate prints that it ran no test, because the runner emits no `go test -json` events; the runner's own logs and profiles are what the analysis reads.
- Costs are standalone times, which include each process's fixture build, so they rank the tests and bound a cut's saving from above.
- Two scripts, `analyze-coverage.sh` and `pure-logic-candidates.sh`, and `git-census.sh`, read the runner's output; the `.tsv` files are their results.

## Verification

The scripts ran end to end on the committed tree, and the figures in `report.md` come from the `.tsv` and `summary.txt` beside it.
`lym gate test` was run only as the runner's carrier; the repository's own test suites are untouched.
