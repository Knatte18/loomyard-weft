# Card 4 — report-splice

**Create:**
- `cmd/testtiming/splice.go`
- `cmd/testtiming/splice_test.go`

**Edit:**
- `cmd/testtiming/redundancy.go`
- `cmd/testtiming/main.go`
- `pattern/PATTERN-test-economy.md`
- `docs/benchmarks/running-tests.md`

**Intent:**
D3: a narrowed `-redundancy` run rewrites only its own sections of an existing report, so this task and `test-suite-prune-rest` can update `docs/benchmarks/test-redundancy.md` in parallel and merge cleanly.

`-pkg` takes a comma-separated list of package patterns (`-pkg ./internal/a,./internal/b`); `listPackages` and `runPackageTimings` pass each pattern as its own argument to `go`.
The acceptance command runs the six packages in one invocation, which a single-pattern flag cannot express.

`splice.go` holds a pure function over the existing report text: it takes the run's tag set, the measured packages and their rendered sections (or a marker that a package lists no tests, or that it failed) and returns the new report text or an error.
`runRedundancy` decides the mode:

- A missing `-out` file, or a run whose target packages are every package of the module, writes the whole report as today.
- Otherwise the existing file is spliced.
  The tags are read from the header line `renderReport` writes; when they differ from the run's tags the run exits non-zero and writes nothing.
  The header and every section of an unmeasured package stay byte-identical.
  A measured package's section is replaced; one with no section yet is inserted in import-path order; one that lists no tests has its section removed.
  A package that fails to list or build keeps its old section (or gets its error section if it had none), the other measured sections are still written, and the run exits non-zero.

Today `runRedundancy` drops a package with no tests before measuring it; the splice needs that package named as measured-without-tests, so the loop records it instead of skipping it.

Bound: the splice touches only the sections of packages the run measured, and refuses rather than mixing tag sets in one report.

Tests: `splice_test.go` drives the pure function with a small report fixture, one row each for replacing a section, inserting a new section in order, removing a section of a package without tests, keeping the old section of a failed package while writing the others (and the error result), refusing a different tag set, and leaving the header and unmeasured sections byte-identical (exact string equality on the whole output).
The comma-separated `-pkg` split gets a row or assertion in an existing test of the flag handling, or in `splice_test.go` if no such test exists.

Docs, same commit: the `main.go` package doc and `docs/benchmarks/running-tests.md` describe the comma-separated `-pkg` and the splice; `pattern/PATTERN-test-economy.md` says a narrowed run rewrites only its own packages' sections.

**ImpactSummary:** Changes how the redundancy mode writes an existing report and parses `-pkg`; the whole-module run and the timing mode behave as before.
