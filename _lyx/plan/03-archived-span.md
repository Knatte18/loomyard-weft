# Card 3 — archived-span

**Create:**
- `tools/tokencount/status.go`
- `tools/tokencount/status_test.go`
- `plan:tools/tokencount#archiveTag` -> `type archiveTag struct`
- `plan:tools/tokencount#parseArchiveTags` -> `func parseArchiveTags(refs string) map[string][]archiveTag`
- `plan:tools/tokencount#archiveTags` -> `func archiveTags(weft string) (map[string][]archiveTag, error)`
- `plan:tools/tokencount#treeFiles` -> `type treeFiles interface`
- `plan:tools/tokencount#ArchivedRun` -> `type ArchivedRun struct`
- `plan:tools/tokencount#readArchivedRun` -> `func readArchivedRun(files treeFiles, slug string, tags []archiveTag) (*ArchivedRun, error)`
- `plan:tools/tokencount#ProducerTime` -> `type ProducerTime struct`
- `plan:tools/tokencount#producerTimes` -> `func producerTimes(archived *ArchivedRun, start time.Time, working []interval) []ProducerTime`
- `plan:tools/tokencount#RunTally.setSpan` -> `func (run *RunTally) setSpan(archived *ArchivedRun)`
- `plan:tools/tokencount#TestParseArchiveTags` -> `func TestParseArchiveTags(t *testing.T)`
- `plan:tools/tokencount#TestArchivedRunTiming` -> `func TestArchivedRunTiming(t *testing.T)`

**Edit:**
- `tools/tokencount#RoleTally`
- `tools/tokencount#RunTally`
- `tools/tokencount#CountRun`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shedrun#SeedTreeRel`
- `plan:internal/shedrun#StatusTreeRel`
- `plan:internal/shedrun#DecodeSeed`
- `internal/shedengine#Status`
- `internal/shedengine#HistoryEntry`
- `tools/tokencount#fakeHistory`

**Intent:**
Read a finished run's archived seed and status and turn them into the run's span, role times, idle and Producers rows (Decisions 2, 3, 4, 5, 8, 9).
The production code goes in the new file `status.go`; its tests go in the new file `status_test.go`, tier 1.

Archive tags:
- `archiveTag` holds a tag's `Name` (`archive/<slug>/<sha>`), the full `SHA` of its commit and the commit's `Committed` time.
- `archiveTags(weft)` runs `gitexec.Run` with `for-each-ref --format=%(refname) %(objectname) %(committerdate:iso-strict) refs/tags/archive/` in the repository at `weft` (the archive tags are lightweight, so the object is the commit) and returns `parseArchiveTags` of its output; its error keeps `finishedRuns`'s wording, `reading the archive tags in <dir>: …`.
- `parseArchiveTags` maps each slug to its tags sorted newest `Committed` first, whatever order the lines come in (`for-each-ref` lists by name), skipping a line whose ref has no `<slug>/<sha>` rest or whose date does not parse.
- `finishedRuns` and `archivedSlugs` stay until card 5, which moves `run` onto `archiveTags` and deletes them with `TestArchivedSlugs`.
- `TestParseArchiveTags` drives `parseArchiveTags` with `TestArchivedSlugs`'s refs, each with a SHA and a date: the two `run-pushes` lines list the older-dated tag first, so a parser that keeps input order fails, and a line with an unparseable date joins the dropped `no-sha` and `other` lines.
  It asserts the two slugs, `run-pushes` holding its two tags newest first with their SHAs and times.

