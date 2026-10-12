# Card 6 — main wiring

**Create:**
- `plan:tools/tokencount#refuseForeignHub` -> `func refuseForeignHub(hub, wd, weft string) error`
- `plan:tools/tokencount#TestRefuseForeignHub` -> `func TestRefuseForeignHub(t *testing.T)`

**Edit:**
- `tools/tokencount#run`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** `run` reads each slug's archived shed status and hands it to the report, in `main.go`.
The weft default, `fabricengine.RecordsWorktree` of the resolved location, now applies whenever the weft flag is empty, not only with no slugs or with the calibration, so a run outside a resolvable worktree with the weft flag empty fails at `lyxcwd.Resolve`, named slugs included.
That default reads the current hub's weft, so a slug could be read against the wrong hub's records; `refuseForeignHub` bounds it: when `weft` is empty and `hub` differs from the current worktree's parent `filepath.Dir(wd)`, both made absolute and cleaned, it returns an error naming the weft flag, spelled with its dash, and `run` calls it right after the weft default's precondition is known, before any transcript is read.
A weft flag that is named skips the check, so the operator can read another hub's records on purpose.
After the runs are counted, `run` calls `ReadShedRecord(newGitShedSource(*weft), *hub, slug)` per slug and sets `report.Shed[slug]`; an error aborts the report, and a slug whose worktree still exists reads no tag, as card 3 sets.
The report title becomes `# Token usage and time by role`; the output path `.scratch/token-usage-by-role.md` is unchanged, and the old title text appears in no test, document or launcher.
The refusal branch is reached by a hub flag of another hub with the weft flag empty, the cwd being a worktree of another hub, which nothing upstream rejects: flag parsing accepts it and `lyxcwd.Getwd` has already resolved `wd`.
`refuseForeignHub` takes the working directory as `wd` and resolves a relative `hub` against it with `filepath.Join` and `filepath.Clean`, so it never calls `os.Getwd`, per `PATTERN-cwd-resolution`.
`TestRefuseForeignHub` is one new top-level test in `main_test.go`, run with `t.Parallel()`, calling `refuseForeignHub` over `t.TempDir()` paths, and its rows are:
- a hub other than the worktree's parent with `weft` empty returns an error whose text contains the flag spelling with its dash;
- the same hub spelled with a trailing slash, and the same hub as a path relative to the `wd` argument (built with `filepath.Rel(wd, hub)`), return nil;
- a foreign hub with `weft` named returns nil;
- `run` called with a hub flag naming a `t.TempDir()` directory, the weft flag empty and a slug returns the refusal naming the weft flag, before any transcript is read, which pins that `run` calls the guard; the test's real working directory is a worktree whose parent differs from that directory.
The new top-level test takes the `tools/tokencount` row of `cmd/lyx/testdata/test-budget.yaml` from 17 to 18.

**ImpactSummary:** `run` has one caller, `main`, and `TestRunRefusesTheRetiredHistoryFlag` fails at flag parsing before the new code runs; the only new refusal is a hub flag naming another hub with the weft flag empty, which no existing test or launcher passes.
