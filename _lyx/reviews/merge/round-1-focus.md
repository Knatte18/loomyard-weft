---
round: 1
focus:
  - "internal/boardengine: the parent changed `Render`, `RenderToDisk`, `renderOwnerWrite` and added `Board.PrimeOwner`, a `primeOwner` argument. Check every call the task's cancel path makes into boardengine render, owner-write or status-clear code against the new signatures and the prime-owner semantics."
  - "internal/boardengine/runlock.go: the task rewrote `RunLockedError.Error` to point at `lyx batten cancel`. Does the parent's boardengine change (ownerset, replay, render) alter what the run lock or status clear the cancel verb relies on, and does `runlock_test.go` still cover the message against the parent's new code?"
  - "internal/reedengine: parent and task both edited `parse.go` (`paneTailSeparator`), `overlay.go`, `ownwindow.go` and their tests. Check the merged result is one consistent change, not a doubled or half-applied one, and that any task code reading or ending panes through reedengine uses the merged separator and symbols."
  - "contracts/stencils/orch/orch-template-role.md, docs/overview.md and the two doc.go files: both sides edited them. Check the merged prose does not contradict itself, for example a batten cancel description beside a parent statement about status clearing or prime ownership."
  - "pattern/PATTERN-glyph-conversion-chokepoint.md: the parent added the `planglyph.EncloseAt` rule for quarry's `Enclose`. Check whether any code the task added converts a location to a glyph outside that call."
  - "Test economy: after the merge, look for a task test that now duplicates a parent test in boardengine or reedengine, and for cancel-verb behavior that touches the parent's new prime-owner render with no coverage."
---

Seed focus from the overlap list and the two ranges' diffs.
The reedengine hunks look identical on both sides, so the question there is the merged tree's consistency, not either side's change.
