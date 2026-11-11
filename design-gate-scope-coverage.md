# A coverage gate scope runs changed packages whole and importing packages' selected tests

`gate-scope-coverage` · Task · feature, gate, testkit, darn · running · Open-Window

Decision record: [Sketch](decision-gate-scope-coverage.md)

Depends on: [`coverage-test-select`](design-coverage-test-select.md)

Filed from the Sketch `gate-scope-by-coverage-map`.

Takes: coverage-test-select's selector and the diff scope as impactset.DeriveTargets and computeScope derive it
Delivers: scope: coverage in gateslot and the template's darn-write, webster-fix and code-burler-fix keys; DeriveTargets keeping changed and lym-binary packages apart from importers; the D3 union per importing package and tier; a second tier invocation with an anchored -run in the run's own args, its green entry carrying them, over-budget fallback to whole; the per-package scope report; maps read only, never built
Exports: the coverage scope value and impactset.Targets' split of changed packages from importers

The part changes these packages:

- internal/gateslot (about 80 changed lines)
- internal/impactset (about 150 changed lines)
- internal/gatecli (about 750 changed lines)

## Changes from the upkeep design (D13 of `testmap-upkeep-design`)

Read the record with `lym board get testmap-upkeep-design --record`; this part follows it where it changes this entry:

- In its D3 union, "every test its map does not record" includes a test whose entry records only an outcome (D4).
