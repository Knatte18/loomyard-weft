# Card 3 — keep-directive

**Create:**
- `cmd/testtiming/keep.go`
- `cmd/testtiming/keep_test.go`

**Edit:**
- `cmd/testtiming/redundancy.go`
- `cmd/testtiming/redundancy_test.go`
- `cmd/testtiming/main.go`
- `pattern/PATTERN-test-economy.md`
- `docs/benchmarks/running-tests.md`

**Intent:**
D2: a candidate the prune keeps carries `//testtiming:keep <reason>` on the comment line directly above its `func Test…` line, and the report lists it under "Kept" with the reason.

`keep.go` scans a package directory's `_test.go` files line by line and returns the kept tests by name with their reasons, or an error.
The rules, each an error naming the file and line (and the test, where there is one):

- the reason is empty or only whitespace;
- the line after the directive does not start a top-level `func Test…` declaration, which covers a directive separated from its test by a blank or other line, a directive at the end of a file, and the upper of two stacked directives on one test.

`measurePackage`, or `runRedundancy` before any package is measured, calls the scan for every target package; a directive error aborts the run with a non-zero exit before anything is measured or written, so a bad directive never yields a partial report.

`verdict` gains the keep reason.
`classify` is unchanged: a kept test stays in the covering pool and in the removable computation.
`renderPackage` omits a kept test from the candidates table and lists every kept test, candidate or not, under a `Kept:` list (`` - `TestX`: <reason> ``) after the table; the "no coverage" list is unchanged.

Bound: the directive changes the report's output only, never which tests run or how coverage is computed, and the reason is printed beside the test, so every keep is visible in the committed report.

Tests: `keep_test.go` drives the scan over fixture files in `t.TempDir()` — a valid directive with and without a preceding doc comment, an empty reason, a whitespace-only reason, a directive separated from its test, a doubled directive, a directive followed by a non-test function, and a directive at end of file — asserting the exact map or the error's file, line and test.
`TestRenderPackage` gains a row with a kept candidate (absent from the table, present under "Kept") and a kept non-candidate (listed under "Kept").
`TestClassify` gains an assertion or row that a kept test still covers another test's blocks, if `classify` reads the keep at all; otherwise the render row suffices.

Docs, same commit: the `main.go` package doc and `docs/benchmarks/running-tests.md` describe the directive; `pattern/PATTERN-test-economy.md` says a kept candidate carries the directive with what it pins that its covering tests do not (an error message, an output shape, an ordering).

**ImpactSummary:** Adds the keep scan and a "Kept" list to the redundancy report; no other package imports `cmd/testtiming`.
