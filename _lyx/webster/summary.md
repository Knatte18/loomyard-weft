# Verify logs, fabric merge base and the darn recipe

All three batches ended `done`.

## Batch 01, verify-log-per-run

The batch failed first and needed a second recovery before it recorded `done`, at head `af89448a`.
It delivers a numbered `verify-<n>.log` per verify run with bounded history, and the failure record names the run's own log.
It also covers the Finalize parent-record commit seam, the pair-teardown board settlement, and the dirty-tree halt note listing the parent pair's dirty paths.
Reported deviations: `internal/testkit/envkit/envkit.go`, `internal/pairteardown/claim.go`, `internal/fabricengine/dirtiness.go`.

## Batch 05, fabric-merge-base

Cards 5 to 9 landed in five commits, ending at head `595473ad`.
They add the go-git merge-base read and its fabric form, the verify gate's told command and optional merge base, and the `darn` seed vocabulary entry.
They also add `internal/darnengine` with its hub-wide config, verify-command reader, stencil and spec.
Reported deviations were test wiring (`wire_seams_test.go`, `fixture_test.go`, `publish_verify_test.go`), the gate and arm wiring files in `loomcli`, `loomshed` and `landingshed`, `internal/gitrepo/doc.go`, and `fabricengine/warpforward.go`.
The test budget file and the strict config set in `cmd/lyx` were raised as well.

## Batch 10, darn-producer-and-entry

Cards 10 to 14 landed in five commits, ending at head `2abe1c81`.
They add the `Darn` row producer and its `DarnWrite` registry entry, the embedded darn recipe, `loomrecipe.NewDarn` with the verify-budget floor, loom's darn arm, batten admitting `darn` children, and the orch stencil, board skill, help and overview wording.
Reported deviations: new `loomrecipe/darnrecipe.go` and `loomcli/darnwiring.go`, edits to `loomshed.go`, `interruptpolicy.go`, `loomcli/cli.go`, `loomcli/wiring.go` and `battenshed/deps.go`, test fixture updates in `shedrecipe`, `battencli` and `contracts/stencils/claims_test.go`, and raised test budget rows.


## Audit warnings

These findings (fork-audit policy findings, and drift about a later card) were recorded as warnings and did not stop the run.

- audit warning (fabric-reference): accepted by accept-audit --batch, HEAD was the batch's start commit and the tree clean: ran a fabric-referencing command ("sed -n 285,300p contracts/specs/refusal-spec.md | cut -c1-200; grep -n \"fabric commit\" -r contracts/specs plugins 2>/dev/null | head -5; lyx fabric --help 2>&1 | head -30") that can rewrite run state — an implementer fork must never touch the fabric repo directly
