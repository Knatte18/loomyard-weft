---
dispositions:
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed-differently
    reason: "The walk goes in `runDobby`'s wire seam, not in `hubDobbyDeps`. Tier1 tests call `hubDobbyDeps` directly (internal/beamcli/dobbydeps_integration_test.go:60 and others), and hubforge removes the build stamp (internal/hubforge/hub.go:365), so a walk there would run a full hub walk inside each of them. `runDobby` (internal/beamcli/cli.go:714) is the one production caller, and a re-exec re-enters it."
  - id: F13
    disposition: fixed-differently
    reason: "A `rollback` parameter on `boardCriticalSection` would make every mutating Board method pass nil, and none of those call sites is in card 8's Edit list (internal/boardengine/siblings.go:121, projectrelease.go:21, decision.go:48, board.go:215 and others). The body moves into a new `boardCriticalSectionRollingBack` that takes the rollback, and `boardCriticalSection` calls it with nil."
---

# Round 2 fixer-report

## Changes

- **F11** (`_lym/plan/09-reconcile-before-load.md`, Tests): the `TestShedVerbs_StaleStampReconcilesBeforeALoomRecipeArms` bullet now holds the seeded pair's run lock, as the `loom step` arm in `internal/shedcli/parity_test.go` does.
  `lym shed step` arms through `loomcli.ArmAt`'s full wire and refuses at `loomPreStep`'s early probe with kind busy, before the seed commit and `reed.Up()`.
  The test asserts that the refusal is the busy one, not the moved-key one, and that the hub base `loom.yaml` holds `code.max_bounces: 2` afterwards.
- **F12, fixed differently** (card 9): `internal/beamcli#runDobby` is added to **Edit:**, with a bullet saying that its wire seam calls `hubgeom.ReconcileBeforeLoad(prime)` and then `hubDobbyDeps`.
  Every dobby start and re-exec therefore walks before its first run notice reads `tapestry.implementer`.
  The ImpactSummary names the dobby verb.
  I did not make the proposed change, putting the call in `hubDobbyDeps`: the self-check found tier1 tests that call `hubDobbyDeps` directly on hubforge hubs with no build stamp, and each of those would then run a full hub walk.
- **F13, fixed differently** (card 8): `Board.boardCriticalSection` is added to **Edit:**, and `Board.boardCriticalSectionRollingBack` to **Create:**.
  The new method takes a `rollback` and runs it under the write lock when `afterSave` or the render fails.
  `boardCriticalSection` calls it with a nil rollback.
  `RewriteRunSettings` passes a rollback that writes back the `board.json` bytes it read before the change.
  The sentences that left the moved entries saved after a render failure (old lines 41 and 49) are replaced.
  The `EnsureFailingRunSettingsCarry` row now names the rollback.
  The tier0 test is renamed `TestCarryMovedRunSettings_FailedBoardWriteIsSkippedAndLeavesTheBoard` and gains a render-failure row: a directory at a rendered output path, after which `board.json` keeps its bytes.
  The ImpactSummary and the doc sentence name the split.
  I did not make the proposed change, adding a parameter to `boardCriticalSection`, because it would force a nil argument at every other mutating Board method's call site.

`lym loom validate-plan` reports ok after every change.
The only finding is the existing informational `create-new-unit` on card 7.

## Self-check

- fix-consistency, LOW: card 8's ImpactSummary and doc sentence did not name the `boardCriticalSection` split. Fixed as proposed.
- fix-claims: no findings.
- fix-proposal, MEDIUM, merged with fix-affected-set, LOW: a walk in `hubDobbyDeps` reaches the tier1 tests that call it directly, and card 9 names none of them. Fixed with my own change: the walk moved to `runDobby`'s wire seam, which no test calls, and card 9 says the direct callers see no change.
  The fork also proposed a test row asserting that a dobby start walks. I added none, because `runDobby` runs the dobby until its context ends. The walk itself is pinned by card 9's loom and shed rows, and the wire seam is one call.

## Disputed

None.

## Deferred

None.

## Rulings

None.
