# Card 8 — wire the archived records into the report

**Create:**
- `plan:tools/tokencount#sameHub` -> `func sameHub(hub, wd string) (bool, error)`
- `plan:tools/tokencount#TestRunRefusesAForeignHubWithoutWeft` -> `func TestRunRefusesAForeignHubWithoutWeft(t *testing.T)`

**Edit:**
- `tools/tokencount#run`
- `plan:tools/tokencount#TestArchiveTagsGitBacked`

**Delete:**
- `tools/tokencount#finishedRuns`
- `tools/tokencount#archivedSlugs`
- `tools/tokencount#TestArchivedSlugs`

**Uses:**
- `plan:tools/tokencount#listArchiveTags`
- `plan:tools/tokencount#LoadArchive`
- `plan:tools/tokencount#RunTally.Measure`

**Intent:** `run` now reads the archive tags once and gives each run its archived records, so the report's time sections have a status to measure against.
`sameHub` goes in `main.go` and reports whether `hub` and the parent of `wd` are the same directory, comparing `filepath.Abs` of each, which cleans them, so a trailing slash or a relative spelling of the same directory is the same.

In `run`, the weft default now applies whenever -weft is empty, not only with no slugs or with `-calibrate`.
Before `lyxcwd.Resolve`, it refuses when `sameHub(*hub, wd)` is false: the error says the default weft is the current hub's and would read the wrong hub's records for `-hub`, and names -weft as the way forward.
The default `-hub`, the parent of `wd`, always passes.
The refusal is reached by `-hub` naming a directory other than the current worktree's parent with -weft empty, which no earlier check in `run` rules out, because flag parsing, `lyxcwd.Getwd` and the projects default come first and accept it; `TestRunRefusesAForeignHubWithoutWeft` runs `run` with a `t.TempDir()` hub, an explicit `-projects` and a slug, and asserts an error naming -weft and no output.
The same test tables `sameHub` over the hub spelled as the parent itself, with a trailing separator and as a relative path built from `lyxcwd.Getwd()`, each true, and over another directory, false.
Outside a resolvable worktree `lyxcwd.Resolve`'s error is returned, so a run with -weft empty now fails there with named slugs too, where it ran before; it is the existing propagated error, with no test row.

`run` then calls `listArchiveTags(*weft)` once, which Decision 8's status read needs for every run, and builds the finished-run set `recentRuns` takes from its slugs, since a second listing of the same refs would be a duplicate to keep in step.
`finishedRuns`, its helper `archivedSlugs` and `TestArchivedSlugs` are therefore deleted; `run` is `finishedRuns`'s only caller, on this card, and `TestArchivedSlugs`'s cases live on in the junk-line rows of `TestLoadArchive`.
The `gitexec` import in `count.go` served `finishedRuns` alone and goes with it.
Each counted run then gets `LoadArchive(*hub, slug, tags[slug], fabric)` and `r.Measure(archive)`, where `fabric` is the one `gitrepo.New(*weft)` that the calibration block also uses, so the calibration no longer builds its own.
The report title becomes `# Token usage and time by role`; the `-out` path the launchers pass stays.
A named slug whose worktree `<hub>/<slug>` still exists takes `LoadArchive`'s nil, so an in-flight run never reads an older run's tag.
The `-hub` flag's help text says that with -weft empty it must be the current worktree's parent.

`archive_integration_test.go`'s `TestArchiveTagsGitBacked` gains a final section that drives `run` over the hermetic weft with explicit `-hub`, `-projects`, -weft and `-out`, a transcript for the finished slug under `projectDir(projects, hub/slug)`, and a second slug that has an older tag and a directory `<hub>/<slug>`.
Both slugs have a project directory with a transcript, since a named run with none fails in `CountRun`.
It asserts the report's title, the finished slug's section carrying a Producers table and a span from the newer commit's seed, and the in-flight slug's section carrying the no-status sentence.
The finished-run set built from the tags is pinned by a no-slug run in the same fixture, where a third slug with a project directory and no tag is not listed and the tagged slugs are; `recentRuns` filters on that set and on a project directory only, and the in-flight rule applies to named slugs alone, so the in-flight slug is listed there too.

The test-budget row stays at 17: this card adds one top-level test and removes one.

**ImpactSummary:** `run` is the entry point, so its flow changes for every invocation, and a named-slug run outside a worktree now fails where it did not; `finishedRuns` has `run` as its one caller and `archivedSlugs` and its test have no other.
