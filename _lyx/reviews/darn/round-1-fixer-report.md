# Fixer-report, darn round 1

## Changes

- F1, fixed with a change of my own to one row: `TestSetValues_Refusals` gained "entry key into a per-name mapping map that holds a list" (`internal/yamlengine/set_test.go`), and `TestSetSparse` gained "per-name mapping entry key under a scalar entry is refused" (`internal/yamlengine/layers_test.go`).
  The proposed `SetValues` row used `profiles.costed.budget`, which the template holds as a plain leaf, so it takes the plain-leaf path and is never refused; the row uses `profiles.wide.budget`, a name the template lacks, which reaches the entry-leaf branch and `perNameShapeError`.
  A self-check fork found the `mappingAt` refusal on the per-name map itself untested in `SetSparse`; a third row, "per-name mapping entry key into a per-name map that holds a list is refused", pins it.
- F2, fixed as proposed: the comment above `formerTemplateValues` (`internal/configsync/configsync.go`) says a per-name map is compared per leaf unit.
- F3, fixed with a change of my own: `TestActive` gained the row `profilesThatIsAListErrors` (`internal/batcher/config_test.go`), seeding `profiles:` as a list and expecting an error naming `batcher.yaml`, `profiles` and `sequence`, marked invalid.
  The proposed target, the error return of `profileOrigins`, is not reached through a layer file: the layer merge in `LoadOrTemplate` refuses a list against the template's mapping first (`key "profiles": template holds a mapping, file holds a sequence`).
  The row pins that observable refusal instead.

Commits, one per finding: F1 `f1863bea3`, `4d8d9fa18`; F2 `b04829910`; F3 `635f47bb5`, `032e18feb`.
`lyx gate test` on `./internal/yamlengine`, `./internal/batcher` and `./internal/configsync` is green.

## Self-check

- fix-code-correctness, MEDIUM: the F3 row may not reach `profileOrigins`; judged true by running it (the merge fails first), fixed with my own change as under F3.
- fix-proposal, MEDIUM: same F3 finding; same disposition.
- fix-affected-set, LOW: the `SetSparse` refusal at `layers.go` on the per-name map itself is untested; fixed with the third F1 row.
- fix-consistency: no findings.

## Disputed

None.

## Deferred

None.

Open point: `profileOrigins`' error return stays unreached by any test because the merge refuses first; it is a defensive branch.
