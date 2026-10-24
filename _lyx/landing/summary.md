# webster record-batch: accept Create targets in the external test package

`lyx webster record-batch` no longer fails a batch with `create-not-done` when a Go Create target was written into the other package of its directory.
A plan naming `plan:<dir>#TestX` while the card wrote `TestX` into `<dir>_test` (or the reverse) now records clean.
Before, quarry answered `not_found` for the named unit, and the batch was refused even though the member existed.
The plan cannot know at plan time which package a new test file declares, so the check resolves it at record time, when the declaration is on disk.

Closes #851

## Changes

- `internal/planparser`: new exported `GoPackageCounterpart`, which maps a Go member glyph to the same member in the other package of its directory.
  It sits beside `goPackageDir` and is built on `ParseRef` and `Glyph.String`, so the unit adjustment stays inside `planparser` as `PATTERN-glyph-conversion-chokepoint` requires.
  It returns false under a non-go plan, for a non-member ref and for a file unit.
- `internal/planglyph` `DoneChecks`: a Create target or Rename new side answered `not_found` counts as done when its counterpart glyph appears in the worktree declarations the check already lists for a missing member.
  No second resolve is issued.
- `internal/planglyph` `BindHandles`: a handle whose expected glyph matches no delta symbol binds to the counterpart glyph when the delta holds it.
  The plan's later refs are rewritten to the glyph the card actually created, so drift detection and later batches resolve it.
- The chokepoint pattern, the loom plan spec and the `planparser` and `planglyph` package docs name the second sanctioned unit adjustment.

## Design decisions

- The counterpart check works in both directions, since a plan may equally name `<dir>_test` for a test written in-package.
- Rename new sides share the `not_found` path, so they get the same treatment as Create targets.
- A delta symbol binds at most one handle of a card.
  A second handle that reaches the same glyph through the other package's spelling counts as unmatched, so it still surfaces as `bind-count-mismatch` rather than silently binding twice.
- A target passes only when the same member name is declared in the same directory's other package.
  A member declared in an unrelated unit still raises `create-not-done`.
- C# plans are unchanged: no counterpart lookup happens.
- A plan-time validator that rejects or rewrites such targets is out of scope.

## Known limitation

A real package directory whose own name ends in `_test` maps to its sibling directory's package, as `goPackageDir` already does.
The counterpart then only passes when that sibling declares the member.

## Tests

New rows in the existing done-check, handle-binding and `targetdirs` tests cover both directions, the unrelated-unit refusal, the Rename new side, a C# plan, and the one-handle-per-delta-symbol rule.
Nothing needs to be run by hand beyond the project gate.
