# Card 3 — Archive tags carry their commit and commit date

**Create:**
- `plan:tools/tokencount#ArchiveTag` -> `type ArchiveTag struct`

**Edit:**
- `tools/tokencount#archivedSlugs` -> `func archivedSlugs(refs string) map[string][]ArchiveTag`
- `tools/tokencount#finishedRuns` -> `func finishedRuns(weft string) (map[string][]ArchiveTag, error)`
- `tools/tokencount#recentRuns` -> `func recentRuns(projects, hub, prime string, finished map[string][]ArchiveTag, last int) ([]string, error)`
- `tools/tokencount#run`
- `tools/tokencount/count_test.go`

**Uses:**
- `internal/gitexec#Run`

**Intent:**
Read each finished run's archive tags with the commit each names and that commit's committer date, so card 4 can pick a slug's newest tag and bound a run's start at its next-older tag (decision record, Decisions 3 and 8).

`ArchiveTag` holds a tag's `Name` (`archive/<slug>/<sha>`), the full `Commit` SHA it names and the commit's `Committed` time.
`finishedRuns` keeps its single `gitexec.Run` of `for-each-ref` over `refs/tags/archive/`, with the format `%(refname)%09%(objectname)%09%(committerdate:iso-strict)`; archive tags are lightweight, as `fabricengine`'s `archiveWeftTip` creates them, so the object name is the commit.
`archivedSlugs` parses that output: it keys each tag by its slug and orders each slug's tags newest first by `Committed`, ties broken by `Name`, so the order is deterministic.
A line that lacks the `refs/tags/archive/<slug>/<sha>` shape, a commit or a date that parses as RFC 3339 is skipped, as a malformed ref is skipped today.
`recentRuns` counts a slug as finished when it has at least one tag.
`run`, the caller of both re-signed functions, passes `finishedRuns`'s map straight to `recentRuns`, so its body compiles unchanged and is listed only as that caller.

Tests in `count_test.go`: `TestArchivedSlugs` becomes a table over `archivedSlugs` that pins a slug with two tags in newest-first order, two tags of one date ordered by name, and the skipped malformed lines (no sha segment, another namespace, a well-shaped ref with an empty object-name field and a valid date, a missing date); `TestRecentRuns` passes its finished slugs as a map of tags.

**ImpactSummary:** The finished-run lookup now also yields each tag's commit and date; which runs count as finished is unchanged for the lightweight tags lyx creates, and `run` is the only production caller of `finishedRuns` and `recentRuns`.
