# Fixer report — plan, round 1

## Changes

- **F1** (fixed as proposed): card 5 gains the Create targets `noFindingsPlanIndex` and `noFindingsPlanIndex.ValidateFormat`, a test index in `fixture_test.go` that answers no findings from `ValidateFormat`.
  The subtest "PlanWriteSetsForkedOnItsPlanEntry" now sets `env.PlanIndex` to it and seeds a minimal parseable plan under `planparser.PlanDir(env.AnchorPath)` with no `lens-review.md`.
  It asserts that `Gate()` passes while `Forked()` fails with findings naming `lens-review-missing`.
  The premise was confirmed against the code: `placeholderIndex` embeds `planindex.Index` and panics on any call, and `ValidatePlan` (`internal/loomshed/gates.go:172`) calls only `index.ValidateFormat` before the webster-record checks.
- **F2** (fixed as proposed): card 1's Edit group names `clearableBoxEngine.ParseSessionSignals` (`internal/shuttleengine/run_test.go:1365`).
  Its Create group names `sessionFakeEngine.TurnStartProgress` and `clearableBoxEngine.TurnStartProgress`.
- **F3** (fixed as proposed): card 2's fixture paragraph says that each line whose shape appears in the bench transcripts is copied verbatim from `ftp-r-1` or `ftp-s-1`.
  Only the shapes the bench lacks are hand-built.
  The file names and expected readings are unchanged.
- **F4** (fixed as proposed): card 4's finding detail now ends at "as the lens review directive describes", without the format clause.
- **F5** (fixed as proposed): `00-overview.md`'s verify section runs `./contracts/specs` right after `./contracts/stencils` in all three `go vet` lines and both `go test` lines.

`lyx loom validate-plan` over the edited plan returns `ok: true` with only informational `create-new-unit` findings.

## Disputed

None.

## Deferred

None.
