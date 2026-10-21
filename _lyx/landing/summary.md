# Add lyx board rename to change an entry slug in one write

`lyx board rename '{"slug":"old","to":"new"}'` renames a board entry in one atomic write.
Until now a rename took a `get --body` to a file, a `merge` with `remove_slugs`, an upsert that re-carried the old entry's fields, and a hand fix of dependents.
That route dropped fields and left other entries' `depends_on` pointing at the old slug.

The entry keeps its id, position, body, brief, kind, labels, priority, issues, dates, run settings, profile, status and unknown keys.
Every other entry's `depends_on` has the old slug replaced by the new one in place, keeping its order.
The write saves, renders and syncs through `boardCriticalSection` like every other board write, so the README and design docs render under the new slug and the old slug's design file is dropped.

The payload is `{slug|id, to}`: the addressing pair the other addressing verbs take, plus a required `to`.
The success envelope is `{"ok":true,"task":{…},"mentions":[…]}`.

Closes #641

## Design decisions

- **A rename is no revision.**
  The critical section judges dates and the run lock against the loaded entries with the one old-to-new slug pair applied.
  Neither the renamed entry nor a rewritten dependent gets a new `revised` date, and a run-held dependent is not refused for the `depends_on` rewrite alone.
  Any other change in the same write is still judged in full.
- **Refusals write nothing and name the way forward.**
  The verb refuses an absent entry, a target that is empty, longer than `boardengine.MaxSlugLength`, or already an entry's slug (the source itself included), and an entry a run holds.
  The run-held refusal is the existing `*RunLockedError`, so it matches `ErrRunLocked`.
- **Prose is reported, not rewritten.**
  `mentions` lists, in store order, the slugs of entries whose body or brief still name the old slug, the renamed entry included, and is `[]` when none does.
  An occurrence counts only as a whole slug: no letter, digit, `-` or `_` on either side, so `board` is not found inside `board-rename`.
- **Addressing by `id`** is an addition to the board entry, which named only `slug`.
- **Not renamed:** worktrees, branches and run records keyed by the old slug.
  The run-held refusal protects a live run, but a finished or abandoned run's records keep the old slug.

## Changes

- `internal/boardengine`: `Store.Rename`, the `Board.Rename` facade under the write lock, and `RenameResult`, in `rename.go`.
- `internal/boardcli`: the `rename` verb with its help, which states the slug limit, and its payload declaration.
- Docs: the `doc.go` of both packages and the `ly:board` skill.
- Tests extend the existing tables where one covers the behavior: engine tests for field retention, dependents, refusals and mention edges, and CLI tests for the verb, its help and its payload keys.

## To check by hand

- `go test` was not run by this description's author; run `lyx gate test --tags integration ./...` before pushing.
- Try a rename against a real board that has a dependent and a prose mention, and confirm `mentions` and the rendered files.
