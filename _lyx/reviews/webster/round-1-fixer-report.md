# Fixer-report, webster round 1

## Changes

- F1, fixed as proposed (`ed74a668`): the fan key paragraph in `internal/burlerengine/doc.go` names `lenses.FanList` and `lenses.ValidateFanKeys` and resolves fans through `lenses.yaml`; the `ModelSpecList` sentence is kept.
- F2, fixed as proposed (`1d648a33`): the `ValidateFanKeys` doc comment in `internal/lenses/fankeys.go` says the load fills a lens or fan the stack lacks from the template.
- Self-check follow-up to F1: `FanShapeError` is now `lenses.FanShapeError`, the added "Both live in the lenses package." sentence is dropped as redundant, and `ModelSpecList` is marked as declared in this package.

All changes are comments; no test added, since neither finding is a coverage gap.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, `FanShapeError` unqualified in `doc.go`: fixed as proposed.
- fix-consistency, LOW, `ModelSpecList` "matching" reads as same-package: fixed, by marking it as declared in this package.
- fix-proposal, LOW, `FanShapeError` unqualified and the "Both live" sentence redundant: fixed as proposed, duplicate of the fix-consistency finding.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
