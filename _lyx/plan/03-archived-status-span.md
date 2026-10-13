# Card 3 — archived-status-span

**Create:**
- `plan:tools/tokencount#archiveTag` -> `type archiveTag struct`
- `plan:tools/tokencount#StatusRecords` -> `type StatusRecords interface`
- `plan:tools/tokencount#ArchivedStatus` -> `type ArchivedStatus struct`
- `plan:tools/tokencount#readArchivedStatus` -> `func readArchivedStatus(records StatusRecords, slug string, tags []archiveTag) (*ArchivedStatus, error)`
- `plan:tools/tokencount#entryEnd` -> `func entryEnd(e shedengine.HistoryEntry) (time.Time, error)`
- `plan:tools/tokencount#ProducerTime` -> `type ProducerTime struct`
- `plan:tools/tokencount#producerTimes` -> `func producerTimes(history []shedengine.HistoryEntry, ends []time.Time, span interval, work []interval) []ProducerTime`

**Edit:**
- `tools/tokencount#finishedRuns` -> `func finishedRuns(weft string) (map[string][]archiveTag, error)`
- `tools/tokencount#archivedSlugs` -> `func archivedSlugs(refs string) map[string][]archiveTag`
- `tools/tokencount#run`
- `tools/tokencount#RunTally`
- `plan:tools/tokencount#RunTally.Timing`
- `plan:tools/tokencount#RunTiming`
- `tools/tokencount#TestArchivedSlugs`
- `plan:tools/tokencount#TestRunTime`

**Uses:**
- `plan:internal/shedrun#TreeSeedRel`
- `plan:internal/shedrun#TreeStatusRel`
- `plan:internal/shedrun#DecodeSeed`
- `internal/shedengine#Status`
- `internal/shedengine#HistoryEntry`

**Intent:**
Read a finished run's archived shed status and seed (Decisions 8 and 9), set the run's span from them (Decision 3) and split the span by history entry into the Producers figures (Decision 5).
No change to `internal/shedengine`'s status shape (Decision 6).

New file `tools/tokencount/status.go` holds:
- `archiveTag`: `Name` (the tag below `refs/tags/`), `SHA` (the full hex commit it names) and `Committed` (that commit's committer time).
- `StatusRecords`, the read side the status needs: `FileAtRevision(rev, relPath string) ([]byte, error)`, which `*gitrepo.Repo` and the test fake `fakeHistory` both satisfy.
- `ArchivedStatus`: `Tag string`, `Seed shedrun.Seed`, `Status shedengine.Status` and `NotBefore time.Time`, the committer time of the slug's next-older tag, zero when there is none.
- `readArchivedStatus`: with no tag it returns nil and no error.
  Otherwise it takes the tag whose `Committed` is newest (ties broken by `Name`), and the next-older one's `Committed` as `NotBefore`.
  A `TreeStatusRel` or `TreeSeedRel` refusal of the slug (wrapped with the slug and the tag, since no file path exists to name), a seed that `DecodeSeed` refuses, a status that `json.Unmarshal` into `shedengine.Status` refuses, and a history entry whose end does not parse each return an error naming the slug, the tag and the file's repo-relative path.
  A `status.json` absent at the tag (`gitrepo.ErrPathNotAtRevision`) returns nil, the no-status fallback; an absent `seed.json` leaves `Seed` zero, so the span starts as for a seed without `started_at`.
  Any other read error is returned wrapped with the slug and the tag.
- `entryEnd`: `LastAt` when `Repeats` > 0, else `At`, parsed as RFC 3339.
- `ProducerTime`: `Producer string`, `Entries int`, `Elapsed` and `Idle time.Duration`.
- `producerTimes`: walks `history` in recorded order with each entry's end from `ends`; entry i owns the interval from the latest end among entries before it (the span start for the first) to its own end, empty when its end is not later; `Elapsed` sums the owned intervals per producer and `Idle` sums the part of them `work` (already unioned) does not cover.
  Rows come in order of each producer's first entry.

`finishedRuns` lists the archive tags with one `gitexec.Run` of `for-each-ref` over `refs/tags/archive/` whose format carries the ref name, the object name, the peeled object name, and the committer date and peeled committer date as Unix seconds, preferring a peeled field when it is set, so a lightweight and an annotated tag both resolve to their commit.
`archivedSlugs` parses that output into tags per slug, skipping a line that is not `refs/tags/archive/<slug>/<sha>` or whose date does not parse.
`run`, the one caller of `finishedRuns`, builds the slug set `recentRuns` takes from the map's keys; `recentRuns` is unchanged.

`RunTally` gains `Status *ArchivedStatus`, nil when no status was read; card 5 sets it.
`RunTiming` gains `HasStatus bool` and `Producers []ProducerTime`.
`RunTally.Timing` with a status: the span ends at the latest entry end;
it starts at the seed's `started_at` when set, else at `First`, then is raised to `NotBefore` when that is later and lowered to the first history entry's end when that is earlier.
Without a status, or with a status whose history is empty, it keeps card 2's `First`..`Last` span with `HasStatus` false and no producers; card 4 words the two cases apart through `run.Status`.
Since `readArchivedStatus` already checked every entry's end, `Timing` re-parses them without an error path.

Tests:
- `TestArchivedSlugs` feeds the new format: two tags of one slug with their SHAs and dates, an annotated tag whose peeled fields win, and the malformed lines it already skips, and asserts the exact tags per slug.
- `TestRunTime` gains rows over a run whose `Status` the row sets on the `CountRun` result:
  - a seed with `started_at` and a history whose last entry is folded (`repeats` 2, `last_at` later than every `at`): the span runs from `started_at` to that `last_at`, and idle is the span minus the union of all working intervals;
  - a seed without `started_at` whose earliest transcript line precedes `NotBefore`: the span starts at `NotBefore`; a second row whose earliest line follows the first entry's end: the span starts at that end;
  - a seed whose `started_at` precedes `NotBefore`: the span starts at `NotBefore`, so the bound applies to a stamped start too;
  - entries out of order (the second's end earlier than the first's) and a producer named twice: the producers' `Elapsed` sums to the span, the out-of-order entry owns nothing, and each producer's `Idle` is the part of its owned intervals no transcript works in;
  - no status: `HasStatus` false, the transcript-window span and no producers.
- `TestRunTime` also drives `readArchivedStatus` over a `fakeHistory`: two tags, the newer one's status read and the older one's `Committed` as `NotBefore`; no tag gives nil; a status absent at the tag gives nil; a seed absent gives a zero `Seed`; a seed, a status, and an entry end that fail to decode each return an error containing the slug, the tag and the file path;
  and the slug `a/b` with one tag returns `TreeStatusRel`'s refusal wrapped with the slug and the tag, the input that reaches the run-id branch, since through `run` a tagged slug never holds a separator.

**ImpactSummary:** `finishedRuns` and `archivedSlugs` change shape with one production caller (`run`) and one test (`TestArchivedSlugs`), both on this card; the slug set `recentRuns` sees is unchanged and nothing renders the new timing until card 4.
