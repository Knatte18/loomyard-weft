# Fixer-report, round 1 (code)

## Changes

- F2 (MEDIUM, design): fixed as proposed in `internal/beamcli/dobbydeps_integration_test.go`.
  The new `TestHubLandDeps_FileTaskReleasesTheTaskAndCodeReviewCapsReadTheHubConfig` runs `FileTask` over the `landingHub` hub, reads the board through `Tasks`, and asserts the task exists with a nil (released) status.
  It writes a hub loom.yaml setting `code_max_bounces: 7` and `code_hard_max_bounces: 9` and asserts `CodeReviewCaps` returns 7 and 9, so a wrong perch or config source fails it.
- F3 (LOW, consistency): fixed as proposed.
  `openMergeTask` uses `boardengine.IsDoneStatus`; `mergeparent_test.go` builds `done` from `boardengine.StatusDone`.
- F4 (LOW, design): fixed as proposed, then amended after the self-check.
  The `FileTask` closure in `internal/beamcli/deps.go` returns an error naming the task slug, saying it stands as a draft with no run, and naming `lym board remove <slug>` as the step to take before filing again.
  The two-step release stays.

Commits: one per finding (F2, F3, F4), then one amending F2 and F4 from the self-check.

## Self-check

- fix-code-correctness, MEDIUM: the `CodeReviewCaps` assertion compared against the same call chain and could not catch a wrong perch or source. Fixed (own change to F2's test, as above).
- fix-code-correctness, LOW: the F4 message said "release it, then retry", which contradicts the rerun's refusal, and `lym board` named no verb. Fixed: the message now names `lym board remove <slug>` before filing again.
- fix-consistency, LOW: the F4 message named a bare `lym board`, unlike sibling errors. Fixed with the concrete verb. Its remark that the rerun refusal stays as described is the case the fixed message now tells the operator to clear first.
- fix-proposal, MEDIUM: the rerun still refuses with a `lym batten cancel` way forward for a draft. Partly fixed: the first error now tells the operator to remove the draft before rerunning, so the rerun finds none. Changing the "stands open" refusal to tell drafts apart is not made: the review's fix key says to keep the rerun's check and change the error, and the narrow case is covered by the new message.
- fix-proposal, LOW: same as the fix-code-correctness MEDIUM. Fixed.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- F1 (MEDIUM, decision class): left to the parent per the instructions; the parent recorded the departure with `lym loom decision add` (creel #653), settling it, and the code stays as built.
