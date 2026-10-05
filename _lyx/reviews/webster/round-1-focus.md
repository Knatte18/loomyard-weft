---
round: 1
focus:
  - "Shared Decision 1: grep the diff for any site that compares a module name to `fabric` or `board` to decide hub-wide handling instead of branching on `configreg.Module.HubWide`; settle whether each remaining name comparison is unrelated to hub-wide handling."
  - "`internal/fabricengine/bolt.go` `Bolt.CommitWritten`: settle whether the write and the commit both happen under the board write lock, whether the commit is scoped to the written paths only, and whether a failed commit leaves the written file in a state the caller can recover from."
  - "`internal/configcli/hubwide.go` staged edit: settle whether only the copy into `_board`, its validation and its commit run under the board write lock, and that the lock is never held while the editor is open."
  - "`internal/configsync/configsync.go` `ReconcileHubWideAt` and `ReconcileAll`: settle whether seeding `board.yaml` from the prime reads `<hub>/<PrimeName>/<AnchorRel>` through the warp junction, whether an existing hub file is never overwritten, and whether retiring a per-worktree copy only removes copies the hub file subsumes."
  - "Hub, board dir and prime resolution: settle whether `configsync` and `boardengine` receive only absolute paths and resolve nothing themselves, per Shared Decision 3."
  - "Card 4 readers: settle whether every board-config reader (`lyx board`, loom's landing board, `lyx ide` menu) loads the hub `board.yaml`, and whether a missing hub file refuses with a message naming `lyx fabric reconcile`."
  - "Card 7: settle whether `PATTERN-hub-wide-config` exists in `PATTERN.md` or `pattern/` in the same change and states the invariant the code implements; settle whether the `doc.go` files and `docs/` the diff touches match the new behavior."
  - "Per-card mechanical check: for each card's groups, settle whether the type-specific mechanical check (rename AST script plus grep, `assert-no-callers` for deletes) ran against every target of the group, not only the first."
  - "Test economy: settle whether the large rewrites in `configsync_test.go`, `configcli_test.go` and `boardcli/cli_test.go` add tests that duplicate an existing test's coverage, naming the covering test, and whether each new behavior (`CommitWritten`, `EditPath`/`StagingFile`, retirement of subsumed copies, prime seeding) has coverage."
  - "Comment conventions: settle whether new and changed doc comments follow the target repository's own `CLAUDE.md` and `PATTERN.md` rules, including one sentence per line in markdown."
---

Seed focus drawn from the plan's Card Index and Shared Decisions and the diff stat against the merge base.
No round has been reviewed yet.
