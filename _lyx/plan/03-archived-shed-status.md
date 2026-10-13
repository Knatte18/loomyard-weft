# Card 3 — archived-shed-status

**Create:**
- `plan:tools/tokencount#archiveTag` -> `type archiveTag struct`
- `plan:tools/tokencount#ShedRecord` -> `type ShedRecord struct`
- `plan:tools/tokencount#shedEntry` -> `type shedEntry struct`
- `plan:tools/tokencount#ArchiveFiles` -> `type ArchiveFiles interface`
- `plan:tools/tokencount#readShedRecord` -> `func readShedRecord(files ArchiveFiles, hub, slug string, tags []archiveTag) (*ShedRecord, error)`
- `plan:tools/tokencount#TestRunRefusesAnotherHubByDefault` -> `func TestRunRefusesAnotherHubByDefault(t *testing.T)`
- `tools/tokencount/shedstatus_test.go`
- `tools/tokencount/shedstatus_integration_test.go`

**Edit:**
- `tools/tokencount#finishedRuns` -> `func finishedRuns(weft string) (map[string][]archiveTag, error)`
- `tools/tokencount#archivedSlugs` -> `func archivedSlugs(refs string) map[string][]archiveTag`
- `tools/tokencount#recentRuns` -> `func recentRuns(projects, hub, prime string, finished map[string][]archiveTag, last int) ([]string, error)`
- `tools/tokencount#RunTally`
- `tools/tokencount#run`
- `tools/tokencount#TestArchivedSlugs`
- `tools/tokencount#TestRecentRuns`
- `tools/tokencount/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shedrun#TreeSeedRel`
- `plan:internal/shedrun#TreeStatusRel`
- `plan:internal/shedrun#DecodeSeed`
- `tools/tokencount#fakeHistory`

**Intent:**
Read each counted run's archived `seed.json` and `status.json` from its archive tag in the weft repository (Decisions 8 and 9), and make the weft default apply whenever the -weft flag is empty, bounded by a refusal for a foreign `-hub`.
This card only reads and keeps the records; card 4 turns them into the span and the Producers table.

Archive tags:
- `archiveTag` is one `archive/<slug>/<sha>` tag: its `Name` (without `refs/tags/`), the `Commit` SHA it names, and that commit's committer time `Committed`.
- `finishedRuns` runs `gitexec.Run` with `for-each-ref --format=%(refname)%09%(objectname)%09%(committerdate:unix)%09%(*objectname)%09%(*committerdate:unix) refs/tags/archive/` in the weft (`PATTERN-gitexec-checked-call`) and returns `archivedSlugs` of the output.
- `archivedSlugs` parses each tab-separated line into the slug's `archiveTag`, taking the dereferenced `*objectname` and `*committerdate` when they are non-empty (an annotated tag) and the plain fields otherwise (the lightweight tags `archiveWeftTip` creates).
  It keeps a line's slug only when the ref has the `refs/tags/archive/<slug>/<sha>` shape, as now, and skips a line whose commit time does not parse as Unix seconds.
  Each slug's tags are sorted newest first by `Committed`, ties by `Name`.
- `recentRuns` treats a slug as finished when it has at least one tag.

