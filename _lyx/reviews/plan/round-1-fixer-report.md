# Fixer report — plan, round 1

## Changes

- **F1 (BLOCKING), fixed differently.**
  Card 3 creates `plan:internal/configsync#burlerFixture`, which marshals only the edited config's `lenses` and `fans`, and edits `TestReconcile_MovesKeysAcrossModules` to build `burlerToday` and `burlerOldTemplate` with it.
  Its ImpactSummary now separates the rows whose fixture is a lenses file, which keep their results, from the move rows' `burler.yaml` fixtures.
  Card 6's ImpactSummary says `lensesFixture` round-trips `remove: []` and `burlerFixture` never holds `remove`.
  Not made: changing `lensesFixture` itself to marshal only `lenses` and `fans`.
  `TestReconcileAll`'s row at `internal/configsync/configsync_test.go:212-219` wants the reconciled standalone file to equal `lensesLacksSecurity` byte for byte (`wantFile`).
  A fixture without `lens_set` would gain `lens_set: ""` at reconcile and fail that row.
  The other callers at `:93`, `:97`, `:429`, `:1595` and `:1596` also build lenses files.
  So card 6 needs no `lensesFixture` edit.
- **F2 (MEDIUM), fixed as proposed.**
  Card 5 gives `TestReconcile_MovesKeysAcrossModules` a `stripped` row field, checked only when set against the lenses Result's `Stripped`.
  The `board` row wants `lenses.concurrency`.
  The Intent says this is the assertion that pins `stripMovedUnits`, since the call runs without `StripBase`.
- **F3 (LOW), fixed as proposed.**
  Card 5's unknown-selector paragraph now covers every path:
  - the reconciled file where it is the base document;
  - the base layer's file on `reconcileSparseFile`, for both prune and strip;
  - the prime's copy on `SeedPrime` and on `moveKeys`' `loadTarget`.

  `moveKeys`, `LayerFile` and `BaseLayer` were added to card 5's Edit and Uses lists.
  `TestReconcile_FailureNamesItsFile` gains a `worktreeFile` field and a hub `ReconcileAll` row that wants the board file's path.
- **F4 (LOW), fixed as proposed.**
  Card 1's bound now says a base write that changes the selector leaves a hub worktree file holding the old value refused at its next load, naming the deletion.
- **F5 (LOW), fixed as proposed.**
  In card 6's refusal, dropping a name from `remove` now comes first, then re-pointing the loom or darn fan key for a later run.
  The ImpactSummary says `resolveRecordedFan` appends its own way forward.
- **F6 (LOW), fixed as proposed.**
  Card 3 drops the `testdata/go-lenses.yaml` Create bullet, its Intent paragraph and the golden assertion, and keeps the go-equals-base check at its own gate.
  Card 4 asserts that each base lens differs from the go layer's prose, decoded from `lensSets()["go"]`.
- **F7 (LOW), fixed as proposed.**
  Card 2 drops `wantSawScaffold` and TestEdit's absent standalone base row.
- **F8 (LOW), fixed as proposed.**
  Card 1 drops the local-file `pick: x` row.
  `refusalLacks` is kept, because the standalone `pick: nope` row uses it.
- **F9 (LOW), fixed as proposed.**
  Card 3 computes the emphasis-line exemption from the base template's fans only.
  Card 4 says `numerics` ends with the emphasis line.
- **F10 (NIT), fixed as proposed, then reworked in the self-check.**
  Card 1's non-scalar `selectorValue` error now gets a way forward.
  Each caller adds it:
  - on a base document, the unknown-value way forward, with hub taken from the caller;
  - on a hub worktree document, `selectorRefusal`.

  The `pick: [x]` row wants `--layer default --set pick=<name>`, and a hub worktree `pick: [x]` row wants `delete the pick line from`.
  Card 2's `setSparse` check counts a non-scalar value as other than the base's.

`lyx loom validate-plan` passes with informational findings only.

## Self-check

- fix-consistency, MEDIUM: card 5's prime-path way forward did not say how it relates to card 1's unknown-value way forward.
  Fixed as proposed: `configsync` appends `; also delete the lens_set line from <prime copy> in an editor`, and the prime rows want `delete the lens_set line from`.
- fix-claims, LOW: same finding as above; merged.
- fix-proposal, MEDIUM: same finding as above; merged.
- fix-consistency, LOW: the `moveKeys` seed path had no failure row.
  Fixed as proposed: card 5 adds a `burlerFile` field and a `ReconcileHubWideAt` row with a board `burler.yaml` that wants the prime's path.
- fix-consistency, LOW: the sentence "`reconcileSparseFile`'s prune gets no new row" read wrong beside the new failure row.
  Fixed as proposed: it now says no row pins the prune's result.
- fix-consistency, LOW: card 5's `board` row did not say how its `burler.yaml` avoids holding `lens_set`.
  Fixed differently: the row's `burler.yaml` is literal YAML holding only a `lenses` mapping.
  `burlerFixture` was not used, because it decodes the whole template and would carry every base lens and fan into the row.
- fix-proposal, MEDIUM: F10's way forward depended on hub context that `selectorValue` lacks, and was wrong for a hub worktree document.
  Fixed as proposed: callers add the way forward, `readLayers` uses `selectorRefusal` for a worktree document, and a row pins that case.
- fix-proposal, LOW: card 4 used `lensSets` without listing it.
  Fixed as proposed: added `plan:internal/lenses#lensSets` to card 4's Uses.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
