# Card 7 — tokencount-main-wiring

**Create:**
- `plan:tools/tokencount#TestRunRefusesAnotherHubWithoutWeft` -> `func TestRunRefusesAnotherHubWithoutWeft(t *testing.T)`

**Edit:**
- `tools/tokencount/main.go`
- `tools/tokencount/tokencount.sh`
- `tools/tokencount/tokencount.cmd`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:tools/tokencount#readShedRecords`
- `plan:tools/tokencount#timeRun`
- `internal/fabricengine#RecordsWorktree`
- `internal/lyxcwd#Resolve`
- `internal/gitrepo#New`
- `tools/tokencount#repositoryFlag`

**Intent:** Wire the records read and the timing into `run`, apply the two bounds of Decision 8, retitle the report per Decision 10, and describe the time columns in the package doc and the launchers.
In `run`, the repository flag's default, the current worktree's weft sibling, now applies whenever the flag named by `repositoryFlag` is empty, named slugs or not:
before resolving it, when `-hub` is set and its absolute, cleaned path differs from the absolute, cleaned parent of the current directory, `run` refuses with an error naming the repository flag as the flag to pass for another hub's records, so a trailing slash or a relative spelling of the same hub is not refused;
then `lyxcwd.Resolve` and `fabricengine.RecordsWorktree` give the default as today, and outside a resolvable worktree that error is returned as it is.
After `CountRun`, for each slug `run` calls `readShedRecords(gitrepo.New(weft), hub, slug)` and `timeRun(r, records)`, stores the result in the run's `Timing`, and returns the first error.
The report title becomes `# Token usage and time by role`;
the output path the launchers write stays `.scratch/token-usage-by-role.md`.
The package doc in `main.go` gains, after the usage paragraphs, the description of the time columns, the idle row and the Producers table: working time as turn time between user and assistant lines, the three turn-ending line kinds, the union per role with `+sub` as its own row, the span from the seed's `started_at` to the last history entry's end with the no-start fallback and its two bounds, the transcript-window fallback for a run with no archived status, idle as the span minus the union of every transcript's working time with the orch left out, the Producers table's per-entry interval rule, how "All runs" sums, and that the archived `status.json` and `seed.json` are read at the slug's newest `archive/<slug>/<sha>` tag from the weft repository.
The same doc states the two bounds: a `-hub` other than the current worktree's parent needs the repository flag, and a named slug whose worktree still exists reads no archive tag.
The repository flag's help text and the doc's sentence about its default change to say the default applies whenever the flag is empty.
The description comments at the top of `tokencount.sh` and `tokencount.cmd` say the report holds token use and time by role, the idle row and the Producers table, and that the repository flag is needed with a `-hub` other than the current hub.
The doc keeps one sentence per line.
`TestRunRefusesAnotherHubWithoutWeft` in `main_test.go` calls `run` with a slug and `-hub` set to a `t.TempDir()` and asserts the error names the repository flag, as a dash followed by `repositoryFlag`, and that nothing was written;
it then calls `run` with `-hub` as the current directory's parent spelled with a trailing slash, and again spelled as the relative `..`, and asserts each error does not name the repository flag, since the test's package directory is no worktree root and `lyxcwd.Resolve` refuses it after the bound has passed.
Raise the `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` from 15 to 16 for the new test.

**ImpactSummary:** run gains a repository-default refusal and reads the weft for every invocation with the repository flag empty, so a named-slug run outside a worktree now fails where it ran before; the report title changes and the time figures appear.
