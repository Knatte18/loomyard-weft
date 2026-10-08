# Review judge accepts an APPROVED round with nothing blocking instead of circling to the budget

`judge-accept-approved` · Note · enhancement, burler

## What happened

On 2026-10-08 the Discussion review segments of `plan-glyph-precision`, `shuttle-state` and `reed-view` all ran their five rounds and escalated to the parent with cause `budget`.
In each, the round-5 reviewer's verdict was APPROVED with no BLOCKING finding, every earlier key was resolved, and each round raised fresh keys on edge cases rather than repeating old ones.
The judge said CONTINUE only because one or more MEDIUM `design` findings were open.
The parent then recorded the remaining design calls as decisions and accepted, which another round would not have changed.

## Proposal

The judge accepts a round whose reviewer verdict is APPROVED with no BLOCKING finding, handing open MEDIUM and LOW findings to the next producer through the record instead of starting another round.
A MEDIUM `design` finding that needs a call the producer cannot make escalates to the parent at once, without spending the rest of the budget.
This saves one to several rounds per review segment and one parent escalation per run.

The rule's exact threshold (for example, whether an open MEDIUM `design` finding on a new key blocks acceptance) is a design call for the task that builds it.
