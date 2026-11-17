# Fixer report — round 1

## Changes

- F1 (MEDIUM): fixed as proposed. The `lockedTree` row in `internal/lymtemp/sweep_test.go` now also holds a link, made with `fslink.CreateDirLink`, to a 0o555 directory outside the root, and asserts after the apply sweep that the directory exists with its mode unchanged. Committed with F2 as `78a368bc`, since both touch `sweep_test.go`.
- F2 (LOW): fixed as proposed in the code. `makeTreeOwnerWritable` is now best-effort and returns nothing: it skips a directory it cannot chmod or list and keeps walking. `removeTreeMakingWritable` ignores the pass and returns `os.RemoveAll`'s result. The doc comments and `doc.go` say so. The row to add is fixed differently, see below. Commit `78a368bc` plus the follow-up commit named in Self-check.
  - Not made as proposed: a row where the pass fails while `RemoveAll` succeeds, or fails through the default remove with the chmod pass failing. An owner's chmod and `ReadDir` do not fail once the bits are set, so staging that needs a seam production has no use for. Instead, the row `a failed permission pass leaves the result to the removal` uses a path below a regular file, where the pass's `Lstat` fails with ENOTDIR, and asserts the result equals `os.RemoveAll`'s, compared by error text so the old early return (an `lstat` error) fails it.
- F3 (LOW): fixed as proposed. `TestEnforcement_PermkitStdlibOnly` in `internal/testkit/enforcement_test.go`. Commit `a35838bd`.
- F4 (LOW): fixed as proposed. The `//testtiming:keep` line sits above `TestSweep_DeleteFailureTolerance` in `internal/logger/retention_test.go`. Commit `c1b171e2`.

The gate `lym gate test ./internal/lymtemp ./internal/testkit ./internal/logger` passed before the self-check.

## Self-check

- fix-code-correctness, MEDIUM: the F2 row compared only nil-ness, so it passed with the defect in place. Fixed: it now compares error text.
- fix-code-correctness, LOW: the F1 mode comparison is relative and weak on Windows, but it does not break and fails on a link-following regression. Disputed, `every fix worse`: a fixed mode would break on Windows.
- fix-proposal, MEDIUM: same as the code-correctness MEDIUM; merged, fixed the same way.
- fix-proposal, LOW: no row exercises the walk continuing past a failing subdirectory. Deferred: staging a failing subdirectory needs a seam in the walk (see F2 above).
- fix-consistency: no findings.
- fix-affected-set: no findings.

## Disputed

- fix-code-correctness LOW (F1 Windows weakness), ground `every fix worse`: the evidence is that `permkit.Chmod` may not change `Perm()` on Windows, so a fixed 0o555 expectation would fail there.

## Deferred

- fix-proposal LOW, walk-continues row: needs a seam inside `makeTreeOwnerWritable`, which production has no other use for; an owner cannot make a chmod or `ReadDir` fail deterministically.

## Rulings

None.
