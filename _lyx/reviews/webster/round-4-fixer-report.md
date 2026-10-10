# Webster round 4 — fixer report

## Fixed

- **F1** (`a0251d2d6`) — `PATTERN.md` is 16382 bytes, under the 16384 cap.
  The `PATTERN-hub-store-gc` line keeps the in-flight skip as ``sets them or runs `gc`, skipped while another pair's session is live; it is no destructive primitive``, and `PATTERN-documentation-lifecycle`'s link text is shortened to `[overview]`, with the same target.
  No constraint is dropped.
  `lyx gate test ./internal/pattern` passes, including `TestOverview_RealFilePassesCheck`.
  The headroom is 2 bytes, so the next entry added on `main` must save bytes too.
- **F3** (`6472e3638`) — `TestBolt_PullThenCommitWritten` gains the subtest `a board with no upstream writes and commits`, with rows for both `pullFirst` branches: the fetch succeeds, and the fetch fails.
  Each row unsets the board's upstream and asserts one landed commit at HEAD, no skip, and the record `[file_written, commit_created]`.
- **F4** (`7b272e304`) — `Bolt.PullThenCommitWritten` records `file_written` only for a returned path that exists after the write.
  A deleted path is still staged as a deletion.
  The fix is in fabricengine, so it covers both callers (`fabric reconcile` and `hubreconcile`), and also a `Retired` module's deleted file.
  The `BoltWrite` and `PullThenCommitWritten` doc comments now state this.
  The regression subtest `a deleted path is committed but not recorded as written` asserts that HEAD adds one file and deletes the other, and that only the surviving path is recorded as `file_written`.
- **F5** (`01e6d3632`) — The `stencilcommit.go` header now names what the file declares, with `CommitSeededStencils` as the commit route for `lyx stencil sync`.
  The retired "does not build on Bolt" rationale is gone.
  `StencilsSubtreeRel` and `SpecsSubtreeRel` are described on their own terms, as the board-relative counterparts of `StencilsDir`/`SpecsDir`.
  `BoardWriteLockFile` names its holders as a group: boardengine's critical section, and this package's pathspec-scoped board commits and seed-commit drops.
  `Bolt.Commit`, which stages everything, does not take this lock, so the comment does not say "every board-writing verb".
- **F6** (`09456e879`) — `underSubtree` has a doc comment.

- **Gate findings** (`eb656dd97`) — The round's comment gate flagged eleven comment lines that the lint reads as fixed-column wrapped: a sentence broken after a comma before `and`/`or`/`so`.
  They are in `boardengine/layer.go`, `seatengine/shuttle.go`, `shedadapters/multillm.go`, `shedrecipe/entries_multillm.go`, `shuttleengine/doc.go`, `stencil/stencil.go`, `tools/tokencount/cost.go` and `tools/tokencount/main.go`, all of which came in with the `main` merge-in.
  Each broken sentence is now joined onto one line.
  Linting from the branch's merge base found five more of the same breaks in the branch's own commits, in `cmd/lyx/stencilseed.go`, `fabricengine/bolt.go`, `fabricengine/destroy.go`, `gitrepo/reset.go` and `tools/deploy/main.go`, and they are joined in the same commit.
  The change touches comments only.

`lyx loom lint-comments` is clean over the working tree and with `--base` set to the merge-in, its first parent and the merge base with `main`.
The unit tier for `fabricengine`, `fabriccli`, `hubreconcile`, `pattern` and `stencilcli` passes, and the `--tags integration` tier for `fabricengine`, `fabriccli`, `hubreconcile` and `stencilcli` passes.

## Disputed

- **F2** — The premise is that a board with no upstream can never push, so its stamp stays absent forever.
  That premise is false.
  - Every push goes through `git -c push.autoSetupRemote=true push` (`internal/gitrepo/push.go:72`).
    A board branch with no upstream but an `origin` remote is therefore pushed on the first try, which also sets the upstream.
    Later walks then see it in sync, and `PushCoalesced`'s `HasUnpushed` returns no work.
  - Every hub's board has an `origin`: `lyx fabric clone` requires `<weft-url>` (`internal/fabriccli/clone.go:139`–`163`), and the board is cloned from it.
  - `internal/fabriccli/fabric.go:846`–`852` already documents the same steady state for the warp-binding push on an orphan-created board: the attempt "either succeeds or yields record_failed".
  - The only case where the push cannot succeed is a board with no remote at all, which no hub-creation route produces.
    A push that fails transiently keeps the stamp absent until the next walk pushes it, which is the behavior card 10 asks for.

## Deferred

None.
