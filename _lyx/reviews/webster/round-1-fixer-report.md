# Fixer report — webster round 1

## Changes

- F1 (fixed with own change): `primeSeedError` in `internal/configsync/configsync.go` adds the "also delete the <selector> line" step only when the module has an overlay selector.
  The proposed change, returning `err` unchanged for an empty selector, was not made: both callers (`configsync.go` reconcile path and `movekeys.go:123`) rely on the result being a `*FileError` naming the prime's copy, and an unchanged error would lose that file name (the reconcile path would wrap it with the board file's path instead).
  `TestPrimeSeedError` in `configsync_test.go` pins the three cases: invalid with a selector, invalid without one, and any other failure unchanged.
  The prime-seed table could not carry the row, because `primeConfig` seeds only from a prime copy that parses, so no registered no-selector module reaches an invalid seed failure through it.
- F2 (fixed as proposed): a `defaultFile` field and one success row each in the `Set` table (`set_test.go`) and the `Edit` table (`edit_test.go`), where a hub worktree write sets the selector to the default file's value.
- F3 (fixed as proposed): one `Set` row where the default file holds an unknown selector value and a hub worktree write is refused with no file left.
- F4 (fixed as proposed): `applyRemove` in `internal/lenses/config.go` skips an unmatched name already in `skipped`.
  The two existing remove subtests now list a repeated unmatched name and a repeated matched name.

Each finding is its own commit; `lyx gate test` passed for `internal/configsync`, `internal/configengine` and `internal/lenses`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: LOW, F1 guard drops the `*FileError` at the key-move call site; fixed with own change (above).
- fix-affected-set: MEDIUM, same F1 loss at `movekeys.go:122-123` and `configsync.go:946`; merged with the fix-proposal finding and fixed with the same change.

## Disputed

None.

## Deferred

None.
