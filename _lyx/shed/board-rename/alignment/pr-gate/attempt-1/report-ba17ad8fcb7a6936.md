---
verdict: APPROVED
---

## Scope against the board entry

The decision record has no `Added after Discussion` entries, so the measure is the board entry `board-rename` as written.

- A verb `lyx board rename` in one atomic write: `Board.Rename` runs through `boardCriticalSection` (`internal/boardengine/rename.go`), and `renameCmd` is a cobra subcommand of the board tree (`internal/boardcli/cli.go`).
- Keeps body, kind, labels, priority, issues, `created` and every other field: `renameSlug` changes only `Slug`, and `TestRename_KeepsEntryRewritesDependentsAndReportsMentions` compares the whole stored entry, a kept unknown key included.
- Rewrites the old slug in every other task's `depends_on`: `renameSlug` replaces it in place, and the test pins a run-held dependent's rewrite.
- Refuses a run-held entry as the run lock does: the refusal is a `*RunLockedError` matching `ErrRunLocked`.
- Refuses a target slug that exists or breaks the slug length limit: both pinned in `TestRename_Refusals`, with the empty target as well.
- Reports the entries whose body or brief still mention the old slug: the `mentions` list in the result and the CLI envelope.

The payload also takes `id`, which the entry does not name; it is the addressing pair every other addressing verb takes and adds no behaviour the entry rules out.
The docs edits (`boardcli` and `boardengine` `doc.go`, the `ly:board` skill) are the same-commit docs the CLAUDE.md docs rule asks for.
No part of the diff belongs to another board entry.

## Open risks

The one carried open risk, run records, worktrees and batten seeds keyed by the old slug staying unrenamed, is the entry's own out-of-scope line and names no parent correction.

## Overlaps

No parent commits are listed.

- `intake-own-repo` (row 0): changes `internal/boardcli/cli.go`, `internal/boardcli/doc.go` and `plugins/ly/skills/board/SKILL.md` at the intake verbs and intake docs; its decision record makes no rename decision.
  Text-only overlap, no design conflict. Landing order is unknown, since the peer's row is 0.
- `origin-reconcile` (row 14): changes `internal/boardengine/doc.go` and adds a board replay entry point beside `boardCriticalSection`, never changing it.
  Text-only overlap, no design conflict. This run, at row 21 (`own_row`), is further along and lands first; `origin-reconcile` takes these changes at Merge-In.
- `test-tiers`, `tuned-defaults-template` and `reed-small-fixes` change no path this run changes.

## Superseded

This run makes no other board entry obsolete.
