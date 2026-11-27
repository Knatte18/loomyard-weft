---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed-differently
    reason: "The lead's limit is raised only while a reviewer runs. The proposal raised it also with an empty landing.describe_review, where there is no review to give time to, so describe_timeout_min would no longer bound the writer (internal/landingshed/describe.go, DescribeTable). A describe_timeout_min below 1 is also left as it stands, so zero keeps the shuttle default."
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
---

## What changed

- F1: added `describeReviewTimeoutMinKey` and used it in the Warn, the `ConfigSpec` map and `LoadConfig`'s positive-value list.
- F2: the unbuildable-text Warn says the review stays open and names the way forward.
  The way forward covers any named cause, not only the change description, since the text can also fail on the engines or regression-check read.
  `ADescriptionThatYieldsNoTextHoldsTheContinuedEnding` asserts the clause.
- F3 (fixed differently): while a reviewer runs, the lead's and the reviewer's limits are `describe_timeout_min` plus `describe_review_timeout_min`.
  The proposed change not made is raising the lead's limit with no reviewer.
  Updated the template and field comments, the doc comments, `docs/overview.md`, the change description and `TestDescribeTable_ReviewerSeat`.
- F4: added a forbidden-term row to `TestReplaceBeamDescription`.
  `TestHubLandDeps_DraftThenRecordStampsAndCommits` (tier1) now drives both recorder paths and `DescriptionFindings` under the template's `landing.yaml`.
  The test is there rather than a new one, since `hubLandDeps` resolves the beam from a real worktree.
- F5: the beam describer's stencil states the rule about the tooling's names.

F1 and F3 share files and are one commit; F2, F4 and F5 have one each, and the self-check added two.
The change description (`_lym/landing/summary.md`) was edited for F3 and the self-check, and is not committed by this round.

The round's gate then found a wrapped comment line in the `describeReviewSeat` doc and one in the `syncText` doc.
Both are rewritten as one sentence per line, in a commit of their own.

Gates passed after that repair: tier0 and tier1 over `beamcli`, `landingshed`, `dobbyengine`, `contracts/stencils` and `shuttleparity`.
Tier0 over `shedrecipe` and `loomcli` passed on the self-check changes, before the comment repair.

The verify command then failed `TestGofmt_ModuleFilesAreFormatted` on a line of `beamdescribe_test.go` that the F4 change left unformatted.
`lym loom fmt` fixed it, in a commit of its own.
Tier0 over `cmd/lym` and `landingshed` passed after it; the full verify command was not rerun by this session.

## Self-check

- fix-code-correctness, LOW (a zero `describe_timeout_min` became the review timeout alone): fixed; the sum is taken only for a positive value, with a test row.
- fix-consistency, LOW (`beamDescribeDeps` comment says the Describe row runs on the same two keys): fixed; the sentence is removed.
- fix-consistency, LOW (change description omits the beam describer's prompt rule): fixed.
- fix-proposal, MEDIUM (texts claim a reservation the single limit does not make): fixed; the three texts now say it holds for a writer that finishes within `describe_timeout_min`.
  A writer that runs past that still eats into the review's time, down to none at the combined limit.
- fix-affected-set: no findings.

## Disputed

## Deferred

## Rulings
