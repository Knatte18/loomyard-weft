---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/shedadapters/bouncerdecision.go:170
    summary: The newly enabled decision route accepts held findings without a decision-record ruling.
    fix: "In settleDecisionAccept, require the recorded ruling that settles each held decision finding before accepting the round. Keep the escalation pending when the ruling is absent, and name recording the answer as the way forward. Update TestBouncer_DecisionEscalation so an accept without a ruling cannot call Approve or Commit, while a recorded ruling permits the appropriate accept or continuation. Keep the new recipe switches."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: contracts/stencils/burler/burler-cluster-fan.md:32
    summary: Consolidation instructions overwrite focus origins and remove their changed-file exemption.
    fix: "Preserve origin: focus for focus decision findings in burler-cluster-fan.md, burler-assist-cluster-fan.md and burler-assist-consolidation.md, alongside the existing Webster exception. Make the cluster-origin rule in burler-review-format.md acknowledge that exception too."
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/burler/burler-review-format.md:78
    summary: Focus findings remain subject to changed-hunk restrictions despite the new changed-file exemption.
    fix: "Exempt new BLOCKING decision findings with origin: focus from both the changed-file and changed-hunk restrictions in burler-review-format.md. Carry the same exception into burler-step-review.md, whose prior-finding rules currently require every new incremental finding to sit on a changed hunk."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedadapters/bouncer_escalation_test.go:478
    summary: The below-budget BLOCKING test reaches dropped-key rejection instead of the settlement behavior it claims to test.
    fix: "Retain alpha as a resolved entry in this row's round-2 ledger, since setup writes it into round 1. Assert that the judged outputs remain and the next round's focus exists, so a rejudge cannot satisfy the expected Stuck outcome."
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/loomrecipe/decisionswitches_test.go:27
    summary: The contained recipe-switch test lacks the required retention marker for its distinct assertions.
    fix: "Keep TestDecisionSwitches_SetOnTheRowsThatAskTheParent and add a //testtiming:keep directive directly above its declaration stating that it pins the exact escalation-switch row sets, which TestAmendmentTarget does not assert."
---

### [BLOCKING] A held finding can settle without its recorded ruling

**Location:** internal/shedadapters/bouncerdecision.go:170

**Issue:** Goal 1 requires the parent's decision-record entry before a held finding can pass review.
The recipe switches newly route Code-Review through decision acceptance, but that route checks no such entry.
`circlingVerb` records an accept without checking a ruling.
`settleDecisionAccept` then accepts when `decisionRefusal` is empty.
That refusal excludes decision findings from its BLOCKING checks.
The carry-over reader accepts an empty added-title list too.
The existing `ABlockingDecisionFindingEscalatesWithoutAReJudgeAndItsAcceptIsNotRefused` case demonstrates the path:
it records only an accept and expects Done plus `approve,commit`.
Thus the new escalation stops automatic settlement but does not establish the required ruling prerequisite.

### [MEDIUM] Consolidation discards focus provenance

**Location:** contracts/stencils/burler/burler-cluster-fan.md:32

**Issue:** The new exemption in `roundCheck.newFindings` requires the exact focus origin.
The cluster and assistant-cluster instructions assign lens or handler origins, preserving only Webster origins.
Assistant consolidation similarly assigns an assistant origin with only the Webster exception.
An undriven focus finding outside changed files therefore loses its exemption when these instructions are followed.
The hardener supports a reviewer fan, so this is a reachable consumer of the new contract.

### [MEDIUM] The unchanged-source case still has contradictory instructions

**Location:** contracts/stencils/burler/burler-review-format.md:78

**Issue:** The format exempts focus findings from its file rule but still requires a changed hunk.
`burler-step-review.md:53` independently requires every new incremental finding to sit on a changed hunk.
It contains no focus exception.
The hardener rubric now requires an undriven entry's finding at the module source it concerns.
For unchanged source, these instructions cannot all be followed.
This is the incremental case Goal 3 asks the diff to admit.

### [MEDIUM] The below-budget regression row tests the wrong path

**Location:** internal/shedadapters/bouncer_escalation_test.go:478

**Issue:** This row's ledger contains only `hard`, while fixture setup writes `alpha` into the prior ledger.
`settleUnlessRefused` detects the dropped key and rejudges before `settleUndecided` runs.
Both paths return Stuck without an escalation, which satisfies the row's assertions.
The test therefore passes without checking the below-budget behavior its name promises.

### [LOW] Retain the distinct switch assertions explicitly

**Location:** internal/loomrecipe/decisionswitches_test.go:27

**Issue:** The supplied redundancy report marks this test as contained by `TestAmendmentTarget`.
Their assertions differ:
the switch test checks exact escalation-switch row sets, while the covering test checks amendment targets.
The rubric requires a retention directive for this case, rather than deleting the test.

## Rejected

- No arbiter eligibility edit is needed: `arbiterstop.Eligibility` already admits decision escalations.
  The arbiter stencil already requires recorded answers and hands unresolved questions onward.
- Removing the BLOCKING path from `TestBouncer_SoftCapRule` matches the new control flow.
  The Loosened-Check trailer identifies the removed path and its replacement escalation assertions.
- The recipe-switch test is not redundant in its assertions; retain it as F5 specifies.
- No unrequested change group was found; the scope lens maps each group to the board entry's three goals.

## Validation

The review covers committed range `30ed1ee588023fba05fb5eab3c24979eda94d479..f2cc07498a2df82e342f7f3b33cd11bd309e7597`.
The supplied diff pieces, commit trailers, board entry, decision record and focus directive were considered.
Budget settlement precedes both automatic acceptance routes when BlockingEscalates applies.
Existing escalation records retain their cause, and malformed records do not accept.
The focus exception retains the BLOCKING and decision-class checks.
The Root-Cause trailer describes the routing and hardener defects addressed by the diff;
F1 identifies a remaining acceptance prerequisite.

The requested tier-0 gate run could not start:
the sandbox made the hub's gate-holder and run-process registries read-only.
No tests ran in that attempt.
Test conclusions above come from source inspection and the supplied gate reports, not a fresh passing run.
