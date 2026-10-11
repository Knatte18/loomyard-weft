---
round: 2
focus:
  - "In _lyx/discussion/decision-record.md D11 and D4's fix-gate-exhausted bullet: when a fix gate is exhausted after the Bouncer's budget is spent, does the Decision bound auto-accept so it cannot accept a target whose gate failed every attempt, and does an acceptance criterion pin that case? Also check whether D4 or D10 states what a gate-exhausted round does under a budget of 1."
  - "In _lyx/discussion/decision-record.md D3 and D4: for each archive exit (orphan review, reviewer gate failure, fix gate exhaustion, review changed after acceptance), does a Decision state whether the acceptance record is archived with the round's review and fixer report, so a resume cannot skip the reviewer on a stale record or wedge on a digest mismatch?"
  - "In _lyx/discussion/decision-record.md D4's Bound line for skipping the fixer on an empty review: does the Decision state what makes a zero-findings count trustworthy when the review parser decodes frontmatter without known-field checks, so a misnamed findings key cannot skip the fixer, and does a criterion pin it?"
---

All three entries target the attack-surface question: what each new skip or auto-accept can let through, and what bounds it.
None falls under the rubric's `Do not flag` list or contradicts the discussion writer's stencil.
