# Fixer report — code review round 1

Gate: `lym gate test --tier 0 --tier 1` over `internal/landingshed`, `internal/fabricengine`, `internal/beamcli`, `internal/loomcli`, `internal/pawlengine` and `contracts/orchrole` passed after the fixes, and over the four packages the self-check touched passed again after them.

## Changes

- **F1** fixed as proposed (`internal/landingshed/finalize_pr.go`, `finalize_pr_test.go`).
  `pullRequestLanding` merges the run's scratch pull request as it stands only when GitHub reports it open.
  A closed or merged one goes through `resolveWithParentRule` and `landingForDecision` like any other record.
  Rows added: closed scratch under `off` (dropped, local landing, no Reopen), closed scratch under `required` (resolver reopens and merges), merged scratch under `required` (already-landed path).
- **F2** fixed as proposed (`internal/fabricengine/beamline.go`, `internal/beamcli/deps.go`, `internal/loomcli/landingdeps.go`).
  `fabricengine.BeamCommitsPast` holds the one rule; `MergedLacks` and the `CommitsMissing` seam a pawl run is wired with both call it, and task recipes keep `CommitLog`.
  Tests: `TestBeamCommitsPast` and `TestLandingDeps_BeamCommitsMissingLeavesOutAMergeOfTheParent`, the latter failing under the old `CommitLog` wiring.
  `internal/fabricengine/doc.go` names the new function.
- **F3** fixed as proposed (`internal/beamcli/deps_integration_test.go`).
  `TestHubLandDeps_ParentUnmergedReadsTheUpstreamTip` pushes a commit to the hub's local bare origin from another clone and expects `ParentUnmerged` true for a beam holding the local tip, then false after the beam merges the upstream tip.
  It is not parallel and clears `FABRIC_SKIP_PUSH` with `t.Setenv`, so the environment cannot hide the fetch.
- **F4** fixed as proposed (`internal/loomcli/pawlwiring_integration_test.go`, new).
  One test runs the drop closure of `beamLandingRecordSeams` over a real beam pair; another runs `pawlRetiredMove` over a pair holding a scratch pull-request record and a ready record.
  The hub's origin is a local path, so the GitHub state read fails and the pull request is carried, as for any state that does not read; the test does not fake GitHub.
- **F5** fixed as proposed (`contracts/orchrole/orch-role.md`).
  The Parent-Align sentences of `## Landing halts` are scoped to task runs, and one bullet covers a beam's Finalize halted on a pull request GitHub refuses, naming `lym beam pr status <slug>` and `lym loom resume`, never `lym batten cancel`.
- **F7** fixed differently (`internal/pawlengine/retiredmove.go`, `retiredmove_test.go`, `doc.go`).
  Proposed change not made as stated: reading the landing record and passing its pull request on unconditionally.
  The code cannot tell a repeated move from a first move with no scratch record, so that would say "stays open" of a landing record pull request GitHub may report closed or merged (fix-code-correctness and fix-proposal lenses).
  The change made reads the landing record when the scratch record is gone and names its pull request only unless `PullRequestOpen` reports it not open; a state that does not read counts as open.
  Tests: a repeat after a failed commit names the pull request, and a repeat with GitHub reporting it closed does not.

## Disputed

- **F6**, ground `premise false`.
  The review says the refusal names GitHub's rules "when the halt comes from an unparsable github.com origin under an on or off parent".
  `ResolvePullRequest` never halts there: for a beam with no recorded pull request under `on` or `off` with no rule, no case of its `wanted` switch holds (`internal/landingshed/pullrequestmode.go:112` has `&& !isBeam`), and `!wanted` returns at line 116 before `checkOrigin` runs at line 122.
  A halt reaches `judgeNoPullRequest` only under a `required` or `merge` parent, where the text names `beam.yaml`, or with `rulesRequired` true, where it names GitHub's rules; `parentRuleRequired` reads that rule only when the origin allows a pull request.
  The current demand text is therefore right for every halt that can occur.

## Deferred

Nothing deferred.

## Self-check

Four forks, all returned a report; reports saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness, LOW: `recordedPullRequest` names a landing record pull request without reading its state. Fixed with my own change (see F7).
- fix-consistency, LOW: `internal/pawlengine/doc.go` omits the repeat behaviour. Fixed as proposed.
- fix-consistency, LOW: the `CommitsMissing` comment in `internal/loomcli/landingdeps.go` does not say a pawl run replaces it. Fixed as proposed.
- fix-affected-set, LOW: `internal/landingshed/doc.go` "as it stands" sentence. The lens found it already matches the open-only behaviour and called it not a finding; no change.
- fix-affected-set, LOW: the merged-lacking re-resolve in `recordedBeamLanding` calls `landingForDecision` with no open-scratch check. Disputed, `premise false`: the lens itself says the beam's landing record holds the pull request there, so the scratch override never applied, and it is not a defect of the fix.
- fix-proposal, LOW: a merged scratch pull request under `on` or `off` lands locally and the new rows covered only closed. Fixed as proposed for the coverage gap: a merged-under-`required` row was added; the `on`/`off` outcome is the resolver's existing handling of any recorded merged pull request and was left as it is.
- fix-proposal, LOW: F7 notice claims the pull request stays open for any landing record pull request. Same finding as the code-correctness one; fixed with my own change.
- fix-proposal, LOW: the F3 test depended on `FABRIC_SKIP_PUSH` being unset. Fixed with my own change: the test clears it with `t.Setenv` and is not parallel.
