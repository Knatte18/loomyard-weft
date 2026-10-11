# Card 5 — weft-wiring

**Create:**
- `plan:tools/tokencount#errHubNeedsWeft` -> `var errHubNeedsWeft error`
- `plan:tools/tokencount#TestRunRefusesAForeignHubWithoutWeft` -> `func TestRunRefusesAForeignHubWithoutWeft(t *testing.T)`

**Edit:**
- `tools/tokencount#run`
- `plan:tools/tokencount#TestArchivedStatusFromWeft`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:tools/tokencount#newWeftArchive`
- `plan:tools/tokencount#readArchivedRun`
- `plan:tools/tokencount#RunTally.useArchive`
- `internal/lyxcwd#Resolve`
- `internal/fabricengine#RecordsWorktree`

**Intent:** Wire the archived status into every report run and bound the weft default (Decision 8 and Decision 10 of the decision record).

In `tools/tokencount/main.go`:
- `errHubNeedsWeft` is an `errors.New` sentinel worded as the refusal: a `-hub` other than the current worktree's parent needs -weft.
- `run`'s weft default now applies whenever -weft is empty, not only with no slugs or with `-calibrate`.
  Before deriving it, `run` compares `filepath.Abs` of `-hub` with `filepath.Clean` of the current directory's parent; when they differ it returns `errHubNeedsWeft` wrapped with both directories and the way forward, to pass -weft naming that hub's records repository, and reads nothing.
  This guard lets through only a `-hub` that names the current worktree's own hub, the one hub whose records the default names, so a slug can no longer be read against another hub's records; an explicit -weft skips the guard, and the caller then owns the pairing.
  The input reaching the refusal after every upstream check is a parsed flag set with `-hub` naming another directory and -weft empty; `TestRunRefusesAForeignHubWithoutWeft` reaches it.
  Outside a resolvable worktree `lyxcwd.Resolve` fails as it does today, now for named slugs too.
- After counting each slug, `run` reads its archive with `readArchivedRun(newWeftArchive(*weft), slug)` and hands the result to `useArchive`; a read error aborts the report.
  A slug whose worktree `<hub>/<slug>` still exists as a directory is in flight, since teardown removes the worktree, so `run` reads no archive tag for it and it keeps the transcript-window span, even when an older run of the same slug left a tag.
- The report title becomes `# Token usage and time by role`; the output path and the launchers' `-out` stay.

`TestRunRefusesAForeignHubWithoutWeft`, in `main_test.go`, is table-driven over `run`'s arguments, each row naming a slug and no -weft:
- a `t.TempDir()` hub fails with an error that `errors.Is` `errHubNeedsWeft` and that names -weft, and writes nothing;
- the package directory's parent spelled with a trailing separator, and spelled as the relative `..`, fail with an error that is not `errHubNeedsWeft`, since the package directory is no anchored worktree and the weft default cannot be derived there.

`TestArchivedStatusFromWeft` gains an end-to-end pass through `run` with -weft naming its fixture repository, `-hub` and `-projects` naming temporary directories, and slugs `alpha` and `beta`, each with synthetic transcripts.
`alpha` has its archive tags, and its section carries the archived span line and a Producers table.
`beta` has an archive tag in the fixture and an existing `<hub>/beta` directory, and its section says no shed status was read and carries no Producers table.
The report opens with the new title.

`cmd/lyx/testdata/test-budget.yaml`: raise the `tools/tokencount` row by the one test this card adds.

**ImpactSummary:** `run` now reads every counted run's archived status from the weft, refuses a foreign `-hub` without -weft, fails outside a resolvable worktree when -weft is empty, and retitles the report; the calibration and profile sections read the same weft as before.

**Commit:** 5: tokencount: read each run's archived status from the weft
