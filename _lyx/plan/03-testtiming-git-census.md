# Card 3 — testtiming-git-census

**Create:**
- `plan:cmd/testtiming#gitCensus` -> `type gitCensus struct`
- `plan:cmd/testtiming#traceEnv` -> `func traceEnv(dir string) []string`
- `plan:cmd/testtiming#readCensus` -> `func readCensus(dir string) (gitCensus, error)`
- `plan:cmd/testtiming#parseTraceEvents` -> `func parseTraceEvents(events []byte) (subcommand string, fixture bool, ok bool)`
- `plan:cmd/testtiming#renderCensus` -> `func renderCensus(rows []resourceRow) string`
- `cmd/testtiming/census_test.go`

**Edit:**
- `plan:cmd/testtiming#resourceRow`
- `plan:cmd/testtiming#measurePackageResources`
- `plan:cmd/testtiming#renderResources`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/gitkit#FixtureGitEnv`

**Intent:** The git census columns of D1 and the fixture split of D2, in a new file `cmd/testtiming/census.go`.
`traceEnv` returns the environment lines that make every git process under the child write one trace2 event file: `GIT_TRACE2_EVENT=<dir>` with a directory, `GIT_TRACE2_ENV_VARS` set to `gitkit.FixtureGitEnv`, and `GIT_CONFIG_COUNT`, `GIT_CONFIG_KEY_0=trace2.maxFiles`, `GIT_CONFIG_VALUE_0=0` so git never stops writing files after its default cap.
`measurePackageResources` creates the trace directory beside the package's temp directory, adds `traceEnv` to the child's environment and fills the row's census from `readCensus` after the child exits.
`readCensus` reads every file in the directory and `parseTraceEvents` takes the `cmd_name` event's `name` as the subcommand and a `def_param` event naming the marker as the fixture flag; a file with no `cmd_name` event counts under `unknown`.
`gitCensus` holds, per subcommand, the fixture count and the code count, plus totals.
`renderResources` adds `GIT_FIXTURE` and `GIT_CODE` totals per package row, and `renderCensus` prints a second table per run with one row per subcommand: `SUBCOMMAND`, `TOTAL`, `FIXTURE`, `CODE`, sorted by total, with a trailer stating that the split is exact for a serial package and approximate where a hub build overlaps other tests of the same package, so a before-to-after comparison reads the total.
`parseTraceEvents` and `renderCensus` are tested in `census_test.go` over fixture event text, untagged; the fixture spells the marker's name as a literal in its JSON and the file references no `gitkit.` export, since the tier-purity scan counts every gitkit export but `HermeticGitEnv` as a spawn.
The package comment gains the census columns and the marker's meaning.
`cmd/testtiming`'s budget row rises to cover the new test functions.

**ImpactSummary:** Three card-2 symbols grow a census field, an environment addition and two columns; nothing outside `cmd/testtiming` changes.
