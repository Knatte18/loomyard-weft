# Support log — done-check-spaced-create-paths

## Interview

A darn run holds no interview: the board entry, its folded issues #805, #809 and #823, and the operator decisions recorded in its body are the whole input.

## Rejected alternatives

- Refusing a spaced Create path in `lyx webster validate`: the operator decided on 2026-10-22 that the done-check accepts it instead.
- Auto-addressing a bare path as `<path>#`: quarry refuses a unit holding a space (`unit_bad_rune`), so the `#` spelling does not resolve either.
- Checking existence through `git ls-tree` on the batch's head: the worktree is the committed tree at done-check time, and planglyph runs no git.
- Restricting the not-found listing to the unit's directory: a test written in another package, the case #823 reports, would stay invisible.
- Truncating the searched-files list: the file the operator needs may be the one cut.

## Review rounds

_No rounds yet._

## Question ledger

- Does quarry resolve `plan:` targets in Go external test packages? Self-pick: it does by its unit rules; a test row pins it, and the not-found detail covers the cases that still miss.
- Should Delete and Rename sides get the disk route too? Self-pick: yes, since a spaced path hits the same quarry rejection in those groups.
- Does item (2) of #809, recover-batch naming itself, belong here? Self-pick: yes, for a `glyph-rejected` done-check failure, the one the issue saw.
