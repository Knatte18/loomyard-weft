# loom-tail-review: run tail rebuilt around Merge-In, a merge fixer and a locked Finalize

All 27 batches (52 cards) reached `status: done`.
Batches 01 and 33 failed `record-batch`'s done-check, because a Create target named a test in the `_test` package of its unit. A backgrounded `recover-batch` finished each as done.
No batch was stuck or dead, and no friction note was written.

## What was built

- **Building blocks (01, 07, 12):**
  - Fabric merges a given SHA through `MergeInAt`, and `mergeresolve` carries the merged tip.
  - `landing.yaml` gained the fixer keys, and the fixer report has a spec and one parser.
  - A fixer session table, `Fabric.LineChange`, `FixCap`, `FixerAttempts` and one shared fixer loop were added, and `landingDeps` fills their seams.
  - Darn-Write lost its failure reroute, and the settled-segment guard lost its verify-failure exception.
- **Old failure routing removed (15, 17, 19):**
  - The round gate, review rubric, Go driver, arbiter and batten no longer read the failure record or run a verify-halt or publish retry.
  - The failure record lost its target and route classifier.
  - The PR-Review stub engine and its flags were added.
- **Recipes (21, 22, 23):**
  - The `webster_*` perch keys became `code_*`, with a migration.
  - Both recipes took their new tails and the Code-Review ring.
  - The Merge-Review seams, rubric and keys were retired.
- **Merge-In and Finalize (24, 28):** the merge record carries the aligned tip, Merge-In merges the upstream tip with no parent lock, and Finalize takes the lock first. Finalize merges the parent and reroutes a conflict or large merge, runs the landing gate and finalize-mode fixer with caps, and re-checks its lock before landing with the task trailer.
- **Review and landing halts (31, 32, 33, 35, 38):**
  - The Bouncer re-entry seam feeds a newer review input.
  - Task-diff readers take the merged tip's merge base.
  - The fix rows run the landing gate.
  - `landingshed.HandsBack` marks landing halts, and the Go driver, arbiter, batten's binary re-step and the LLM driver stencils skip or refuse them.
  - The reroute-cap clause names Finalize's fixer causes, and idle release keeps the lock of a running Finalize.
- **PR surface removed and config migrated (41 to 48):**
  - Batten's PR-Gate lock arms are gone, and so are Publish, PR-Align, PR-Gate, the approve and reject verbs, PR-Rework, the alignment pr-gate point and `alignment_pr_approve`.
  - The Publish failure record became the landing failure record.
  - `require_pr_to_base` became `deploy_branches`, and the `publish` gate kind went with a gate migration.
  - Run-settings migration lines log at Warn.
- **Reports and cleanup (50, 51):** `reportengine` and `tokencount` read old and new runs, and the retired row-name constants were deleted.
- **Docs (52):** the PATTERN lines, background files, `README.md`, `docs/overview.md`, `CLAUDE.md`, specs and package docs follow the new tails.

## Deviations and departures

Each batch's `deviations` list names the files its glyph scope guard found changed outside the batch's own targets. They are test fixtures, seam lists and docs that a card's change broke, and they are informational.
No batch reported a `Pattern-Departure:` trailer, so `departures` was empty throughout.

## Failures batch forks reported

Two forks reported failing tests they judged outside their cards, and I did not check either report:

- **Batch 43:** `TestDriverStencil_IsRecipeBlind` in `cmd/lyx`.
- **Batch 47:** six tests in `internal/loomrecipe`, five `TestResume_*` and `TestSequence_FullRunBlocksAtMergeIn`.

## Verify gate findings

The plan-level verify gate failed on its first attempt (attempt 1 of 2), with these failing identities:

- `cmd/lyx.TestDriverStencil_IsRecipeBlind`
- `internal/shedbuild.TestBuild_EveryRegisteredEngineBuilds/PRReview`
- `internal/loomrecipe.TestResume_DoesNotRestartAtRowOne`
- `internal/loomrecipe.TestResume_CrashRecoveryRecallsUnconditionally`
- `internal/loomrecipe.TestResume_PauseStopsAtBoundaryAndClearsFlag`
- `internal/loomrecipe.TestResume_DiscussionWriteRespawnsRatherThanReportDoneOffFileExistence`
- `internal/loomrecipe.TestResume_LiveMatchingRunAttachesInsteadOfRespawning`
- `internal/loomrecipe.TestSequence_FullRunBlocksAtMergeIn`

The findings file names cards 21, 22, 43 and 47 as the ones whose commits touched a failing package.
One fixer fork ran for this attempt, and the gate verifies again after this turn.

## Left for later

- Stale mentions remain of the Publish failure record or the Publish row in `contracts/specs/loom-final-summary-spec.md` and `webster-spec.md`.
- `loomcli/runstart.go` (`findRunPullRequest`) has no production caller now, and `parentlock.Store.HandOff` has none either.
- `TimingAttempts.Alignment` stays and reads zero.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `dc3edaca196f1f5dd66fedf366f4b9fd48914d4d` fix: recipe-blind driver stencil, PRReview engine fixture row, loom fixture stops Merge-In at the origin reconcile


## Audit warnings

These findings (fork-audit policy findings, and drift about a later card) were recorded as warnings and did not stop the run.

- audit warning (later-card-drift): later card: plan-references-deleted-symbol/42-landing-drops-pr-producers[blocking]: card 42 references "internal/shedrecipe#armPRGate", which the delta reports deleted with no corresponding rename
- audit warning (later-card-drift): later card: plan-references-deleted-symbol/42-landing-drops-pr-producers[blocking]: card 42 references "internal/shedrecipe#armPRAlign", which the delta reports deleted with no corresponding rename
