# Card 3 — The run's span from its archived shed status

**Create:**
- `plan:tools/tokencount#archiveTag` -> `type archiveTag struct`
- `plan:tools/tokencount#ArchivedEntry` -> `type ArchivedEntry struct`
- `plan:tools/tokencount#ArchivedRun` -> `type ArchivedRun struct`
- `plan:tools/tokencount#ArchiveReader` -> `type ArchiveReader interface`
- `plan:tools/tokencount#entryEnd` -> `func entryEnd(e shedengine.HistoryEntry) (time.Time, error)`
- `plan:tools/tokencount#RunTally.ReadArchive` -> `func (run *RunTally) ReadArchive(records ArchiveReader, hub string, tags []archiveTag) error`
- `plan:tools/tokencount#refuseForeignHub` -> `func refuseForeignHub(hub, wd string) error`
- `tools/tokencount/archive_test.go`
- `plan:tools/tokencount#fakeArchive` -> `type fakeArchive map[string]string`
- `plan:tools/tokencount#fakeArchive.FileAtRevision` -> `func (f fakeArchive) FileAtRevision(rev, relPath string) ([]byte, error)`
- `plan:tools/tokencount#TestReadArchive` -> `func TestReadArchive(t *testing.T)`
- `plan:tools/tokencount#TestRefuseForeignHub` -> `func TestRefuseForeignHub(t *testing.T)`
- `tools/tokencount/archive_integration_test.go`
- `plan:tools/tokencount#TestRunReadsArchivedStatus` -> `func TestRunReadsArchivedStatus(t *testing.T)`

**Edit:**
- `tools/tokencount#finishedRuns` -> `func finishedRuns(weft string) (map[string][]archiveTag, error)`
- `tools/tokencount#archivedSlugs` -> `func archivedSlugs(refs string) map[string][]archiveTag`
- `tools/tokencount#recentRuns` -> `func recentRuns(projects, hub, prime string, finished map[string][]archiveTag, last int) ([]string, error)`
- `tools/tokencount#RunTally`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#repositoryFlag`
- `tools/tokencount#run`
- `tools/tokencount#TestArchivedSlugs`
- `tools/tokencount#TestRecentRuns`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shedrun#RunFilesRel`
- `plan:internal/shedrun#DecodeSeed`
- `plan:tools/tokencount#RunTally.SetSpan`
- `internal/shedengine#Status`
- `internal/gitrepo#Repo.FileAtRevision`

**Intent:**
Read each finished run's archived shed `status.json` and `seed.json` from its archive tag in the weft and span the run from its start to its last history entry's end, replacing card 2's transcript-window span where a status is read (Decisions 3, 6, 8 and 9).
No start timestamp is added to shed history entries (Decision 6): the history's finish times are the whole input.

Archive tags:
- `archiveTag` holds a tag's `Name` (`archive/<slug>/<sha12>`), its commit `SHA` and the commit's `Committed` time.
  Archive tags are lightweight (`internal/fabricengine/archive.go` runs `git tag <tag> <tip>`), so the ref's object is the commit.
- `finishedRuns` runs `gitexec.Run` with `for-each-ref --format=%(refname) %(objectname) %(committerdate:iso-strict) refs/tags/archive/` (`PATTERN-gitexec-checked-call`) and returns `archivedSlugs` of the output.
- `archivedSlugs` groups the lines into tags per slug; a line without a `<slug>/<sha>` part under `refs/tags/archive/`, without three fields, or whose date fails `time.Parse(time.RFC3339, ...)` is skipped, as a malformed ref is now.
- `recentRuns` takes that map and counts a slug as finished when it has a tag.

