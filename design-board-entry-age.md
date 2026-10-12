# Board entries show their age

`board-entry-age` · Note · feature, board

## Idea (operator, 2026-10-12)

Show each board entry's age, so old notes and tasks are easy to spot, weigh and prioritize: what has sat on the board a long time.

## Seen (2026-10-12)

- A board entry carries no date: `lyx board get` shows no created or updated field, and `lyx board list` has no sort or age option.
- The board lives in git (`board.json` on Bolt), so an entry's creation date can be read from the history of the commit that first added its slug, for a one-time backfill.

## Open

- Which dates: created, and whether also last updated (a body edit) or last touched by a run.
- Where age shows: a column in `lyx board list --text` and the README, a sort by age, a filter such as older than N days.
- Whether a merge keeps the oldest creation date of the entries it joins, and whether a promote keeps the note's date.