Reading:
- `treeFiles` is the one method `FileAtRevision(rev, relPath string) ([]byte, error)`, which `*gitrepo.Repo` and the test's `fakeHistory` satisfy.
- `ArchivedRun` holds the read tag's name, the decoded `shedengine.Status`, the seed's parsed `started_at` (zero when absent), each history entry's parsed end, and `Older`, the committed time of the slug's next-older tag (zero when none).
- `readArchivedRun` returns nil for no tags.
  Otherwise it takes `tags[0]`, the newest, reads `shedrun.StatusTreeRel(slug)` at its SHA and decodes it with `json.Unmarshal` into `shedengine.Status`.
  A status absent at the tag (`gitrepo.ErrPathNotAtRevision`), or one with no history entry, returns nil: the run takes the no-status fallback.
  It then reads `shedrun.SeedTreeRel(slug)` and decodes it with `shedrun.DecodeSeed`; an absent seed leaves `started_at` zero.
  An entry's end is its `last_at` when `repeats` > 0, else its `at`.
  A read error other than absence, a status or seed that fails to decode, and an `at`, `last_at` or `started_at` that fails to parse as RFC 3339 each return an error naming the slug, the tag and the file, `<slug> at <tag>: <path>: <cause>`.
  An error from `StatusTreeRel` or `SeedTreeRel` is returned the same way as a fail-loud wiring guard with no row: every slug with tags comes from `parseArchiveTags`, whose slug is one ref segment, which git never lets be empty, `.` or `..` and which holds no `/`, so `ValidateRunID` accepts it.

Span and times:
- `RoleTally` gains `Time time.Duration`; `RunTally` gains `Start, End time.Time`, `Idle time.Duration`, `Archived *ArchivedRun` and `Producers []ProducerTime`.
- `setSpan(nil)` sets the span to `First`..`Last`; `CountRun` calls it last, so every counted run has the transcript-window span until card 5 re-sets it from an archive.
- `setSpan(archived)` sets `End` to the latest entry end and `Start` to the seed's `started_at`, or to `First` when the seed has none; either start is then raised to `Older` when `Older` is later and lowered to the first entry's end when that is earlier, in that order.
- Then for every role with intervals, `Roles[role].Time = covered(Working[role], Start, End)`, and `Idle` is the span minus `covered` of every role's intervals together, orch never among them.
- `ProducerTime` holds a producer name, its entry count, `Elapsed` and `Idle`.
  `producerTimes` walks the history in recorded order with a cursor at the span start: an entry whose end is after the cursor owns `[cursor, end]` and moves the cursor there, and any other entry owns nothing.
  Per producer, in first-appearance order, it sums entries, elapsed and the part of each owned interval `covered` leaves uncovered; elapsed sums to the span because the start is never after the first entry's end.
  `setSpan(archived)` stores it in `Producers`; `setSpan(nil)` leaves it nil.

`TestArchivedRunTiming`, one table test whose subtests build `RunTally` values with `Working` set by hand and `fakeHistory` files keyed `<sha>:<path>` from `shedrun.StatusTreeRel` and `SeedTreeRel`:
- Two tags: the first slice element's status is the one read, and `Older` is the second tag's time.
- No tags, a status absent at the tag, and an empty history each give nil.
- A status present with no seed at the tag reads with a zero `started_at`.
- A seed with an unknown key, a status that is not JSON, an entry `at` of `"nope"`, a folded entry's `last_at` of `"nope"` and a seed `started_at` of `"nope"` each return an error containing the slug, the tag name and the file path.
- Span with `started_at`: the end is a folded entry's `last_at`, later than every `at`; role times are the union clipped to the span, and idle is the span minus the union of all roles.
- A `started_at` earlier than `Older` starts the span at `Older`.
- Seed without `started_at`: the start is `First`; with `First` before `Older` it is `Older`; with `First` after the first entry's end it is that end.
- `setSpan(nil)` gives `First`..`Last` and no producers.
- Producers: history Preflight, Plan-Write, an out-of-order Preflight whose end is before Plan-Write's, Webster; elapsed sums to the span, the out-of-order entry owns nothing but still counts as an entry, and each producer's idle is the part of its intervals no working interval covers.

`cmd/lyx/testdata/test-budget.yaml` raises the `tools/tokencount` row by two for `TestParseArchiveTags` and `TestArchivedRunTiming`.

**ImpactSummary:** additive; `CountRun` now ends with `setSpan(nil)`, which fills new fields and changes no token figure, and no caller reads the archive yet.
