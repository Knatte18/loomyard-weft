---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed-differently
    reason: "The reviewer has the writer's budget as proposed. The writer's wait is not measured from a persisted first-run time: that is wall-clock time, so a pause, a machine sleep or a re-attach would pass the text unreviewed, while the run deadline counts awake, unpaused time (internal/shuttleengine/doc.go). The limit is unbroken waiting held in memory, restarted after a gap between checks. template.yaml, config.go and docs/overview.md say so instead of keeping their old wording."
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed-differently
    reason: "describe-review.sent is not written before the findings are returned: a re-prompt refused as busy, unlanded or paused is retried by re-evaluating the gate (internal/shuttleengine/wait.go, handleGatedBoundary), so an early record drops the findings. It is written once the re-prompt is delivered, through a new GateResult.Reprompted recorder."
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
  - id: F19
    disposition: fixed
  - id: F20
    disposition: fixed
  - id: F21
    disposition: fixed
---

# Code review round 1 — fixer report

All findings are fixed and committed.
The round's gate then refused comment lines that wrapped mid-sentence or reached the line limit, so it ran no test.
Those comments are rewritten, with no change to code, and committed on their own.
`lym gate test` (tier0 and the guard step) ran after that repair and reported 0 failed.
Nothing was pushed.

## What changed

- **F2, F4, F5, F7, F12, F15, F17, F18, F20:** the `Describe-Review` entry's first call writes `describe-review.ready`, which the reviewer now awaits.
  A fresh spawn removes that record and the sent record through `landingshed.ResetDescribeReview`.
  The reviewer runs on the writer's budget, and a writer wait past `describe_review_timeout_min` passes with a Warn.
- **F1, F3, F8:** the `validate-description` refusal names its way forward, and rows show forbidden patterns reaching the checks.
- **F9:** Code-Review flags the tooling's names in documents and in the description.
- **F10:** an agent whose operator-started turn ends with outputs missing is typed one line naming them (the fix key's first option).
- **F11:** a new `loom-fragment-tooling-names` is included by the briefs that commit code; the conflict resolver and the arbiter carry the rule as a sentence.
- **F13:** PR-Review compares the text under the continue flag too, and holds the review open while the text is stale.
  The commit carries a `Loosened-Check` trailer for the one row whose expected calls changed.
- **F14, F16, F19, F21:** as the fix keys say.
- **F6:** `_lym/landing/summary.md` calls the door one-way and the blast radius loom, and its text is brought up to date with the other fixes.
  It is not committed, being under `_lym`.

## Self-check

All four forks returned a report.

- fix-consistency, LOW, review-directory comments name only two files: fix as proposed.
- fix-consistency, LOW, `describe_timeout_min` descriptions omit the reviewer: fix as proposed.
- fix-consistency, LOW, reviewer stencil header omits the seat's values: fix as proposed.
- fix-affected-set, MEDIUM, arbiter and card-fixer briefs lack the tooling-names rule: fix as proposed.
- fix-affected-set, MEDIUM, the rule binds plan and discussion fixers' `_lym` files: fix as proposed (same as fix-proposal 2).
- fix-affected-set, MEDIUM, forwarding of the patterns still untested: fix with own change; the two `wiring.go` closures and `beamDescribeDeps` are covered, the `internal/beamcli/deps.go` sites are not (see deferred).
- fix-affected-set, LOW, an unbuildable text does not hold the review open: fix as proposed.
- fix-affected-set, LOW, new review error paths have no rows: fix with own change; an unwritable ready record and a failed reset have rows, a failed stat or write of the sent record has none.
- fix-code-correctness, MEDIUM, a failed reminder send uses up the reminder, and a covering pause is not checked: fix as proposed.
- fix-code-correctness, LOW, no reminder without a notifier or under `QuietHold`: fix as proposed.
  The widening needed one more guard: no reminder while a typed prompt line has started no turn.
- fix-code-correctness, LOW, the review timeout counts pauses and sleep: fix with own change (unbroken waiting).
- fix-code-correctness, LOW, the fresh-spawn reset wiring is untested: fix as proposed.
- fix-code-correctness, LOW, boundary-only empty matches pass `LoadConfig`: fix as proposed.
- fix-proposal, MEDIUM, an undelivered re-prompt loses the findings: fix with own change (`GateResult.Reprompted`).
- fix-proposal, MEDIUM, the tooling rule reaches the run's own records: fix as proposed.
- fix-proposal, MEDIUM, wall-clock review timeout: fix with own change (unbroken waiting).
- fix-proposal, MEDIUM, the reminder can push an agent the operator told to wait: fix with own change; the line now says to do as the operator said.
- fix-proposal, LOW, an unbuildable text keeps the review blocked after a failed edit: dispute, every fix worse (see Disputed).
- fix-proposal, LOW, boundary-only empty matches: fix as proposed.

## Disputed

No review finding is disputed.

One self-check finding is: fix-proposal's "an unbuildable text keeps the review blocked", on the ground `every fix worse`.
Releasing the hold when the text cannot be built would end a review while the pull request holds text known to be stale, which F13 and the fix-affected-set finding on `internal/landingshed/prreview.go` (`syncText`, `judge`) require it not to do.
The build failure is logged, and repairing the description releases the hold.

## Deferred

No review finding is deferred.

From the self-check: the forwarding of `ForbiddenTerms()` at `internal/beamcli/deps.go`'s two recorder closures and its `DescriptionFindings` seam still has no test.
Those closures resolve a real beam pair, so a test needs a hub fixture above tier0.

## Rulings

None reached this round.
