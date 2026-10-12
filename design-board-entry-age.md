# Board entries carry created and revised dates; views show age

`board-entry-age` · Task · feature, board · running · Darn-Bouncer

## Idea (operator, 2026-10-12)

Show each board entry's age, so old notes and tasks are easy to spot, weigh and prioritize: what has sat on the board a long time.

## Seen (2026-10-12)

- A board entry carries no date: `lyx board get` shows no created or updated field, and `lyx board list` has no sort or age option.
- The board lives in git (`board.json` on Bolt), so each entry's dates can be read from the history of the commits that touched its slug.

## Build (operator, 2026-10-12; darn)

- Each entry stores two dates in `board.json`: when it was created and when it was last revised (any change to its scope: title, brief, body, kind, labels, priority, dependencies; a status change from a run is not a revision).
- The rendered views never store an age: `lyx board list --text` and the README compute it from the dates when they render, shown compactly (such as `3d`, `5w`) as two columns, age and time since last revision.
- The README shows ages, not dates (operator: dates are far harder to read at a glance), and states at its top the date and time it was rendered, so its ages read as of that moment.
- A merge keeps the oldest creation date of the entries it joins; a promote keeps the note's dates.
- A one-time backfill of the dates of every existing entry: the task spawns a Haiku 5.5 agent that reads the board's git history (the first commit that added each slug, and the last that changed its scope) and writes the dates through `lyx board`, never by editing `board.json` by hand.

## Open

- Whether `list` gets a sort by age or a filter such as older than N days.
