# Plan round 4 — fixer report

All seven findings were checked against the code, the plan spec, the writer stencil and the decision record, and every one holds; all are fixed.
`lyx loom validate-plan` passes over the result with informational findings only (see the gate note below).

## Fixed

- **F1 (BLOCKING), cards 12, 14, 17, overview.**
  Confirmed against `freshPendingDrop`, `headBeforeEveryStart` and `AcceptPendingAudit`: on the archive-only path the HEAD refusals named `accept-audit`, whose `reconcileHead` on the last batch head refuses back toward a reset.
  New shared decision `reset-guard-judges-against-head`: the reset's guard judges every pending path against the worktree's HEAD and reads no recorded commit.
  After a moving reset HEAD is the start, so the checks match `run --fresh`'s; on the archive-only path HEAD is the tree the next run starts from.
  Card 12 factors `freshPendingDrop`'s path checks (contract, suspect, plan) into one unexported function in `archive.go` that takes the evidence base and the caller's step.
  `freshPendingDrop` keeps its missing-commit, `headBeforeEveryStart` and HEAD refusals and its texts.
  The only change is that its HEAD refusal now fires before the path checks, which the card states.
  `ArchiveRunAfterReset` calls the shared checks with HEAD, so it refuses only on `rm`, `restore-plan`/`rebaseline --card` or `git checkout <HEAD> -- <path>`, each a step that clears the state, then the reset again.
  It drops every other finding with a warning.
  `headBeforeEveryStart` leaves card 12.
  `TestArchiveRunAfterReset` gains the rewritten-branch row that archives with a drop warning per finding.
  The refusal-spec row lists only those reachable steps.
  Card 14 says no archive-only cause reaches a guard refusal, and card 17 rewords only `Run --fresh`'s texts.
- **F2 (MEDIUM).**
  Swept every signature change in the plan and wrote the re-sign arrow on each one:
  - card 2: `costBatcher.Batch`, `identityBatcher.Batch`, `PeakContext`, `breakdownOf`, `startContext` and `peakAccumulator.start`;
  - card 5: `NewLoomPreflight`;
  - card 8: `formBatches`, `batchCards` and `ExecutionBatches`;
  - card 9: `startEstimate` and `Calibration.addStarts`, which now takes the history and base trees and returns an error, stated in the Intent;
  - card 11: `PlanReset` and `resolveResetSHA`, which takes the geometry and the batch number, stated in the Intent;
  - card 16: `reconcileReportHead`;
  - card 17: `fingerprintMismatchWayForward`;
  - card 18: `RenderRecoveryPrompt`;
  - card 23: `NewPlanDirRotator`, now a listed member.
  `Batcher.Batch` stays without an arrow, since it is an interface method.
- **F3 (LOW).**
  Card 21 lists `internal/planparser#splitFraming`, and card 23 lists `internal/loomshed#NewPlanDirRotator`.
  `loomCLI.wire` moved to card 24.
  The gate then reported `redundant-file-target` for a file listed beside one of its own members, which contradicts the review's "keep `parse.go`" advice.
  So card 21 drops `parse.go` and card 24 drops `wiring.go`.
  Each card names its new unexported helper (`splitFraming`'s bounds helper, `planCommitPathspec`) as part of the member's change.
- **F4 (LOW).**
  New card 13 `uncommitted-paths` creates `plan:internal/websterengine#UncommittedPaths` with the accept-audit refactor.
  Its existing tests (`TestAcceptBatchFabricReference_ToleratesTheRunsOwnState`, the "dirty worktree" refusal row) keep pinning the filter.
  Cards 14, 15 and 18 `Use` it.
  Card 22 was split into card 23 (the rotation seam, `ArchivedPlanWebsterDirs` declared as a `plan:` handle, PATTERN, refusal row) and card 24 (`CommitPlan`'s pathspec, `TestPlanCommitPathspec` and its `internal/loomcli` budget raise).
  The old cards 13–22 are now 14–23.
  Every prose cross-reference, the Card Index and the Shared Decisions' `Applies to` lines were renumbered and checked.
- **F5 (LOW), card 18.**
  `recoverSpawn` lists the paths with `UncommittedPaths(deps.Geom)`, which is in the card's `Uses`; the `Git` use is dropped.
- **F6 (LOW), card 15.**
  `TestKeepResetRefusal` and its `internal/websterengine` budget raise are dropped.
  `TestResetCmd`'s keep-refusal step now runs over two dirty files the move would overwrite, one run-written and one foreign, and asserts each path's way forward.
  It is `KeepResetRefusal`'s one test.
- **F7 (NIT).**
  Card 11 rewrites `resetCmd`'s `Short` and the `--to` flag description.
  Cards 14 and 15 rewrite the `internal/webstercli/reset.go` file header for the archive and the standalone keep-reset.

## Gate note

`caller-uncovered` lists 75 informational entries on card 2, all false positives.
They are every file that names the `batcher.Batch` type, read as callers of the re-signed `costBatcher.Batch`/`identityBatcher.Batch`, and unrelated `.start(` calls read as callers of `peakAccumulator.start`.
A grep for real calls finds `partition.go`'s `batchCards`, `cost.go`, `estimate.go` and the batcher and partition tests, and every one is already on card 2 or card 8.

## Disputed

None.

## Deferred

None.
