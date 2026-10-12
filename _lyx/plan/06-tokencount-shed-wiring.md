# Card 6 — tokencount-shed-wiring

**Create:**
- `plan:tools/tokencount#checkHub` -> `func checkHub(hub, wd string) error`
- `plan:tools/tokencount#TestCheckHub` -> `func TestCheckHub(t *testing.T)`

**Edit:**
- `tools/tokencount#run`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** `run` reads each run's archived shed record and wires the bounds Decision 8 sets.
The weft default, `fabricengine.RecordsWorktree` of the resolved location, now applies whenever the weft flag is empty and no longer only with no slugs or with `-calibrate`, so a run with that flag empty outside a resolvable worktree now fails at `lyxcwd.Resolve`, named slugs included.
That default reads the current hub's weft, so `checkHub` runs first, once `-hub` has its default, whenever the weft flag is empty: it joins a relative `hub` onto `wd`, cleans both sides and compares the result with `filepath.Dir(wd)`, and it refuses a difference with an error that names the weft flag as the way forward, built from the existing `repositoryFlag` constant.
The guard bounds the reach of the new default: a `-hub` spelled with a trailing slash, as a relative path or with `.` elements still names the current hub and passes, and a different hub passes only with the weft flag named, which the guard never inspects.
After each `CountRun`, `run` sets `RunTally.Shed` from `newShedSource(*hub, *weft).Read(slug)`, so a slug whose worktree `<hub>/<slug>` still exists, or that has no archive tag, keeps `Shed` nil and prints the no-status section; a read or decode error from `Read` aborts the report, naming the slug, the tag and the file.
`TestCheckHub` is a table over strings built from `t.TempDir()` with no disk access, and the refusal branch is reached by its first row, `hub` a sibling directory of `wd`'s parent: the error names the weft flag and the hub; the rows that pass are `hub` equal to `filepath.Dir(wd)`, the same with a trailing separator, the same with a `.` element and a relative `..` against `wd`, and a further refused row names a directory that shares only a string prefix with the parent.
The weft default itself has no test: it needs a resolvable worktree, and the existing flag test stops at flag parsing.
`cmd/lyx/testdata/test-budget.yaml` raises the `tools/tokencount` row by one for `TestCheckHub`.

**ImpactSummary:** `run` is called by `main` and the package's tests only; its flags and output are unchanged except that an empty weft flag now always resolves the default, and a foreign `-hub` with that flag empty is refused where it used to read the current hub's records.
