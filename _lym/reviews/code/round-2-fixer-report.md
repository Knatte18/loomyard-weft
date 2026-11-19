---
dispositions:
  - id: F15
    disposition: fixed
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
---

## Changes

- F15, `internal/shedadapters/bouncer.go` and `parentseat.go`:
  `settleUnlessRefused` now settles the round's before-judge ask and applies `parentFindingsRefusal` beside `judgmentRefusal`, after `routeDecisions`.
  The attach harvest, the `entryHarvest` replay and `judgeCall`'s fresh harvest therefore all refuse a CONVERGED verdict over the parent's findings.
  `judgeCall`'s own settle and check are gone.
  A new `parentAsker.heldBeforeJudgeAsk` returns the number of the ask the round holds under its generation seed, 0 for a seat that does not ask before the judge, and sends nothing.
  `Bouncer.parentAsker` builds the asker for both call sites.
  `TestBouncer_Call_RefusesARecoveredConvergedOverParentFindings` lays a CONVERGED verdict and ledger on disk with an unchanged subject and a done ask whose report holds findings.
  It asserts Stuck with the refusal reason, the archived verdict, no commit and no judge spawn.
  The recorded-verdict replay at `bouncer.go:528` is reached by a CONVERGED verdict only while a decision escalation is open, which `routeDecisions` takes first, so the entry-harvest path is the one the test drives.
- F16: the text of `judgeParentBlock` moved to `contracts/stencils/bouncer/bouncer-fragment-parent-review.md`, registered in `stencils.go` (embed variable and registry entry) and `markers_test.go`.
  `judgeCall` reads it through `stencilstore` in the new `Bouncer.renderParentReviewBlock`, and a read or fill failure degrades through `renderFailureReason`.
  The constant is deleted.
  The assembled prompt is byte-for-byte what it was, so `TestBouncer_JudgeCall_AsksTheParentBeforeJudging`'s heading and await-verb checks stand unchanged.
  `stencilstore.Read` takes no engine, so the fragment is read from the Bouncer's stencils directory like the other bouncer fragments.
- F17, `internal/driftengine/verify.go`: `p.judgeStartedAt = p.now()` moved from `judge` to the top of `runAgent`, which the line-mode and command-mode judges both call before the table is built.
  `TestDriftVerify_RunAgentCapsTheParentAskByTheRunsStart` runs `runAgent` over a fake seat runner that evaluates the table's agreement entry in its run.
  It asserts the recorded ask's deadline is the clock's reading plus `judgeTimeout` less `judgeParentMargin`, with `judge_parent_timeout_min` set to twice the judge timeout.

Commits: F17 is `5870902a`.
F15 and F16 share one commit, `c3ca6d45`, because both rewrite the same hunks of `bouncer.go` and `parentseat.go` and the environment has no interactive staging to split them.

Gate repair: the comment-style gate refused comment lines this round added, wrapped mid-sentence or 120 characters and over.
They are now split one sentence per line in `bouncer.go`, `parentseat.go`, `bouncer_parent_test.go` and `judgeparent_test.go`.
A second pass rewrote the doc comment of `TestDriftVerify_RunAgentCapsTheParentAskByTheRunsStart` as three whole sentences, one per line.
No code changed, and each repair is its own commit.

Gate: `lym gate test ./internal/shedadapters ./internal/driftengine ./contracts/stencils` passed, tier 0, 1728 targets, 0 failed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
