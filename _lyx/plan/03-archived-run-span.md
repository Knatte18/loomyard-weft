# Card 3 — archived-run-span

**Create:**
- `tools/tokencount/archive.go`
- `plan:tools/tokencount#archiveTag` -> `type archiveTag struct`
- `plan:tools/tokencount#producerEntry` -> `type producerEntry struct`
- `plan:tools/tokencount#archiveRecords` -> `type archiveRecords interface`
- `plan:tools/tokencount#RunTally.readArchive` -> `func (run *RunTally) readArchive(hub string, tags []archiveTag, records archiveRecords) error`
- `plan:tools/tokencount#checkHubWeft` -> `func checkHubWeft(hub, weft, wd string) error`
- `tools/tokencount/archive_test.go`
- `plan:tools/tokencount#TestReadArchive` -> `func TestReadArchive(t *testing.T)`
- `plan:tools/tokencount#TestCheckHubWeft` -> `func TestCheckHubWeft(t *testing.T)`
- `tools/tokencount/archive_integration_test.go`
- `plan:tools/tokencount#TestReadArchiveGitBacked` -> `func TestReadArchiveGitBacked(t *testing.T)`

**Edit:**
- `tools/tokencount#RunTally`
- `tools/tokencount#finishedRuns` -> `func finishedRuns(weft string) (map[string][]archiveTag, error)`
- `tools/tokencount#archivedSlugs` -> `func archivedSlugs(refs string) (map[string][]archiveTag, error)`
- `tools/tokencount#recentRuns` -> `func recentRuns(projects, hub, prime string, finished map[string][]archiveTag, last int) ([]string, error)`
- `tools/tokencount#run`
- `tools/tokencount#Report.WriteMarkdown`
- `tools/tokencount#TestArchivedSlugs`
- `tools/tokencount#TestRecentRuns`
- `tools/tokencount/main_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shedrun#TreeSeedRel`
- `plan:internal/shedrun#TreeStatusRel`
- `plan:internal/shedrun#DecodeSeed`
- `internal/shedengine#Status`
- `internal/shedengine#HistoryEntry`
- `internal/gitrepo#ErrPathNotAtRevision`
- `internal/gitrepo#Repo.FileAtRevision`
- `internal/fabricengine#RecordsWorktree`
- `internal/gitkit#MustRun`
- `internal/gitkit#Git`

**Intent:**
Read each run's archived shed `status.json` and `seed.json` from its archive tag in the weft and set the run's span from them (decision record, Decisions 3, 8 and 9), so card 2's role times and idle measure the whole run, waits before the first session and after the last included.

Archive tags.
`archiveTag` holds a tag's `Name` (`archive/<slug>/<sha>`), the tagged commit's full `SHA` and its committer time `Committed`.
`finishedRuns` runs `for-each-ref` through `gitexec.Run`, as now, with the format `%(refname)%09%(objectname)%09%(committerdate:iso-strict)`; fabric's archive tags are lightweight, so `objectname` and `committerdate` are the tagged commit's.
`archivedSlugs` parses that output into each slug's tags; a line outside `refs/tags/archive/<slug>/<sha>` is skipped as now, and a committer date that does not parse as RFC 3339 is an error naming the ref.
`recentRuns` treats a slug as finished when it has at least one tag; its behavior is otherwise unchanged.

Reading a run.
`archiveRecords` is the one-method read seam, `FileAtRevision(rev, relPath string) ([]byte, error)`, which `*gitrepo.Repo` satisfies.
`RunTally` gains `Archive string`, the tag whose status was read, `NoStatus string`, why none was read, and `Entries []producerEntry`, each history entry's `Producer` and parsed `End` in recorded order; `producerEntry` is that pair.
`readArchive`, in `archive.go`, sets them and `Span`:
- When the directory `<hub>/<slug>` exists the run is in flight, since teardown removes the worktree: no tag is read, even when an older run's tag for the slug exists, `NoStatus` says the worktree still exists, and `Span` keeps card 2's transcript window.
  This bounds the carried-over `named-inflight-slug-reads-older-archive` finding: a reused slug's older archive is read only once the new run's worktree is gone.
- With no tag, `NoStatus` says the slug has no archive tag and `Span` keeps the transcript window.
- Otherwise the tag is the one whose commit is newest by `Committed` (tag name descending on a tie), and the next-older tag's `Committed`, when there is one, is the lower bound of the start.
- `status.json` is read at the tag's SHA from `shedrun.TreeStatusRel(slug)`; absent at that revision (`gitrepo.ErrPathNotAtRevision`), the run never armed shed, so `NoStatus` says the tag holds no status and `Span` keeps the transcript window.
  It is decoded with `encoding/json` into `shedengine.Status`, and each entry's end, its `LastAt` when `Repeats` > 0 and else its `At`, parsed as RFC 3339 into `Entries`.
  A status with no history entry is treated like an absent one.
