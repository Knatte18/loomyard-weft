# Decision record — done-check-spaced-create-paths

## Goal

Webster's post-batch done-check judges every Create, Delete and Rename target that `lyx webster validate` accepts, and a `create-not-done` refusal tells a missing declaration apart from a lookup that misses one that exists.

## Scope

- `internal/planglyph`'s `DoneChecks` judges a target no glyph can spell, such as a Create path holding a space, as a file path on disk instead of handing it to quarry.
- A Create or Rename-new target quarry answers `not_found` names, in its finding, the files that declare the looked-up unit and every declaration elsewhere in the worktree carrying the looked-up name.
- A batch failed on a `glyph-rejected` done-check finding names the plan edit as its way forward, never `lyx webster recover-batch` alone, which would only repeat the rejection.
- A test pins that quarry resolves a `plan:` handle whose unit is a Go external test package (`<dir>_test`).
- Out of scope: making `lyx webster validate` refuse a spaced Create path (operator decision of 2026-10-22), and the `loop-exited` halt reason (task `loop-exit-halt-names-its-cause`).

## Decisions

### A done-check target with no `#` is judged on disk

Decision: in `DoneChecks`, a Create, Delete or Rename-side key that is neither a glyph nor a `plan:` handle (no `#` after `resolveKeyFor`) never enters the batched resolve.
A Create or Rename-new key passes when a file exists at that worktree-relative path; a Delete or Rename-old key passes when nothing exists there.
The findings keep their check IDs, `create-not-done`, `delete-not-done` and `rename-not-done`, with a detail that names the disk path checked.

Rationale: canonicalization turns every path it can into a self glyph, so a key still without `#` is a path `glyph.Self` refused, such as one whose unit holds a space.
Quarry can never resolve it, and its existence on disk is exactly what a self glyph's resolve would have answered.
Bound: only keys without `#` take this route, and validate's `directory-target` already refuses a slashed extensionless path, so a directory never reaches it as a Create.
A stat error other than not-exist returns an error, never a pass.

### A `not_found` lookup names what quarry searched and where the name lives

Decision: when any Create or Rename-new key answers `not_found`, `DoneChecks` lists the worktree's declarations once through quarry's glyphs query.
The finding's detail then names the files holding a declaration of the looked-up unit, and every declaration elsewhere whose own name equals the looked-up member's name, with its glyph and file.

Rationale: issue #823's refusals could not tell a test the card never wrote from one written under another package or file.
The listing runs only on a failing done-check, so a clean batch pays nothing.
An error from the listing is quarry infrastructure and returns wrapped in `ErrQuarryUnavailable`, as the resolve's own error does.

### A `glyph-rejected` done-check failure names the plan edit

Decision: when a batch fails on a `glyph-rejected` done-check finding, the way forward is to amend the card's target to a spelling quarry resolves, run `lyx webster rebaseline --card NN` naming each edited card, then `lyx webster recover-batch NN`.
A record holding uncheckable entries names the fresh restart steps instead, as the delete-referenced way forward does.

Rationale: recover-batch re-runs the same done-check on the same plan, so naming it alone sends the operator around a loop (issue #809).

## Constraints

- `PATTERN-refusal-way-forward`: the new way forward is named in the refusal and a test reaches it.
- `PATTERN-no-denied-recovery`: the named steps are verbs agents may run.
- `PATTERN-test-economy`: new cases extend `TestDoneChecks_Verdicts`, `TestDoneCheckVerdicts_Rules` and `TestRecordBatch_DoneChecksBlockOnUnresolvedCreate` rather than adding parallel tests.
- `PATTERN-comment-line-breaks`: one sentence per line in every Go comment.

## Auto-mode assumptions

- The worktree's files at done-check time are the batch's committed tree, so a disk check equals "exists in the batch's commits".
- A full-worktree glyphs listing on a failing done-check is cheap enough, since it runs only on the failure path.

## Open risks

- A large unit makes the searched-files list long; the detail lists them all, since a truncated list cannot show the file the operator is looking for.

## Acceptance criteria

- A Create target `dir/with space/Spaced.cs` passes when the file exists and reports `create-not-done` naming the path when it does not.
- A Delete of a spaced path passes when the file is gone and reports `delete-not-done` when it remains.
- A bare `LICENSE` Create target that reaches `DoneChecks` uncanonicalized passes when the file exists.
- A `plan:sub_test#TestFoo` Create target passes when `sub/a_test.go` declares `TestFoo` in package `sub_test`.
- A Create target `sub#Missing` answered `not_found` names the unit's files in its detail, and names `other#Missing` with its file when another unit declares it.
- A batch failed on a `glyph-rejected` done-check finding names the amend, rebaseline and recover-batch steps as its way forward.