Reading, in a new file `tools/tokencount/archive.go` holding every new non-test symbol of this card but `refuseForeignHub`:
- `ArchiveReader` is the one method `FileAtRevision(rev, relPath string) ([]byte, error)`, which `*gitrepo.Repo` satisfies.
- `ArchivedEntry` is one history entry's `Producer` and `End`; `ArchivedRun` holds the read tag's `Name` as `Tag` and the entries in recorded order as `Entries`.
- `entryEnd` parses an entry's end with `time.RFC3339`: its `LastAt` when `Repeats > 0`, else its `At`.
- `RunTally` gains `Archive *ArchivedRun`, nil when no status was read, and `NoStatus string`, why not.
- `ReadArchive(records, hub, tags)` does, in order:
  1. It takes `seedRel, statusRel` from `shedrun.RunFilesRel(run.Slug)` first, so a named slug that is no valid run id, such as `a/b`, aborts the report whatever its tags: the error names the slug and wraps the validation error.
     It names no tag or file, a deviation from Decision 9's wording forced by the input: `archivedSlugs` keys a slug on the text before the first `/`, so no tag can belong to such a slug, and no file path can be formed from it.
  2. When `<hub>/<slug>` is an existing directory, the run is in flight, since teardown removes the worktree: it sets `NoStatus` to say the worktree still exists and returns nil, reading no tag even when older runs' tags for the slug exist (Decision 8's bound).
  3. With no tags it sets `NoStatus` to "no archive tag" and returns nil.
  4. It picks the newest tag by `Committed` (ties broken by the greater `Name`) and the next-older one, the newest of the rest, when there is one; the slice's order does not matter.
  5. It reads `statusRel` at the newest tag's `SHA`.
     `gitrepo.ErrPathNotAtRevision` sets `NoStatus` to name the missing file and the tag and returns nil; any other read error returns an error.
  6. It decodes the bytes with `json.Unmarshal` into `shedengine.Status`, unknown fields allowed since archived statuses span shapes, and computes every entry's end with `entryEnd`; a decode or parse failure returns an error.
     An empty history sets `NoStatus` to say the status records no history and returns nil.
  7. It reads `seedRel` at the same `SHA`; an absent file means no `started_at`, other read errors return an error, and the bytes decode through `shedrun.DecodeSeed`, whose failure, or a `StartedAt` that fails `time.RFC3339`, returns an error.
  8. The start is the seed's `StartedAt`, else `run.First`; it is raised to the next-older tag's `Committed` when earlier, and lowered to the first entry's end when later or still zero.
     The end is the latest entry end.
     It sets `Archive` and calls `run.SetSpan(start, end)`.
  Every error from steps 5 to 7 names the slug, the tag's `Name` and the file's relative path, for example `alpha: archive tag archive/alpha/0123456789ab: _lyx/shed/alpha/status.json: <cause>`, built from `seedRel`/`statusRel`, so the tool spells no `_lyx`, `shed` or `seed.json` literal.
  A run whose `ReadArchive` sets `NoStatus` keeps card 2's transcript-window span.

`run` in `main.go`:
- The weft default now applies whenever the weft flag is empty, named slugs included (Decision 8).
  Before resolving it, `run` calls `refuseForeignHub(*hub, wd)`, after the `-hub` default is applied, so only an explicit other hub can be refused.
- `refuseForeignHub` joins a relative `hub` onto `wd`, cleans it, and compares it with `filepath.Dir(wd)`, so the same hub spelled with a trailing slash or as a relative path passes.
  A different directory returns an error naming `-hub`, this worktree's hub and the weft flag as the way forward: name the weft repository of the hub `-hub` names.
  What it lets through: with the weft flag set, any `-hub` is accepted and the operator owns the pairing; without it, only this worktree's hub is read against this worktree's weft.
  It avoids `filepath.Abs`, which calls `os.Getwd` (`PATTERN-cwd-resolution`).
- Outside a resolvable worktree `lyxcwd.Resolve` fails, so a run with the weft flag empty now fails there with named slugs too, as Decision 8 accepts.
- `run` calls `finishedRuns(*weft)` once for every invocation, passes the tags to `recentRuns` when no slugs are named, and after each `CountRun` calls `ReadArchive(gitrepo.New(*weft), *hub, tags[slug])`, returning its error, which aborts the report.
  The calibration reuses that `gitrepo` handle.
- The weft flag's usage text and `repositoryFlag`'s comment add the runs' shed status to what the repository holds.

