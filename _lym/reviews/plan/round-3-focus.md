---
round: 3
focus:
  - "The decision record's added entry 'This part's fabric test code shrinks in net' against `_lym/plan/00-overview.md` and cards 2, 3, 4 and 15: does the plan show, before any card runs, that deleted _test.go lines exceed added ones across internal/fabricengine, internal/fabriclayout, internal/fabriccore and internal/testkit/fabrickit, with no step that finds this out or halts on it while running? For each named test deletion, does another test hold its assertion, or does it guard no rule of its own?"
  - "`_lym/plan/15-snapshot-reader.md` and `_lym/plan/14-executor-loop.md`, read-failure contract: could two implementers differ on the order stores are read in, the Store recorded for each field's failure (FieldWorktrees, FieldAnchors, FieldRecordsStore, FieldPendingTeardowns), whether a phase whose read failed enters Snapshot.Phases, and which Phase a failed re-read records? Do TestRunRule's rows assert the result?"
  - "`_lym/plan/15-snapshot-reader.md`, TestSnapshotReader_RefStorage: can the `files` ref-storage row pass when FieldLocalRefs is read from a subdirectory of a linked worktree? readLocalRefs reads a `files` store through gitrepo, whose open may refuse a path that is not itself a repository (internal/gitrepo/gogit.go, internal/gitrepo/doc.go)."
---
