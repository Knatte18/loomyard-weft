# Fixer report — darn round 1

## Changes

- **F1** fixed with my own change: `internal/configsync/movekeys.go` gains `formerSourceDefaults`, which `takeKey` consults at a base layer to drop, and report in `KeptTarget`, a darn value equal to its key's former template default.
  The review proposed restoring the darn entries in `formerTemplateValues`; I did not, because `TestFormerTemplateValues_DifferFromTheTemplate` requires every entry to name a unit the module's template still holds, and darn's template no longer holds these keys.
  The table lists the five keys the review named plus `review_fan` and `review_fix_fan`, whose older default was `""` (found by the self-check).
  New rows in `TestReconcile_MovesKeysAcrossModules` and a pin test that every table key is a declared move source; the `moveKeys` and `Result.KeptTarget` docs name the rule.
- **F2** fixed: `internal/loomcli/wiring_darn_integration_test.go` (tier 1) drives `commitDarnDiscussion` over a real pair and checks the snapshot is in the discussion commit and a missing entry commits nothing; a tier 0 subtest of `TestWire_DarnRecipe` pins that the seam `wireDarn` sets writes the snapshot.
- **F3** fixed: rows for `darn_reviewer_model`, `darn_fixer_model`, `darn_reviewer_fan`, `review_fan`, `review_fix_fan`, `review_hard_max_bounces` and `review_amend_max_bounces` in `TestReconcile_MovesKeysAcrossModules`, plus a kept-loom-value row at a base and at a sparse layer.
  The `TestPrepareRunSettings` rows for the fan keys expect the guard refusal, since fan keys are guard keys in run settings.
- **F4** fixed: `internal/configcli/doc.go` and the profile fixtures of `configcli_test.go` use `{"loom":{"code_max_bounces":2}}`; the twin example in `internal/boardengine/doc.go` is updated too.
- **F5** fixed: `TestRerouteReviewed_ReadsTheCodeBudgetFromLoomYaml` in `landingdeps_test.go`, covering budget 0, budget above 0 and a load failure for both seams.
- **F6** fixed: the `OpenHub` and `GetTask` refusals of `writeBoardEntrySnapshot` end with `way forward: fix the board, then hand off again`.

## Disputed

- **F7**, ground `premise false`: the row "a code cap, the darn recipe's perch's too" is the only `Code-Bouncer` row of `TestPerchHardCap` (`internal/loomcli/circling_test.go`, rows Plan-Bouncer, Discussion-Bouncer, Code-Bouncer; `grep` finds no other `code_hard_max_bounces` test there).
  Deleting it would leave the `loomPerches` mapping of `Code-Bouncer` untested.

## Deferred

None.

## Self-check

- fix-consistency, LOW, `moveKeys` doc did not state the former-default rule: fixed as proposed.
- fix-consistency, LOW, `boardengine/doc.go` example still names the retired key: fixed as proposed (doc only; the render test uses generic run-settings data and stays).
- fix-affected-set, MEDIUM, `review_fan` and `review_fix_fan` missing from `formerSourceDefaults`: fixed as proposed, with a test row.
- fix-affected-set, LOW, `internal/runprofile/template.yaml` and generic fixtures still use darn's retired profile keys: not changed; they work through the run-settings move, and rewriting shipped starting profiles is an improvement F4 did not ask for.
- fix-code-correctness, MEDIUM, the integration test calls `commitDarnDiscussion` directly, so the `wireDarn` assignment is unpinned: fixed as proposed, with a tier 0 subtest that calls the wired seam.
- fix-proposal: returned no findings.