`Report.WriteMarkdown` prints, below a run's role table when `Archive` is nil and `NoStatus` is set, `No shed status was read: <NoStatus>; the span is the run's transcript window, so idle covers only that window.`
`run` calls `ReadArchive` for every counted run, which sets one of the two, so a run built by `CountRun` alone, as in `TestCountRun` and `TestCountOrch`, prints no note.

Package doc in `main.go`: the weft flag's sentences (finished runs and the "one repository flag" paragraph) say the default applies whenever the flag is empty, and that with `-hub` naming another hub the weft flag must be named;
the usage example `go run ./tools/tokencount -hub ~/Code/loomyard-LYXHUB -out .scratch/tokens.md test-suite-prune-core`, which the refusal now refuses outside that hub's prime, gains the weft flag naming that hub's weft repository; card 2's span sentence becomes Decision 3's span (seed `started_at` or the earliest transcript line, the two bounds, the latest entry end with a folded entry ending at its `last_at`), the newest tag by committer date, the in-flight rule, the transcript-window fallback with its note, and the abort naming slug, tag and file.

Tests:
- `TestReadArchive`, new in `archive_test.go`, table-driven.
  Each row builds a run by `CountRun` over one burler transcript working 10:00-10:30 and 11:00-11:10 (`First` 10:00, `Last` 11:10), or, where the row says untimed, over one assistant line with no timestamp (`First` zero); a closure inside the test writes the transcript.
  It fills a `fakeArchive` keyed `<sha>:<relPath>` at the paths `shedrun.RunFilesRel` gives, whose `FileAtRevision` returns `gitrepo.ErrPathNotAtRevision` for a missing key and an error for a value starting `error:`, and passes a `t.TempDir()` hub and the row's tags.
  It wants `SpanStart`, `SpanEnd`, `Idle`, burler `Time`, `Archive.Tag`, `NoStatus`, or an error holding every substring of the row's list. Rows:
  - a seed with `started_at` 09:00 and a history whose first entry ends 09:30 and whose last entry is folded (`at` 10:40, `repeats` 2, `last_at` 11:20): span 09:00-11:20, burler 40m, idle 1h40m;
  - a seed without `started_at` and a first entry ending 10:20: span starts at the earliest transcript line, 10:00;
  - no seed file, an older tag committed 10:10, a first entry ending 10:20 and a last ending 11:10: span 10:10-11:10, burler clipped to 30m, idle 30m;
  - no `started_at` and a first entry ending 09:30: start 09:30, not 10:00;
  - untimed, no `started_at`, no older tag and a first entry ending 09:30: start 09:30, not zero;
  - two tags passed oldest first, at different SHAs whose statuses end 12:00 and 11:20, the newer committed later: the newer tag's status is read, span end 11:20 and `Archive.Tag` the newer name;
  - no tags: `Archive` nil, `NoStatus` "no archive tag", span 10:00-11:10, burler 40m kept;
  - `<hub>/<slug>` created as a directory, with a tag whose status is in the fake: `Archive` nil, `NoStatus` names the existing worktree, span 10:00-11:10;
  - a tag whose tree lacks the status: `Archive` nil, `NoStatus` names the tag;
  - a status holding an empty history: `Archive` nil;
  - a status read failing (`error:` value): an error holding the slug, the tag name and `statusRel`;
  - status bytes `{`: an error holding the slug, the tag name and `statusRel`;
  - a history entry whose `at` is `yesterday`: an error holding `statusRel`;
  - seed bytes `{"recipe":"loom","bogus":1}`: an error holding the slug, the tag name and `seedRel`;
  - seed bytes `{"recipe":"loom","started_at":"soon"}`: an error holding the slug, the tag name and `seedRel`;
  - slug `a/b` (passed to `CountRun` as the slug) with no tags: an error naming `a/b`, which step 1 reaches before the no-tags step.
- `TestRefuseForeignHub`, new in `main_test.go`: rows over `refuseForeignHub` with `base` a `t.TempDir()` and `wd` `filepath.Join(base, "prime")`, so every path is absolute on every platform: `filepath.Join(t.TempDir(), "other")` refused with the weft flag's name, dash included, in the error; `base` plus a trailing separator, `..` and `filepath.Join(wd, "..")` passed.
  Two assertions drive `run` from the package directory with `-projects` a `t.TempDir()` and one slug, each wanting nothing written: with `-hub` another `t.TempDir()`, the weft flag refusal, reached before `lyxcwd.Resolve`; with no `-hub`, an error that is not the refusal, `lyxcwd.Resolve`'s failure, since the package directory is not an anchored worktree root (Decision 8's last bound).
- `TestRunReadsArchivedStatus`, new in `archive_integration_test.go` (`//go:build integration`), builds a weft with `gitkit.MustRun`/`gitkit.Git`: a commit holding `alpha`'s seed and status at `shedrun.RunFilesRel("alpha")`, tagged `archive/alpha/<sha12>`, and a commit holding `beta`'s, tagged likewise, with `<hub>/beta` created as a directory; `gamma` has no tag.
  It writes the three runs' transcripts under a `t.TempDir()` projects directory, calls `run` with the weft flag, `-hub` and `-projects` naming the fixtures, and asserts that the report starts with `# Token usage and time by role`, that alpha's role table carries the idle row the status span gives, that beta's section says `No shed status was read` with the in-flight reason, and that gamma's says `No shed status was read: no archive tag` below a role row with its transcript time, and so that the real `for-each-ref` output parses.
- `TestArchivedSlugs`: its input lines gain the object name and an `iso-strict` date, and it asserts the tags per slug with `Name`, `SHA` and `Committed`, plus rows for a line without a date and a line whose date does not parse, both skipped.
- `TestRecentRuns`: `finished` becomes a map of one tag per finished slug.
- `TestCountRun` and `TestCountOrch` render runs with neither `Archive` nor `NoStatus`, which print no note, so neither changes.
- `cmd/lyx/testdata/test-budget.yaml`: `tools/tokencount` rises from 13 to 16.

**ImpactSummary:** all changes are inside `tools/tokencount`; the three re-signed helpers' callers are `run` and their own tests, all listed, and the observable change is a git read and a decode per counted run plus a new refusal for a foreign `-hub` without the weft flag.

**Commit:** 3: tokencount: span each run from its archived shed status
