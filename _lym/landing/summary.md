# Webster Create refusals: short details, signature mismatch, real way out

A batch that fails webster's done-checks on a Create target now gets a short refusal that names the real cause and a way forward that can clear it.
This fixes issues #61, #62 and #63.

## What changed

**Shorter not_found details (`internal/planglyph/donecheck.go`).**
A Create target or Rename new side that quarry answers `not_found` used to list every file in the target's unit and every declaration of its name in the repository.
The unit list repeated for each target, and common names produced kilobytes of unrelated packages.
The detail now gives the count of the unit's files (`quarry searched 2 files of unit internal/a/b`).
It names at most `maxListedDeclarations` (three) declarations of the name, sorted by how many leading unit segments they share with the target's unit, and counts the rest (`... and 4 more`).
The one-line finding still opens the detail.

**New blocking finding `signature-mismatch`.**
Before, a C# target written with a parameter list whose member exists with another parameter list was reported as `create-not-done` with "the unit exists and the member is missing".
It is now its own finding, which opens with the declared signature and its file, for example `declared as Shop#CartTests.Adds(string) in tests/CartTests.cs, not with the parameter list Create target "Shop#CartTests.Adds(int)" names`.
It fires only when the target carries a parameter list and its unit declares the same owner and name with other parameters only.
Go glyphs never carry parameters, so Go plans never get this finding.

**Way forward for a missing Create target (`internal/websterengine/recordbatch.go`).**
`cardNotDoneFor` now flags `create-not-done` and `signature-mismatch` findings, and `failCardNotDone` picks the way forward through the new `createTargetWayForward`:

- record-batch on `create-not-done`: `lym webster recover-batch NN` to build the target as the card names it, or, when the member is declared where the finding says, amend the card's Create target, run `lym webster rebaseline --card NN`, then `recover-batch`.
- `signature-mismatch`, or `create-not-done` from recover-batch (a recovery already failed the same target): only the amendment route.
  Before, recover-batch pointed back at itself, so the run could only repeat the same refusal.
- A record that lists uncheckable entries: the reset-to-start steps, as the existing delete-not-done and glyph-rejected way forwards already do.

Each `rebaseline --card` records an amendment, which makes the next recovery spawn uncounted.
That means the amendment route works even after both counted recoveries are used up.

## Design decisions

- The signature mismatch gets its own check ID rather than a reworded `create-not-done` detail.
  That keeps it distinct for the reader and lets webster choose a different way forward for it without parsing detail text.
- The issue's alternative, a new verb to retarget a create target, is not added.
  The existing amend-and-rebaseline route already moves a target, and the refusal now names it.

## Tests

- `TestDoneCheckVerdicts_NotFoundDetail` (new) pins the signature-mismatch finding and the capped, nearest-first declaration list.
- The existing not_found detail expectations in `TestDoneCheckVerdicts_Rules` and `TestDoneChecks_Verdicts` now expect the file count.
- `TestRecordBatch_DoneChecksBlockOnUnresolvedCreate` asserts that the record-batch way forward offers both routes.
- `TestRecoverBatch_TerminalRunsTheSamePostBatchChecksAsRecordBatch` asserts that a failed recovery names only the amendment.
- `TestDoneCheckVerdicts_NotFoundDetail` covers the `signature-mismatch` finding itself, but no webster-level test drives that finding through `failCardNotDone`.
  The C# fixture such a test needs does not exist yet.
  The amendment-only text it would produce is covered by the recovery test.
