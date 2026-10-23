---
round: 1
focus:
  - "internal/fabricengine/bolt.go `stageablePaths`: does a path missing on disk and in HEAD's tree get left out in both `CommitWritten` and `PullThenCommitWritten`, while a missing path HEAD holds still reaches `git add` so its deletion commits? Check the unborn-HEAD case and how a `FileAtRevision` error is told apart from not-found."
  - "Both Bolt commit routes: when nothing stageable is left, does the write commit nothing without an error, and does the pull-first route still pull and push as before? Check the order of the filter against the pull."
  - "Scope against the board entry: the entry's expected result is that a stale manifest never fails the owner write. Does the diff leave `renderOwnerWrite` in `internal/boardengine/ownerset.go` unchanged, and does any other caller of the two methods change behavior it did not ask for?"
  - "Docs: the two methods' doc comments and the pull-first paragraph of `internal/fabricengine/doc.go` against the code; check each claim in `_lyx/landing/summary.md`, including the claim that a path staged in the index but absent from disk and HEAD is left out."
  - "Tests: the deleted-path subtests of `TestBolt_CommitWritten` and `TestBolt_PullThenCommitWritten` against the new behavior; name any added behavior left uncovered, such as the unborn-HEAD and nothing-stageable cases, and any new test that duplicates an existing one."
---

Seed focus from the board entry, the change description and the review range `main..HEAD`.
