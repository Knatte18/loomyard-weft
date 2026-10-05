# Fixer report — plan review round 1

Every finding was fixed in the plan directory; `approved:` in `00-overview.md` was not touched.

## Changed

- F1 (card order): swapped cards 4 and 5, so the readers card is now `04-board-readers-hub-file.md` and the retire card is `05-retire-worktree-copy.md`.
  The headings, `**Commit:**` numbers and Card Index entries follow, and the retire entry states why it runs after the readers card.
- F2 (`03-hub-wide-reconcile.md`): the prime seed applies only to modules that declare `OpenMaps`.
  `fabric` keeps its legacy fold-in, reported as `Seed == "legacy"`, and the template warning is raised only for modules with open maps.
  The table-driven test gains the two `fabric` rows.
- F3 (`07-config-cli-hub-wide.md`): the Intent states the atomic restore, the joined restore error naming the file, and the two residual kill windows with their bound.
  One of them exists because `configengine.Set` uses a plain `os.WriteFile`; the card says so rather than claiming atomicity.
- F4 (`07-config-cli-hub-wide.md`): added the concurrent-edit message, the exception to staging-file removal, and one row in the table-driven `dispatch` test.
- F5: card 5 (retire) now updates `ReconcileAll`'s doc and the `Retired`/`Divergent` field comments; card 3 names `ReconcileAll`'s doc, the `runReconcile` comments and the `configsync.go` header correctly.
- F6: card 4 (readers) lists `plugins/ly/skills/board/SKILL.md` as an edit target and rewrites its board.yaml sentence; card 6 changes the two generic `configengine.md` examples.

## Deferred

Nothing.
