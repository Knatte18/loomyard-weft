# Plan-Review round 1 — fixer report

Every finding was validated against the code and the decision record, and all eleven are fixed.

## Fixed

- **F1** (gate-slot row): card 12's Intent now edits the loom section's `gate slot: gate.yaml absent or invalid` row, whose trigger names the `Darn` row's verify gate beside the Webster-Burler round gate and Publish's and Finalize's verify, its way forward unchanged.
  Confirmed at `contracts/specs/refusal-spec.md:289`.
- **F2** (per-symbol teardown tests): card 3 no longer plans a `settleClaim` table test or a `teardownRequest` test.
  `Teardown` gains four substrate seams (`landed`, `battenRun`, `readClaim`, `writeClaim`) in place of the `settle` seam, so `RemovePair` runs `settleBoardClaim` and `settleClaim` for real, and `TestTeardown_CallSequence` covers every rule outcome as rows driven through `Run`.
  The batten landed flag is asserted in the four-row lifecycle step (`internal/battencli/lifecycle_integration_test.go`), where only the flag can make the entry `done`; `wire_test.go` left the card's targets.
  `settleBoardClaim` now returns nothing, since it only warns.
- **F3** (redundant darn recipe assertions): the interrupt-policy and structural-check assertions are gone from `darnrecipe_test.go`;
  `TestRecipe_StructuralCheckHasNoFindings` becomes a table over both embedded recipes, and `internal/loomrecipe/recipe_test.go` is on card 11's `Edit:`.
- **F4** (webster verify seams): card 1 lists `websterengine#NewVerifyGate`, `#verifyGateSeams`, `#newVerifyGate` and `verifygate_test.go`, and its Intent says the seams drop the fixed `logPath` and read the log of the result they report on.
  The same grep found `internal/webstercli/verify_integration_test.go` reading `paths.Log` (lines 89, 98), so it is on the card too.
- **F5** (empty command in the whole-diff form): card 6's whole-diff form treats an empty command as an error and never passes it, so the form has no skip path of its own.
- **F6** (whole-diff test gaps): card 6's loomshed gate scenario gains a failure-record step (failing tests, and the `tmux`-tier `./...` for `publish_verify`), and a read-error step and an empty-command step, each asserting an error and that nothing runs.
- **F7** (clearing bound): card 3's Intent states what the clearing lets through (scope edits on a halted or unseeded run's entry, and the push-failed Finalize case, which ends cleared and needs `lyx board set-status`) and what bounds it.
- **F8** (darn seed skip): card 12's Intent states that a darn seed skips every review but PR-Gate, and that the bounds are the batten admit set over the operator's board field, or a hand `lyx shed seed`, plus `lyx loom start` keeping a recorded recipe, the single read at arming and PR-Gate's operator approve.
  Card 13 names its admit set as that bound.
- **F9** (tracked paths): card 4's `Fabric.DirtyPaths` returns the tracked uncommitted paths of both checkouts through the guard's own tracked-scope read, and its integration step asserts that an untracked file is omitted.
  Confirmed: `Fabric.Status` includes untracked files (`gitrepo.WorktreeChangedFiles`), while `pairDirtyReason` uses `scopeTracked`.
- **F10** (nonexistent params test): card 13 asserts the darn child's `params.parent` in batten's four-row lifecycle step, now run once per admitted recipe, and lists `lifecycle_integration_test.go`.
- **F11** (verified-tree opening): card 11 rewords the background's opening to the told verify command, which is the plan's in loom and `darn.yaml`'s in darn, and its `publish_verify` clause to name that command as `BaseCommand`.

## Disputed

None.

## Deferred

None.
