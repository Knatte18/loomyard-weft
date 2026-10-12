# Board entries carry created and revised dates; views show age

Board entries now record when they were created and when their scope was last revised.
The README and `lyx board list --text` show each entry's age and the time since its last revision, so entries that have sat on the board a long time stand out.

## Dates on entries

`Task` and `BriefTask` gain `created` and `revised`, RFC 3339 times in whole UTC seconds.
`board.json` omits a date while it is unknown, so existing entries load unchanged.

The write critical section in `boardengine.Board` sets the dates after each mutation, comparing the result with the entries loaded under the lock (`stampDates` in `internal/boardengine/dates.go`).
Doing it there covers every write path at once: upsert, batch, merge, set-deps, promote, set-status and intake.

- A new entry gets the write time for whichever date it lacks.
- An entry whose scope changed gets the write time as `revised`, unless the write set `revised` itself.
  "Scope" is the run lock's existing `scopeChanged`, which now also ignores the two dates.
- A status change is not a revision.
  Neither is prune dropping a removed done dependency, nor a note becoming a task, so a promote keeps the note's dates.
- An existing entry with no `created` date keeps none, since no write can know when it was added.
- A merge gives the upserted entry the oldest known `created` date among itself and the entries it removes.

`upsert` accepts `created` and `revised`, which is how a backfill writes them.
The run lock lets such a write through on a run-held entry, because the dates are not scope.
`Config.Clock` sets the time source, defaulting to the wall clock, and the tests use it to inject a clock.
The upsert, merge, promote and import results are read back after the dates are set, so they carry their dates.

## Views

The views store no age; each computes ages when it renders, through `boardengine.FormatAge`, as one compact unit (`45m`, `5h`, `3d`, `5w`, `4mo`, `2y`).

- The README opens with `Rendered <date> <time> UTC; every age below is as of then.`
  Every table gains `Age` and `Revised` columns before `Labels`, left blank for an unknown date.
  `Render` and `RenderToDisk` now take the render time.
- `list --text` and `find --text` add the same two columns after the priority column.
  As with priority, each column is dropped when no listed entry has that date.
- `list --sort age` lists the oldest entries first, and `list --sort revised` lists the longest unrevised first.
  Entries without the date go last, ties keep README order, and any other value is refused.

## Backfill, after deploy

The entry's one-time backfill writes the dates of every existing entry from the board's git history, through `lyx board upsert`.
It is not part of this diff.
It needs the deployed binary, because today's production `lyx board upsert` refuses `created` and `revised` as unknown fields.
Once this lands and is deployed, the orch spawns a Haiku 5.5 agent that, for each slug in `board.json` on Bolt:

1. Takes the time of the first commit that added the slug as `created`.
2. Takes the time of the last commit that changed its scope as `revised`: title, brief, body, kind, labels, priority or dependencies, with status-only changes not counted.
3. Writes both with `lyx board upsert '{"slug":"<slug>","created":"<RFC 3339>","revised":"<RFC 3339>"}'`.

Until then, existing entries show blank ages.
Entries created or revised after the deploy get dates right away.

## Docs and tests

- The `boardengine` doc gains a Dates section and covers the README's render line and the new columns.
- The `boardcli` doc, the `list`, `find` and `upsert` help text, and the `ly:board` skill describe the age columns, `--sort` and the date fields.
- `TestEntryDates` covers stamping across a new entry, a status change, a scope change, a promote, a backfill on a run-held entry and a merge.
- The README goldens, `TestRenderCompact` and the CLI `list --text` step are extended.
  The `internal/boardengine` test budget goes up by one.
