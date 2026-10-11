# Card 1 — gofmt-guard

**Create:**
- `plan:cmd/lyx#gofmtScanFloor` -> `const gofmtScanFloor = 500`
- `plan:cmd/lyx#gofmtFinding` -> `func gofmtFinding(file string, src []byte) (string, bool)`
- `plan:cmd/lyx#TestGofmtFinding` -> `func TestGofmtFinding(t *testing.T)`
- `plan:cmd/lyx#TestGofmt_ModuleFilesAreFormatted` -> `func TestGofmt_ModuleFilesAreFormatted(t *testing.T)`

**Edit:**
- `internal/boardcli#Command`
- `internal/fabricengine/pull.go`
- `internal/pattern/doc.go`
- `internal/reedengine/lifecycle_test.go`
- `internal/shedverbs#TestUntilStop_LoserWaitsOnWinner`
- `internal/shell#posixShell.Quote`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/testkit/scankit/scankit.go`
- `cmd/lyx/testbudget_test.go`

**Intent:** Add the Tier 1 gofmt guard of decision D7 to `cmd/lyx`, in a new `cmd/lyx/gofmt_test.go` beside the other repository guards, and bring the six files it flags today to gofmt-clean in the same commit.
`gofmtFinding` takes one file's module-relative path and source, parses it whatever its build tags, and compares the source with `go/format.Source` of itself, the library gofmt formats with.
A source that does not parse answers a finding naming the file and the parse error.
A source whose formatted text differs answers a finding naming the file, the first line where the two texts differ, the current line and the formatted line, so the fix is an Edit;
the message never tells the reader to run `gofmt -w` or any rewriting tool, since agents change files only with Edit or Write.
A formatted source answers no finding.
`TestGofmtFinding` pins those three outcomes over in-memory sources, as `TestBudgetViolations` does for the budget guard: an unformatted source names file, line, current and formatted text; an unparseable source names the file; a formatted source passes; a file whose only difference is import order still fails with that line.
`TestGofmt_ModuleFilesAreFormatted` carries `//lyx:guard` directly above its `func` line, calls `t.Parallel`, walks every `.go` file through `scankit.Walk` with `Filter: scankit.All`, which already skips every directory whose name contains `testdata` and so the nested module under `internal/planglyph/testdata`, requires `gofmtScanFloor` scanned files through `scankit.RequireFloor` so a broken walk cannot pass vacuously, and reports every finding through `t.Errorf`.
It spawns nothing and needs no `gofmt` binary on `PATH`.
The six flagged files are fixed by hand, each the way gofmt would format it: the `Use` and `Short` alignment in `boardcli.Command`'s cobra literal, the trailing blank line at the end of `pull.go`, the import order in `lifecycle_test.go`, the one-line closure in `TestUntilStop_LoserWaitsOnWinner`, and the indented example line in `internal/pattern/doc.go`, which gofmt rewrites into a doc-comment list item.
`posixShell.Quote`'s doc comment names the `'\''` idiom, which gofmt's doc formatter turns into curly quotes; reword it so the idiom sits in backticks and gofmt leaves it alone.
No PATTERN entry is added, since gofmt-clean code is the Go toolchain's convention, not a loomyard invariant.
The `cmd/lyx` row of `cmd/lyx/testdata/test-budget.yaml` rises by two, one per new top-level test; the record counted one, but the in-memory cases cannot share the guard's own function, which does the real scan.

**ImpactSummary:** One new guard in cmd/lyx plus whitespace-only or comment-only fixes in six files; no behavior changes outside the test suite.

**Verify:** [ -z "$(gofmt -l .)" ]
