# Regression baseline and verify: drift engine, CLI and recipe rows

All three batches recorded `done`, with no `departures` and no `renames` in any digest.

## Batch 01 (drift-config, head 76e4c93d3)

Built the `internal/driftengine` package: the hub-wide drift config module and paths, the scope and capture records with the counted-capture predicate and run counter, the `drift-verify` gate key and its mirrors, breakaway-admitting Windows jobs, the capture run, side capture, detached spawn and lock probe, and the compare run with its skip list.
The scope guard listed deviations in `internal/configcli/configcli.go`, `internal/configreg/configreg_test.go` and files of `internal/driftengine` outside the batch's declared targets.

## Batch 07 (drift-verbs-cli, head 1a614898a)

Built the `lyx drift` verb rules and the `internal/driftcli` subtree, the two agent stencils with their seat tables, the shared `Deps`, and the Drift-Baseline, Drift-Await and Drift-Verify producers.
The scope guard listed deviations in `cmd/lyx/sandbox_coverage_test.go`, `internal/driftcli` and `internal/driftengine` files.

## Batch 12 (drift-activity, head 3fdf33415)

Added the capture lock and compare marker as loop activity, the pair teardown step that ends the capture's named job, and the Drift rows in the loom and darn recipes with their registry entries and wiring.
The scope guard listed deviations in the recipe, registry and teardown tests of `internal/loomrecipe`, `internal/shedrecipe`, `internal/loomcli`, `internal/battencli` and `internal/pairteardown`, plus `internal/testkit/envkit/envkit.go`.

The gate runs Linux, so the Windows-only breakaway and named-job paths are pinned by their seams only.

## Verify gate findings

The plan-level verify failed on the `internal/shedbuild` package (attempt 1 of 2); the findings file reports no recorded card touching it.
One fixer fork was spawned for it.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `b5ac95755fa2821764949ba40883b8b7d34a5269` fix: give DriftBaseline and DriftVerify minimal config in the every-engine build test
