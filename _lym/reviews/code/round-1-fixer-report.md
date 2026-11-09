# Fixer report, code review round 1

Every finding is fixed as proposed, except where marked. Tier 0 passed over `internal/loomcli`, `internal/loomshed`, `internal/loomrecipe`, `internal/loomengine` and `internal/runprofile`; tier 1 passed over `internal/loomcli`, where the F2 and F3 tests ran.

## Changes

- **F1** (`internal/loomcli/decision_test.go`): the fake gained error fields for its three raddle seams.
  `TestDecisionVerb_AppendsAndCommits` gained a row for a perch-mode run at Raddle-Write and a row for a perch-mode state over a recipe with no RaddleFile row.
  `TestDecisionVerb_Refusals` gained one row per seam failure, each asserting the "read the run's raddle filing" refusal and its way forward.
- **F2** (`internal/loomcli/raddlewiring_integration_test.go`): `TestRaddleRunFiling` covers no store entry, another Sketch's entry, an earlier entry's created stamp, a match, a match with its board task gone, and a board read failure.
- **F3** (`internal/loomcli/raddlecontext_integration_test.go`): `TestRaddleBoardContextPerch` covers perch state, interactive state, no state file and an unreadable state.
  The `forgeRaddleRun` helper is shared with F2's test.
- **F4** (`internal/loomcli/raddlewiring_test.go`): `TestRaddleWriteEnvStoreCold` asserts the cold proposal and `cold_stored` are set in perch and auto mode and not in interactive mode.
- **F5** (`internal/loomshed/raddlewrite_test.go`): the auto-approve error subtest became a table that also holds a `SplitPerch` error row.
  The row asserts the error names the split-perch read, the state records nothing and the inner producer never runs.
- **F6** (`internal/loomengine/template.yaml`): the `raddle_auto_approve` comment now ends ", and never in a run whose recipe rows hold a split perch", the wording of `config.go`.
- **F7** (`contracts/stencils/raddle/raddle-template-agent.md`): the heading reads "the entries that get approved".
- **F8** (`internal/loomrecipe/recipekeys.go`, `recipekeys_test.go`): `RowsHoldSplitPerch` compares `filepath.Clean(filepath.FromSlash(text))`, and the test holds a `./`-prefixed row.
- **Self-check follow-up** (`internal/runprofile/template.yaml`, `docs/overview.md`, `internal/loomcli/raddlewiring.go`): three further descriptions of `raddle_auto_approve` now name that a split-perch run never reads it.

- **Gate repair** (`internal/loomcli/raddlecontext_integration_test.go`): the `forgeRaddleRun` doc comment ran past the line cap, so no test ran; it is now two sentences on two lines.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, twin descriptions of `raddle_auto_approve` in `internal/runprofile/template.yaml` and `docs/overview.md` omit the split-perch exception; fixed as proposed.
- fix-proposal: no findings.
- fix-affected-set: LOW, the same runprofile twin, plus `internal/loomcli/raddlewiring.go:39`; merged with the consistency finding and fixed as proposed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
