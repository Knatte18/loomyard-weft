# Fixer report — discussion round 2

I checked each finding against the code before fixing it.
`lyx loom validate-discussion` passes on the edited pair.

## Changes

- **F1** (converged carry-over text misstates an every-fix-worse dispute): confirmed.
  `renderCarryOver` in `internal/discussionparser/carryover.go` renders "converged; each finding below was addressed by the round's fixer, and no fresh reviewer has seen the fix".
  `loom-template-plan.md` and `loom-template-rework.md` both say an entry "lists review findings whose fix in the record no fresh reviewer has seen".
  An every-fix-worse dispute makes no fix, so both texts would be false for it.
  I took the review's primary fix, not its fallback of accepting the mislabel as a risk.
  The new D7 rewords the converged closing text to "fixed or disputed … the fixer report says which", along with the `CarryOverConverged` comment, its existing test row and the two stencil sentences.
  Scope In now lists those sites, and Scope Out keeps only the entry's structure, the accepted-circling text and `carriesOver`.
  The changes also cover:
  - D5's opening sentence no longer lists the carry-over text as unchanged, and its closing-round paragraph points at D7.
  - The Out bullet on burler's wiring names the carry-over text, which burler does not own.
  - A new Auto-mode assumption explains why D7 is in scope.
  - Open risks bullet 2 says the entry now points at a fixer report that says the finding was disputed.
  - Two Acceptance criteria cover the new text and the stencil sentences, and the package list gains discussionparser.
  - In the support log, Interview step 4 records the defect, the "Change the carry-over text" rejected alternative becomes the rejected "accept as an open risk" option, and a new ledger entry Q8 records the decision.
- **F2** (D4 credited with a settling effect): confirmed.
  `decisionRuleMarker` rules from class, severity and earlier-open keys alone, so the reviewer's answer is not an input the judge acts on.
  D4's rationale now says every D5 path exists without the rule, and that the answer is for the next fixer, which can concede or dispute again on new evidence, so a persisting dispute is decided on its merits rather than by repetition.
  I made the same change to the D4 Auto-mode assumption, the support log's Interview step 6 and the "Leave the reviewer untouched" rejected alternative.
  All of them now drop the judge as a reader and drop "stays open instead of settling".

## Disputed

None.

## Deferred

None.
