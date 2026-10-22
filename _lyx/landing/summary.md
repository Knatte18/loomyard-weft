# webster done-check: judge spaced paths on disk, explain not_found

Webster's post-batch done-check no longer refuses a plan target that `lyx webster validate` accepts.
A Create path whose directory holds a space, such as `internal/engine/testdata/csharp/tree/with space/Spaced.cs`, used to be handed to quarry, which rejected it as a glyph (`no_separator`).
`record-batch` and `recover-batch` then both refused with `glyph-rejected`, and `recover-batch` named itself as the way forward.
The `create-not-done` refusal also could not tell a test the card never wrote from a lookup that missed one that exists.

Closes #805
Closes #809
Closes #823

## What changes

- `internal/planglyph` `DoneChecks` judges a target that is still path-shaped after canonicalization on disk, never through quarry.
  A Create or Rename-new path passes when the file exists at that worktree-relative path; a Delete or Rename-old path passes when nothing exists there.
  The findings keep `create-not-done`, `delete-not-done` and `rename-not-done`, with a detail naming the disk path.
  A stat error other than not-exist returns an error, never a pass.
  `planparser.IsDiskCheckedTarget` holds the keep/skip policy.
- A Create or Rename-new target quarry answers `not_found` now names, in its finding, the files declaring the looked-up unit and every declaration elsewhere in the worktree with the looked-up name, each with its glyph and file.
  The detail lists them all, since a truncated list could hide the file the operator needs.
- A batch failed on a `glyph-rejected` finding names the plan edit as its way forward: amend the card's target, run `lyx webster rebaseline --card NN`, then `lyx webster recover-batch NN`.
  A record recovery would refuse as uncheckable names the fresh restart steps instead.
- Tests pin that quarry resolves a `plan:` handle into a Go external test package (`<dir>_test`), as asked in #823.
- `contracts/specs/loom-plan-spec.md`, and the `planglyph` and `websterengine` `doc.go` files, describe the new behavior.

## Design decisions

- Per the operator decision, `lyx webster validate` is not made to refuse a spaced Create path; the done-check is made to agree with it.
- Only keys with no `#` after canonicalization take the disk route.
  The worktree at done-check time is the batch's committed tree, so a disk check equals "exists in the batch's commits".
  Validate's `directory-target` already refuses a slashed extensionless path, so a directory never reaches this route as a Create.
- The glyphs listing behind the `not_found` detail runs once, and only when such a finding is due, so a clean batch pays nothing.
  An error from the listing returns wrapped in `ErrQuarryUnavailable`, as the resolve's own error does.

## Out of scope

- The `loop-exited` halt reason from #809 belongs to task `loop-exit-halt-names-its-cause`.

## To check by hand

Not run against a real hub: a `record-batch` on a batch whose plan creates a spaced path (the `csharp-alphabet` card 9 case) and a `not_found` refusal's new detail on a real quarry hub.