- `seed.json` is read from `shedrun.TreeSeedRel(slug)` and decoded with `shedrun.DecodeSeed`; an absent seed, like a seed with an empty `StartedAt`, gives no seeded start.
- Any other read error, a status or seed that fails to decode, and an end or `started_at` that fails to parse abort with an error naming the slug, the tag and the file's tree path.
  A `TreeSeedRel` or `TreeStatusRel` validation error takes the same wrapping as a fail-loud wiring guard: `archivedSlugs` cuts the slug at the first slash and git refuses `.` and `..` ref components, so no tag reaches it with an invalid run id, and it takes no test row.
- The start is the seed's `started_at`, else the run's `First`, else the first entry's end; it is raised to the next-older tag's `Committed` when earlier, then lowered to the first entry's end when later, so it never passes that end.
  The end is the latest entry end; `Span` is start to end and `Archive` is the tag's name.

Wiring in `run`.
`checkHubWeft`, in `main.go`, refuses when `weft` is empty and `hub`, made absolute against `wd` when relative and cleaned, differs from `filepath.Dir(wd)`, the cleaned current worktree's parent that `-hub` defaults to.
Its error names -weft as the way forward: the default weft is the current hub's, so another hub's runs need their own hub's weft passed with -weft.
It lets through every `-hub` that names the current worktree's parent, spelled with a trailing slash or as a relative path included, and every run with -weft set; it bounds the wider default below to the hub it belongs to.
`run` calls it right after defaulting `-hub`, then applies the -weft default whenever -weft is empty, slugs named or not, so outside a resolvable worktree a run with -weft empty now fails at `lyxcwd.Resolve`.
It reads the tags once with `finishedRuns`, passes them to `recentRuns` when no slug is named, and calls `readArchive` with `-hub`, the slug's tags and `gitrepo.New` of the weft for each counted run before the orch is counted.
`Report.WriteMarkdown` writes under each run's role table the tag its status was read from, or that no shed status was read and `NoStatus`, so idle there covers only the transcript window.
The package doc in `main.go` replaces the sentence limiting the -weft default to unnamed slugs or `-calibrate` with the default applying whenever -weft is empty, the `-hub` refusal, and a paragraph on the span: what it runs from and to, its two start bounds, the in-flight and no-status fallbacks.

Tests.
`TestArchivedSlugs` becomes the parse of tab-separated refs with SHAs and dates, asserting each slug's tags exactly, a skipped malformed ref, and the error on an unparseable date.
`TestRecentRuns` passes its finished set as tags; its assertions are unchanged.
`TestReadArchive`, new in `archive_test.go`, is a unit-tier table over a fake `archiveRecords` keyed by SHA and path, which returns `gitrepo.ErrPathNotAtRevision` for a missing file, over runs built with `CountRun` from synthetic transcripts, with `hub` a `t.TempDir()`:
a seed with `started_at` and a status whose folded entry's `last_at` is the latest end give that span, and the role table's idle is the span minus the union of working intervals;
a seed without `started_at` starts at the earliest transcript line, raised to the next-older tag's date when that is later, lowered to the first entry's end when that is earlier;
tags passed oldest first still read the newest commit's status;
no tag, a tag without `status.json`, and an existing `<hub>/<slug>` directory beside a tag each keep the transcript window and set `NoStatus`, the last with a fake that fails the test when read;
an undecodable seed and an undecodable status each return an error holding the slug, the tag name and the tree path.
`TestCheckHubWeft`, new in `main_test.go`, refuses another hub with -weft empty, naming -weft, and lets through the current worktree's parent spelled with a trailing slash, spelled `..`, and another hub with -weft set.
`TestReadArchiveGitBacked`, new in `archive_integration_test.go` under the `integration` build tag, builds a hermetic weft with `gitkit.MustRun` (`git init`, records written at the `shedrun` tree paths, two commits each tagged `archive/<slug>/<first 12 hex>`), asserts `finishedRuns` returns both tags with the SHA and committer time `gitkit.Git` reports for each, and asserts `readArchive` over `gitrepo.New` of the weft, given one slug's single tag, reads its status into `Span`, `Entries` and `Archive`.
The new top-level tests push `tools/tokencount` past its `test-budget.yaml` row, so the row rises to the package's new count.

**ImpactSummary:** Changes how `tools/tokencount` resolves the weft and finished runs, adds an archived-status read per counted run that sets its span, and adds one refusal for another hub without -weft; token counting and the calibration sections read the same weft as before.

**Commit:** 3: tokencount: span each run from its archived shed status