The record (in a new file `tools/tokencount/shedstatus.go`):
- `ArchiveFiles` is the read the record needs, `FileAtRevision(rev, relPath string) ([]byte, error)`, which `*gitrepo.Repo` and the test's `fakeHistory` both satisfy.
- `ShedRecord` holds the tag it was read at (`Tag`), the seed's `started_at` as `Started` (zero when the seed has none or no seed is archived), the history's entries in recorded order as `Entries`, and `NotBefore`, the `Committed` time of the slug's next-older tag (zero when the slug has one tag).
- `shedEntry` is one history entry's `Producer` and `End`: its `last_at` when `repeats` > 0, else its `at` (Decision 3).
- `readShedRecord` returns `nil, nil`, no record, when `<hub>/<slug>` exists as a directory (the run is in flight, since teardown removes the worktree, so an older run's tag for a reused slug is never read), when `tags` is empty, or when `status.json` is absent at the newest tag (`gitrepo.ErrPathNotAtRevision`).
  Otherwise it reads `tags[0]`: the status from `shedrun.TreeStatusRel(slug)` decoded with `json.Unmarshal` into `shedengine.Status`, and the seed from `shedrun.TreeSeedRel(slug)` decoded with `shedrun.DecodeSeed`, an absent seed leaving `Started` zero.
  The in-flight check applies to every counted slug, named or picked by `recentRuns`: Decision 8 bounds it to named slugs, but an auto-picked finished slug whose worktree exists again is the same reused-slug case by Decision 8's own test, so one rule serves both and the widening is stated in `doc.go`.
  A seed or status that fails to decode, or a `started_at`, `at` or `last_at` that does not parse as RFC 3339, returns an error of the form `<slug>: <file> at <tag>: <cause>`, where `<file>` is the tree path, which aborts the report (Decision 9).
  A read error other than an absent path is a propagated raw I/O error in that same form and takes no row, since `fakeHistory` yields only data or `gitrepo.ErrPathNotAtRevision`.
  A run id `TreeStatusRel` rejects returns `<slug>: status file at <tag>: <cause>`; it is a fail-loud wiring guard, since `archivedSlugs` never yields a slug with a separator and git refuses a `..` ref component, so no test reaches it.
- `RunTally` gains `Shed *ShedRecord`, nil when no record was read.

`run` (Decision 8):
- When `-hub` is given and the -weft flag is empty, the hub is compared with `filepath.Dir(wd)`, the `-hub` default, both as absolute, cleaned paths: a relative `-hub` is joined onto `wd` and cleaned rather than passed to `filepath.Abs`, which would call `os.Getwd` (`PATTERN-cwd-resolution`).
  When they differ, `run` returns, before resolving anything, the error: -hub <hub> is not the hub of the current worktree, <dir>, so the default records repository would be another hub's; name that hub's records repository with -weft (the flag name built from `repositoryFlag`).
  This guard bounds the new default: without it a slug could be read against the wrong hub's archive tags.
  It lets through any run with the -weft flag set, and any `-hub` spelling of the current worktree's parent.
- When the -weft flag is empty, `run` resolves the default `fabricengine.RecordsWorktree` for every invocation, named slugs and no `-calibrate` included; outside a resolvable worktree `lyxcwd.Resolve`'s error returns, where named slugs ran before.
- `run` lists the tags with `finishedRuns` once, before choosing slugs, and passes them to `recentRuns` when no slug is named.
- After `CountRun` for each slug, `run` sets the tally's `Shed` from `readShedRecord(gitrepo.New(weft), hub, slug, tags[slug])`, returning its error; the calibration keeps its own `gitrepo.New(weft)` reader.

Docs: `doc.go`'s paragraphs on finished runs and on the -weft flag say the default applies whenever the flag is empty, the `-hub` refusal and its bound, that each run's archived seed and status are read from its newest archive tag by commit time, that a slug whose worktree still exists is in flight and reads no tag, and that a record that fails to decode aborts the report naming the slug, the tag and the file.

Tests:
- `TestArchivedSlugs` feeds tab-separated lines: two lightweight tags of `run-pushes` whose names sort opposite to their commit times, one annotated tag of `test-tiers` with a tag-object SHA and a dereferenced commit SHA and time, a ref with no SHA segment, a non-archive ref and a line with an unparsable time.
  It asserts the exact map: `run-pushes` with both tags newest first, `test-tiers` with the dereferenced commit and time, nothing else.
- `TestRecentRuns` builds `finished` as `map[string][]archiveTag` with one tag per finished slug and an empty slice for `empty`, and keeps its `new,mid` expectation.
- `shedstatus_test.go` holds `TestReadShedRecord`, table-driven over `fakeHistory` files keyed `<sha>:<tree path>` and a `t.TempDir()` hub, asserting the returned record field by field or the error text:
  - two tags: reads the newer tag's status and seed, `Tag` is its name, `NotBefore` is the older tag's `Committed`;
  - a folded entry (`repeats` 2, `at` 10:00, `last_at` 10:40) ends at 10:40, an unfolded one at its `at`;
  - a seed without `started_at`, and a tag with no `seed.json`: `Started` is zero and the entries are read;
  - a `<hub>/<slug>` directory beside a tag: nil record, even though an older run's tag for that slug exists;
  - no tags: nil record;
  - a tag with no `status.json`: nil record;
  - a seed with an unknown key: the error names the slug, the tag and `seed.json`'s tree path;
  - a status that is not JSON: the error names the slug, the tag and `status.json`'s tree path;
  - an `at` of `yesterday`: the error names the slug, the tag and `status.json`'s tree path.
- `shedstatus_integration_test.go` (`//go:build integration`) holds `TestArchivedShedRecordFromGit`: it inits a repository under `t.TempDir()`, commits a status and seed at the tree paths with `GIT_COMMITTER_DATE` set through `exec.Command`'s `Env`, as `internal/gitrepo`'s blob-read integration test does, tags that commit `archive/run-x/ffffffffffff`, commits a second status a day later and tags it `archive/run-x/000000000000`, so name order and date order disagree.
  Through `finishedRuns` and `readShedRecord(gitrepo.New(dir), hub, "run-x", tags["run-x"])` it asserts the second commit's entries are read, `Tag` is `archive/run-x/000000000000` and `NotBefore` equals the first commit's time, pinning the real `for-each-ref` format and the blob read against git.
- `TestRunRefusesAnotherHubByDefault`, a new top-level test in `main_test.go`, calls `run` with `-projects` set to `t.TempDir()` and slug `none`, the -weft flag unset, from the package directory the test binary runs in.
  It reads `wd` with `lyxcwd.Getwd`, never `os.Getwd` (`PATTERN-cwd-resolution`).
  Its rows: `-hub` set to a `t.TempDir()`: the error is the refusal, containing "-" + `repositoryFlag`; `-hub` set to `filepath.Dir(wd)` plus a trailing separator, and to `..`: no refusal, and the error is `lyxcwd.Resolve`'s (`errors.Is` `lyxcwd.ErrCwdOutsideAnchor` or `lyxcwd.ErrNotAGitRepo`), since the package directory is no anchor, which also pins that named slugs now need a resolvable worktree.
  Each row asserts that `run` wrote nothing to its writer.
- `cmd/lyx/testdata/test-budget.yaml`'s `tools/tokencount` row rises from 13 to 16 for `TestReadShedRecord`, `TestArchivedShedRecordFromGit` and `TestRunRefusesAnotherHubByDefault`.

**ImpactSummary:** The three re-signed functions' only callers are `run` and their own tests, all updated here; `run`'s new refusal and wider weft default change the CLI for named slugs, and `RunTally.Shed` is a new field no existing code reads.

**Commit:** 3: tokencount: read each run's archived shed status from its archive tag
