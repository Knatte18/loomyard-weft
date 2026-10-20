# Plan round 2 — fixer report

## Changes

- **F1 (MEDIUM), fixed as proposed.**
  `08-darn-landing-tree-scope.md`: after the `TestDarnTreeScope_ClassifiesAtEachCall` sentence, a new sentence covers Card 7's step where the hub `darn.yaml` holds `./tools` under `tree_scope`.
  At that step, `VerifyCommand()` returns an error naming `./tools`, `lyx config darn`'s editor and "then re-run the lyx command that refused", and `PublishVerifySkip()` returns the same error.
- **F2 (MEDIUM), fixed as proposed.**
  - `07-darn-gate-impact-scope.md`: the `darnTreeScope(l)` paragraph now has the seam call `darnengine.VerifyCommand` over `fabricengine.ConfigDirs(l)` first and return its error unchanged.
    It then loads the tree keys through `darnengine.LoadConfig`.
    Only the merge-base read, the diff read and the second load, which only a config edit between the two loads reaches, return their errors as is.
    The `WithEditorWayForward` clause is dropped from this paragraph.
  - Card 7 also gains a test row: with a hub `darn.yaml` that does not unmarshal, the seam error names `lyx config darn --set verify=` and "then \"lyx loom resume\"".
  - `02-darn-tree-scope-config.md`: `darnTreeScope` is removed from the list of `WithEditorWayForward` callers, and the sentence now reads "calls it, directly or through `VerifyCommand`".
- **F3 (LOW), fixed as proposed.**
  `08-darn-landing-tree-scope.md`: "read the way the merge-commit read fails" becomes "the way `holdParent` returns a parent-guard read error".
- **F4 (LOW), fixed as proposed.**
  `00-overview.md`: `./internal/websterengine` is appended to the end of the verify block's `lyx gate test --tags integration` line.
  `lyx loom validate-plan` reports `ok` on the changed plan.

## Self-check

- fix-consistency, LOW: Card 2's "every reader ... calls it" no longer held once the seam's second `LoadConfig` returns its error as is.
  Fixed with my own change: Card 2's sentence now exempts a reader's second load of the same files in one call, which only an edit between the two loads makes fail.
- fix-claims, LOW: Card 7 said only "an edit of `darn.yaml`" makes the second load fail, but `LoadConfig` also reads other layers and the fan's `burler.yaml`.
  Fixed as proposed: the wording is now "a config edit".
- fix-proposal, LOW: under F2 the seam fails whenever `verify` is empty, but the seeding sentence of `TestDarnTreeScope_ClassifiesAtEachCall` named no `verify`.
  Fixed as proposed: Card 7's seed now holds a `verify` of `run all`, which the seam's `VerifyCommand` call needs to be non-empty.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
