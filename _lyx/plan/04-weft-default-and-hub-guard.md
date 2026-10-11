# Card 4 — weft-default-and-hub-guard

**Create:**
- `plan:tools/tokencount#errForeignHub` -> `var errForeignHub error`
- `plan:tools/tokencount#ownHub` -> `func ownHub(hub, wd string) string`

**Rename:**
- `tools/tokencount#TestRunRefusesTheRetiredHistoryFlag` -> `plan:tools/tokencount#TestRunRefusals`

**Edit:**
- `tools/tokencount#run`

**Uses:**
- `plan:tools/tokencount#ReadRunSpan`
- `tools/tokencount#finishedRuns`

**Intent:**
Read every counted run's span from the weft, whether or not slugs are named, and bound the wrong-hub edge that opens (Decisions 8 and 10).

In `run`, in `main.go`:
- The weft default, `fabricengine.RecordsWorktree` of the `lyxcwd.Resolve`d current directory, now applies whenever -weft is empty, not only with no slugs or with `-calibrate`.
  Outside a resolvable worktree `lyxcwd.Resolve` fails, so a run with -weft empty now fails there with that error, named slugs included, where named slugs ran before.
- Right after `*hub` gets its default, `run` sets `*hub = ownHub(*hub, wd)`, so every later use, `CountRun`'s project directory and `ReadRunSpan`'s worktree check included, sees the absolute, cleaned hub.
  `ownHub` joins a relative hub onto `wd` (never `filepath.Abs`, which reads the process cwd) and returns it cleaned.
- When -weft is empty and that hub differs from `filepath.Clean(filepath.Dir(wd))`, the directory -hub defaults to, `run` returns an error wrapping `errForeignHub` before it derives the default.
  The message says the default weft belongs to the current hub, names both directories, and tells the operator to pass -weft with the records repository of the hub -hub names.
  `errForeignHub` is `errors.New` of the refusal's fixed part, declared in `main.go`.
  This is the edge the default opens: with -hub naming another hub, the default weft would read the current hub's archive tags for that hub's slugs.
  The bound: a foreign -hub with -weft empty is refused, and a given -weft is trusted as the operator's choice, as it is today; the same hub spelled with a trailing slash or as a relative path compares equal and passes.
- `finishedRuns(*weft)` now runs on every invocation, and its map feeds `recentRuns` when no slugs are named.
- After `CountRun` returns each run, `run` sets `r.Span` from `ReadRunSpan(r, *hub, finished[slug], records)`, `records` being one `gitrepo.New(*weft)` that the calibration also reuses in place of its own `gitrepo.New(*weft)`, and returns its error.
  `ReadRunSpan` reads no tag for a run whose worktree `<hub>/<slug>` still exists, so a named in-flight slug takes the no-status fallback even when an older run left a tag for it.
- The report title becomes `# Token usage and time by role`; the output path the launchers pass, `.scratch/token-usage-by-role.md`, stays (Decision 10).

Test: `TestRunRefusesTheRetiredHistoryFlag` becomes `TestRunRefusals`, one table of `run` refusals, which `PATTERN-test-economy` asks for refusals of one kind.
It keeps the retired-flag row as it is: the undefined-flag error naming -history and no report written.
It reads its own directory through `lyxcwd.Getwd`, the package directory `go test` runs in, and adds rows with slug `slug` and no -weft:
- -hub a fresh `t.TempDir()`: `errors.Is(err, errForeignHub)`, the message names -weft, and nothing is written to `stdout`.
- -hub the package directory's parent with a trailing separator, and -hub `..`: `errors.Is(err, errForeignHub)` is false.
  Both go on to `lyxcwd.Resolve` of the package directory, which is not the anchored worktree root, so `run` returns `lyxcwd.ErrCwdOutsideAnchor` before any git runs, and each row asserts that error, which also pins that -weft empty now resolves the weft for named slugs.
`main_test.go`'s header comment, which says the file checks that the retired history flag is refused, now says it checks `run`'s refusals.
The package's test count is unchanged, so `cmd/lyx/testdata/test-budget.yaml` keeps card 3's row.

**ImpactSummary:** Changes `run` alone, the tool's entry point with no other caller: every invocation now needs a resolvable worktree or a -weft, reads the archive tags once and reads each run's span, and the one test of `run` becomes its refusal table.
