# verify-diff-scope: darn verifies the impacted set and tree scope

All four batches recorded `done`; none needed recovery, and no digest listed a `Pattern-Departure:`.

## Batch 01-both-sides-diff-read

Added the verbatim, rename-splitting changed-path read for darn's classification.
Added `tree_scope` and `tree_scope_verify` to `darn.yaml` with their refusals and the pure tree classifier.
A hub-wide editor save now loads through the module's loader and keeps the staging copy on a refusal.
Head `d65626c3`.
Scope guard deviations: `internal/configcli/configcli_test.go`, `internal/configcli/hubwide.go`, `internal/darnengine/config.go`, `internal/darnengine/config_test.go`.

## Batch 04-module-guard-scan

Guard-scan errors gained a kind and line, and the module-wide guard scan was added.
The derived command's test and guard steps run through `lyx gate test`, and `lyx gate test --guards` was added.
Head `c5376ab0`.
Scope guard deviations: `internal/impactset/guards.go`, `internal/impactset/guardscan_integration_test.go`, `internal/loomshed/verifygate_integration_test.go`.

## Batch 07-darn-gate-impact-scope

Replaced darn's whole-diff gate form with the impact-scoped, tree-aware form, selected by a told tree-scope seam that is nil for loom.
Head `2a25846b`.
Scope guard deviations: none.

## Batch 08-darn-landing-tree-scope

Darn's Merge-In and Finalize command resolves per call, and `publish_verify` is skipped for a tree-scoped diff.
The darn writer stencil, the PATTERN backgrounds and the overview were restated.
Head `e9f38261`.
Scope guard deviations: `internal/loomcli/wiring_darn_test.go`.
