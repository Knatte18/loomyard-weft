# Round 1 fixer report — darn

Every finding was fixed, one commit each, with its package tests green through `lyx gate test`.

## Changes

- **F1 (LOW, consistency) — fixed differently.** `ee089211e`
  `contracts/stencils/bouncer/bouncer-template-escalation.md` step 1 no longer names a recipe.
  It now reads "Record each design call … with `lyx loom decision add` …, when the decision record exists. Without one, name each design call in your report back to your parent session instead."
  That condition matches the Materials list's existing item 4, "The decision record at …, when it exists", which loom's brief already carried, so loom's parent fork reads no sentence about another recipe.
  `contracts/stencils/claims_test.go` pins the new wording and adds a `wantNone` claim that the shared brief never says `darn`.
  Proposed change not made: a darn-only marker filled by the darn wiring.
  The stencil's own header says "Go interpolates only paths, round, cause, segment and slug; the wording lives here and nowhere else", and it allows no `{{if}}` conditionals, so a Go-filled wording marker would break that contract.
  The summary's claim, that the brief records design calls only when the record exists and otherwise has the fork report them, still holds.
- **F2 (LOW, scope) — fixed as proposed.** `e456f8a25`
  `TestArmAt_RenamesRetiredDarnRows` in `internal/loomcli/wiring_darn_test.go` gains a "darn with a live driver" row.
  The row holds the run lock, then asserts that `armAt` returns no error and that the status file still names `Darn`.
- **F3 (NIT, consistency) — fixed as proposed.** `829acbb31`
  `wireDarn` in `internal/loomcli/darnwiring.go` now returns an error when `BouncerRunSubdir` reports that `Darn-Bouncer` is not a Bouncer with a `run_subdir`.
  The error names the embedded recipe as broken.
  It does not fall back to the reviews root.

## Disputed

None.

## Deferred

None.
