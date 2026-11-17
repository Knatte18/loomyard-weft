# Sweep read-only leftovers a killed test leaves under the scratch root

A test that makes a directory read-only and restores it only in `t.Cleanup` leaves it read-only when its binary is killed.
The `lym temp` sweep could not unlink such a leftover, so the next run's scratch-segment cleanup test failed on machine state rather than on its change.
This change makes the sweep remove those leftovers and moves the read-only test pattern into one testkit helper.

## What changes

- `internal/lymtemp`: the dead-owner sweep now removes a run directory through `removeTreeMakingWritable`.
  It first gives every directory in the tree owner read, write and search permission, then calls `os.RemoveAll`.
  Files and links keep their bits, and a link is never followed.
  The permission pass is best-effort: its failures are ignored, and the removal's own result is what the sweep reports.
- New `internal/testkit/permkit` with `Chmod(tb, path, mode)`.
  It sets the mode, registers a cleanup that restores the original bits, and drops the restore's own error so a test may remove the path first.
  The kit imports the standard library only; `TestEnforcement_PermkitStdlibOnly` in `internal/testkit` pins this.
- Tests that called `os.Chmod` with a read-only or unreadable mode and a hand-written restore now call `permkit.Chmod`.
  They span the packages the `grep -rn 'Chmod(.*0o555'` in the board entry finds, plus further ones of the same shape.
- `PATTERN.md` and `pattern/PATTERN-test-isolation.md` name `permkit.Chmod` and the sweep's permission pass.

## Design decisions

- The permission pass sets a directory's bits before it lists the directory, so an unreadable directory is opened too.
- Only directories are touched: removing a file or link needs only its parent's write permission.
- The pass does not follow links, so a link inside a run directory cannot change the mode of a directory outside it.
  `TestSweep` covers this with a link to a read-only directory outside the tree.
- A failed pass falls through to `os.RemoveAll`, so a path that cannot be changed reports the same error as before.

## To check by hand

- `lymtemp` link safety was the open review finding: the sweep's link handling had no test.
  Check that the `lockedTree` row of `TestSweep` and the "a failed permission pass leaves the result to the removal" case in `internal/lymtemp/sweep_test.go` cover it.
- Nothing ran by hand: no regression check ran for this task.
  To reproduce the original failure, kill a test binary mid-run while it holds a directory made read-only without `permkit.Chmod`, then run `lym cleanup`.
