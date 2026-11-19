# Hardener campaign: seven batches built

All seven batches (01, 06, 09, 13, 14, 17, 20) recorded `status: done`; no batch went through recovery.
No digest listed a rename or a Pattern-Departure, and no batch carried warnings.
The scope guard listed files outside each batch's own targets, all informational; they are named per batch below.

## Batch 01, gate-copies (head 9a3a421d)

- Cards 1–5: `lym gate test --copies` and the started-binaries line, the harden gate keys, the `hardener.yaml` unit in `hardenshed`, the loom contract's build row in a review segment, and Preflight's hardener refusals with the cyclic model rotation.
- Deviations: `cmd/lym` guard pins, `internal/gatecli` (`copies.go`, `detachfront.go`, `run.go`, `tierrun.go`), `internal/gateslot/startedlines*`, `internal/hardenshed` campaign and config files, `internal/preflightshed/preflight.go`, `internal/shedbuild/loomcontract.go`.

## Batch 06, live-gate (head bcba7793)

- Cards 6–8: the live gate and its round report, the sandbox builder and lister, the sandbox reaper, and the `PATTERN-child-processes` change.
- Deviations: `cmd/lym/testonlycode_linux_test.go`, `internal/fabricengine/seedlocalbares*`, and `internal/hardenshed` live, reap and sandbox files with their tests.

## Batch 09, round-row (head 8ad607c6)

- Cards 9–12: the round row's reap, archive and Go files, the Bouncer's facts section and halt seam, the `HardenFix` engine and harden keys, and the harden rubric stencil.
- Deviations: `internal/hardenshed` archive, campaign rows and round files, `internal/shedadapters` bouncer files, `internal/shedrecipe` entries and registry tests, and `internal/testkit/envkit/envkit.go`.

## Batch 13, hardener-recipe (head f9abf62b)

- Card 13: the `hardener` recipe and its place in every closed recipe list.
- Deviations: `internal/loomrecipe` tests, `internal/reportengine/timing_test.go`, and `internal/testkit/envkit` (`envkit.go`, `envkit_test.go`).

## Batch 14, loom-harden-wiring (head 8beee889)

- Cards 14–16: the Harden perch, campaign seams and tree scope in loom, Finalize's sandbox reap, and the `lym harden` sandbox verbs.
- Deviations: `cmd/lym` guard tests, `internal/boardcli/cli.go`, `internal/configcli/configcli.go`, `internal/fabriccli/clone.go`, `internal/gatecli/cli.go`, `internal/hardencli/cli.go`, `internal/loomcli` (`arm.go`, `darnwiring.go`, `hardenwiring*.go`), and `internal/testkit/envkit/envkit.go`.

## Batch 17, teardown-sandbox-clear (head 79b3ba99)

- Cards 17–19: pairteardown clears a worktree's sandbox root, treadle's callers are removed, and treadle with its stencils and roles is deleted, with the `agents.yaml` reinterpretation.
- Deviations: `internal/agentconfig` (`config.go`, `config_test.go`, `reinterpret.go`), `internal/pairteardown/teardown.go`, and `internal/shuttleparity/producers_test.go`.

## Batch 20, crucible-docs (head 66006462)

- Card 20: crucible is deleted and its method moved into `hardenshed`'s package doc and the docs.
- The digest listed no deviations.

## Verify gate findings

The plan-level verify failed on two findings from the findings file:

- The `internal/gatecli` package run, whose failing triage entry was `TestRunTierCopies/RedCopyMakesTheCallRed`.
- `internal/shedbuild.TestBuild_EveryRegisteredEngineBuilds/HardenFix`.

One fixer fork was spawned for them; Go appends its commits.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `8bdb302b17b7636a60f39a8166641ea91510f3bf` fix: make the red-copy test race-free and cover the HardenFix engine in the build test
