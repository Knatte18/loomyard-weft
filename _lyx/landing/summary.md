# report: map each darn run's archive tag to its own landing

`lyx report convergence` no longer maps several darn runs' archive tags to one older landing and reports a "claim one landing commit" clash.
Each darn tag now falls to the time join and maps to its own landing.

Closes #831

## Why

A darn run writes no task-commit trailer of its own.
The exact join walked the first-parent history below the tag for the newest trailer.
A pair's records branch forks from its parent's, so that walk ran past the run into the parent's history and found a trailer the parent's sync commit wrote for an earlier landing.
Every recent darn tag borrowed the same trailer, mapped to the same commit, and clashed.

## What changes

- `internal/reportengine`: the walk in `taskCommitBelow` stops at the commit that opened the tag's own pair, without reading its trailer or anything below it.
  A tag with no trailer in its own history is not exact-attempted and goes to the time join, which already maps a darn run through its landing checkpoint and the board's done mark.
  The package doc states the new floor.
- `internal/fabricengine`: new exports `OriginRecordSubject(slug)` and its inverse `OriginRecordSlug(subject)` name the pair-opening commit's subject.
  `Add` commits the origin record under `OriginRecordSubject`, so the subject text is unchanged.

## Design decisions

- The report reads the subject through a `fabricengine` export instead of restating the text, as it already does for the archive tag and the trailer.
- The bound only shortens the walk.
  A tag whose own history holds a trailer reads the same task commit as before, and a history with no pair-opening commit, from a pair made before that commit existed, walks as before.
- The floor matches the tag's own slug only; another slug's opening commit in the history does not stop the walk.
- Out of scope: making darn runs write a trailer, and changing how a loom run's trailer is found above its pair-opening commit.

## Tests

- Rows in the existing landings table test pin: two darn runs over a parent history whose sync commit names an earlier landing, each mapping to its own landing with no clash; the exact join still mapping a trailer above the opening commit, with another slug's opening commit in the history; and the floor on a page after the first.
- `TestAdd_OriginRecord` asserts the subject `Add` commits equals `OriginRecordSubject(slug)` and the literal `fabric: record parent branch for <slug>`.

## To check by hand

- A darn run whose time window holds no unclaimed parent commit stays unmapped.
  That is the time join's existing behaviour, not a wrong mapping.
- Run `lyx report convergence --since 2026-10-20` on the loomyard hub and confirm the nine darn tags named in #831 each map to their own landing.
