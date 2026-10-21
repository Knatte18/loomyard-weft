# lyx board rename: change an entry slug in one write

`lyx board` had no verb to change an entry's slug.
The manual route, a `merge` with `remove_slugs` plus an upsert carrying the old entry's fields, lost the id, position and kept keys, stamped new dates, and left every dependent's `depends_on` to be fixed by hand.

`lyx board rename '{"slug":"old","to":"new"}'` (or `{"id":…,"to":…}`) now does it in one locked write:

- The entry keeps its id, its position in `board.json` and every other field: body, kind, labels, priority, issues, dates, status, run settings, profile and unknown keys.
- Every other entry's `depends_on` names the new slug in place of the old one, in place.
- The envelope is `{"ok":true,"task":{…},"mentions":[…]}`: `mentions` lists the entries whose body or brief still name the old slug, the renamed entry included, and is `[]` when none does.
  Prose is never rewritten.
- It refuses, writing nothing, an absent entry, a run-held entry (the run lock's `*RunLockedError`, matching `ErrRunLocked`), and a target that is empty, over `MaxSlugLength` or already an entry's slug.
- The render after the write drops the old slug's design doc and writes the new one.

## Changes

- `internal/boardengine/rename.go`: `Board.Rename`, `Store.Rename` and `RenameResult`, plus the whole-slug mention match.
- `internal/boardengine/board.go`: `boardCriticalSection` recognises a `RenameResult` and applies the rename to the loaded entries before stamping dates and checking run locks.
- `internal/boardcli/cli.go`: the `rename {slug|id, to}` verb and its `renamePayload` declaration.
- Docs: the `boardcli` and `boardengine` package docs and the `ly:board` skill.
- Tests: `rename_test.go` pins what a rename keeps, rewrites, renders and reports and every refusal; the CLI `TestCLI` gains a rename step, and the help and payload-token tables gain a rename row.

## Design decisions

- **A rename is no revision and no scope change.**
  The critical section judges dates and the run lock against the loaded entries with the one old-to-new pair applied.
  So neither the renamed entry nor a rewritten dependent gets a new `revised` date, and a run-held task that depends on the renamed entry takes the rewrite instead of refusing the whole rename.
  The translation covers only the pair the rename names, and only in a rename write.
  The renamed entry itself is refused when run-held, before any mutation.
- **A mention is a whole-slug occurrence**: the old slug with no letter, digit, `-` or `_` beside it.
  A plain substring match would report `board` inside every `board-…` slug.
- **Slug or id addressing**, like every other addressing verb, though the board entry named only `slug`.
- **Renaming an entry onto its own slug is refused** as an existing target rather than treated as a no-op.

## Not covered

Run records, worktrees and batten seeds keyed by the old slug are not renamed.
The run-held refusal keeps a live run's entry from being renamed, but a finished or abandoned run's records keep the old slug.
