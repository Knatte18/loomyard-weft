---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The proposed third row cannot pass: runRedFirst calls TierList before goModuleOf, and TierList refuses a hub with tiers but no tier0 (internal/gateslot/config_test.go, row `a hub with no tier0`), so `neither tier0 nor tier1` is unreachable through red-first. The row drives the reachable module refusal instead."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "cmd/testtiming is not the only remaining non-tier holder: internal/cardverify/runner.go:95, internal/driftengine/steprun.go:134 and internal/planglyph/typesload.go:114 also apply gateslot.HolderEnv. The doc lists all of them."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
---

## Changes

- **F1** (fixed differently): added `TestGateRedFirst_RefusesWhatItCannotDiffOrBuild` in `internal/gatecli/redfirst_integration_test.go`.
  Its rows are no `--base` with no parent branch, a `--base` naming no commit, and a tier dir holding no `go.mod`.
  The proposed row for a hub with neither tier0 nor tier1 was not made.
  That message in `goModuleOf` (`internal/gatecli/gomodule.go`) stays untested, since no red-first call reaches it.
- **F2**: `_lym/landing/summary.md`, `## Merge danger` names the effect on older binaries.
- **F3**: `_lym/landing/summary.md`, `## Hub config` names the manual deletion of the stored maps.
- **F4**: rewrote the `cmd/testtiming` bullet in `pattern/PATTERN-test-speed.md`.
- **F5** (fixed differently): `internal/gateslot/doc.go` lists every non-tier holder, not `cmd/testtiming` alone as proposed.
- **F6**: removed the re-run mark clause in `internal/gatefixer/rowgate.go`.
- **F7**: added a `TestLoadConfig` row for retired report-step keys, formatted with `lym loom fmt`.

F1 and F4 to F7 each have their own commit.
F2 and F3 change only a `_lym` file, which this round does not commit.

Gate: `lym gate test --gate code-burler-fix ./internal/gatecli ./internal/gateslot ./internal/gatefixer ./cmd/lym` passed.
Its first run failed on gofmt in the new F7 row, which was fixed before the passing run.
The tier1 log shows the new F1 rows passing.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW (the Merge danger sentence claimed more than older templates do): fixed, the sentence is limited to keys whose older template lists reports.
- fix-proposal, LOW ("no reader" is false while an older binary runs): fixed, the deletion now waits until every binary of the hub is updated.

## Disputed

## Deferred

## Rulings
