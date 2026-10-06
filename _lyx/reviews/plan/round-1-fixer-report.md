# Round 1 fixer report

Every finding in `round-1-review.md` was fixed.
`approved:` in `00-overview.md` stays `false`.
`lyx loom validate-plan` exits clean after the edits (informational `create-new-unit` notes only).

## Changed

- **F1 (BLOCKING)**: `00-overview.md` gains the Shared Decision `skill-loading-tests-untouched`, applied to cards 7, 8, 10, 12, 13 and 14, and `prune-procedure` step 3 now says a protected candidate is kept, never deleted or folded.
  Card 8 names the protected tests (`startup_test.go`, `session_test.go`, `skillFakeEngine` in `fakes_test.go`) and leaves `shuttlefake`'s `SkillLoader` methods as they are.
  Cards 13 and 14 carve the stub-driver skill-loading tests out of what they may fold or merge.
- **F2**: card 8 now targets `internal/testkit/shuttlefake#`, and card 17 targets the six package units.
  `untag-procedure` says the shared kit directory does not exist before the card runs, so the card's commit message names the path, which covers cards 9, 11, 13 and 15.
- **F3**: `untag-procedure` states the `PATTERN-hubforge-fixtures` rule (a real hub is built through `internal/hubforge`, a moved test drops the hub), and card 15's fixture sentence matches it.
- **F4**: card 2 now excludes `os.Args[0]` only, and its `TestScanSpawns` rows cover `os.Args[0]` and an `os.Args[1:]` read that stays judged.
- **F5**: card 3 drops the conditional `TestClassify` wording and plans one `TestRenderPackage` row in which a kept test appears in another candidate's Covering column.

## Deferred

Nothing deferred.
