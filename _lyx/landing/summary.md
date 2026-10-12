# Board entries carry created and revised dates; views show age

Every board entry now stores when it was created and when its scope last changed, and the README and `lyx board list --text` show both as a compact age, so long-lying entries stand out.

## What changes

- `board.json` entries gain `created` and `revised`, in whole UTC seconds.
  Neither key is written while the date is unknown.
- The write critical section in `internal/boardengine` stamps the dates after the mutation: a new entry gets the current time for whichever date it lacks, and an entry whose scope changed gets it as `revised`.
  Scope is what the run lock guards (title, brief, body, kind, labels, priority, dependencies), so a status change from a run is not a revision.
- No age is stored.
  `FormatAge` renders it at display time in one unit, such as `3d` or `5w`.
- `lyx board list --text` gets an age column and a time-since-revision column, each omitted when no listed entry has that date.
  `list` takes `--sort age` or `--sort revised`, oldest first, with entries lacking the date last.
- The README opens with the UTC date and time it was rendered, and each row gets Age and Revised cells, so the ages read as of that moment.
- `upsert` accepts `created` and `revised` as RFC 3339 times, which is how a backfill writes them; the run lock lets such a write through on a held entry.
- A merge gives the surviving entry the oldest `created` among the entries it joins.
- `boardengine/doc.go`, `boardcli/doc.go` and the board skill document the dates.

## Design decisions

- Stamping runs against the entries as loaded, in one place, so no mutation has to remember to set a date.
- A promote keeps the note's dates: the engine marks the promoted slug, and only that entry's kind change is exempt from revising.
  An upsert that flips a note's kind is a normal revision.
- Prune dropping a done dependency from a survivor is not a revision, so pruning leaves the survivors' dates alone.
- An existing entry without `created` keeps none; no write can know when it was added, and views show it blank.
- `Config.Clock` supplies the time, defaulting to the wall clock, so tests pin it.
- Ages use fixed unit boundaries (minutes under an hour, hours under a day, days under 14, weeks under 91 days, then 30-day months, then years); a blank date renders empty and a future date clamps to `0m`.

## To run by hand

- The one-time backfill of dates for every existing entry is not part of this diff.
  It reads the board's git history (first commit adding each slug, last commit changing its scope) and writes the dates through `lyx board upsert` with `created` and `revised`, never by editing `board.json`.
- The open question of an older-than-N-days filter on `list` is left out; only the sorts were added.
