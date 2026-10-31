## Changes

Every finding is fixed; each fix is its own commit.

- **F1** (tests for `decision add` on a beam): `decisionParent` and `decisionReceivers` now take the orch-live and live-strand seams, so the beam rules run over a forged hub.
  New tier-1 tests in `internal/loomcli/projectrecord_integration_test.go` cover the beam-orch caller rule, `memberPairs` (member pair, forked pair, deduplication, member with no pair) and the receiver fan-out for a project record against a discussion record.
  `TestRunRecordOf` now pins the `_lym`-relative record path for a beam and a task run.
  Fixed differently: the proposed assertion that an absent pair worktree is skipped is not made, because that branch in `memberPairs` cannot be reached by a fixture; see Self-check.
- **F2** (`withProjectDirective` untested): `TestWithProjectDirective` in `projectrecord_test.go` covers a member under a beam (directive names the record) and a pair under no beam (marker empty, other values untouched).
  The self-check added a darn-table assertion that a blank `project_directive` validates.
- **F3** (wiring sites): `wiring_beam_test.go` asserts the pawl row's `DecisionRecordPath`, `SupportLogPath`, the empty `PatternRecordPath`, the Record-Write record, support log and directory, the arbiter prompt's record, the record the BeamDescribe seam reads, and the landing deps' `DecisionRecordPath` per recipe.
  `TestCommitRunRecord_ProjectRecordCommitsItsDirectory` (tier 1) pins that a project commit stages the project directory and not the discussion record.
- **F4** (goto to Preflight lifts the body lock): fixed by the doc option; the `ProjectRecordHeld` comment states that the hold is judged from the status alone and that a goto to Preflight lifts it while Record-Write keeps the record.
- **F5** (seed-read refusal): the `decision add` refusal names a way forward.
  The self-check found the same refusal in the review verbs' target resolution and gave it the same way forward.
- **F6**: removed the static-prose loop from `RecordsRenderAsBulletsInOrder`.
- **F7** (beam with no resolvable orch): fixed with my own change, not the proposed one.
  The proposed `lym beam start <beam-slug>` cannot resolve the case, since `NearestLiveOrch` returns an empty parent only when the hub records no shortname, and falls back to the prime's orch when the beam's orch is not live.
  The refusal now says the hub records no shortname and names `lym fabric shortname <shortname>`; the tier-1 test asserts it.
- **F8**: removed the repeated `Rel`-form assertions and kept the two-beams-differ check.

## Self-check

- fix-consistency, LOW: the review verbs' seed-read refusal lacked the way forward `decision add` now has. Fixed as proposed by the lens (`review.go`).
- fix-code-correctness: no findings.
- fix-proposal, MEDIUM: F7's `lym beam start` way forward cannot resolve the only case that reaches the branch. Fixed (own change above).
- fix-proposal, LOW: the F2 test did not pin that a blank marker is accepted by the darn table. Fixed with the `Validate` assertion in `darnengine/spec_test.go`.
- fix-proposal, LOW: the "absent pair worktree is skipped" branch of `memberPairs` is never reached by the F1 fixture. Not changed: `ReadBeamLandBlockers` already drops a member whose pair is absent and a worktree gone from disk is listed as prunable and dropped, so the `os.Stat` branch is reachable only by a race, and a fixture cannot reach it.
  The test pins the observable outcome, that a member with no pair is not returned.
- fix-affected-set, LOW: `wiring.go:838` (`driftDeps`) still reads the discussion record. Not changed: the pawl recipe has no Drift row, so a beam never reaches it, and the review judged the equivalent `landingdeps.go` and `arbiterprompt.go` sites the same way.

No fork returned without a report.

## Disputed

None.

## Deferred

None.
