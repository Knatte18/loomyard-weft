---
round: 1
focus:
  - "internal/burlerengine/parentreview.go `AnswerParentAskOrUnreached`: settle whether an answer to a no-parent or undelivered PR-Review ask is accepted only for the run's latest, uncleared ask, and whether the check and the write happen under the one lock, so a racing expiry or a newer ask cannot leave two end states."
  - "internal/burlerengine/parentreview.go: settle whether a satisfied answer to an ended ask stands at the head that ask named, so `pr-continue` accepts it, and whether a non-PR-Review ask that ended no-parent is still refused."
  - "internal/loomcli/parentreview.go `parentReviewAnswerVerb`: settle whether the verb routes to the new function, keeps its refusal text for the cases still refused, and reports the ended-ask path distinctly from a live answer."
  - "Scope against the board entry (issue #206): settle whether the diff makes the ask resolve the orch's current pane, as the entry's 'Expected' says, or only lets the parent answer a no-parent ask, and whether the unmet part is an unlicensed omission or covered by the entry's wording."
  - "Root-Cause trailer on 4e863bbab: settle whether it names the defect the diff removes."
  - "Contract text in contracts/orchrole/role-autonomy.md and the landing-block-pr-review-autonomous stencil: settle whether the wording matches the behavior the code now has, and whether any existing rule in those texts is dropped or weakened."
  - "Test economy: the report lists `TestBouncer_Call_RefusesARecoveredConvergedOverParentFindings` as contained in `TestBouncer_JudgeCall_AsksTheParentBeforeJudging` and `TestBouncer_Reentry`; compare their assertions before filing anything. Also check that the diff's edits to existing tests in driftengine, landingshed, shedadapters and loomcli loosen no assertion."
---

Seed focus only; the reviewer judges the whole range.
