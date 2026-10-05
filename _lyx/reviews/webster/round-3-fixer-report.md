# Round 3 fixer report

## Fixed

- F1: added the `decision: commit failed after the append` transient row to `contracts/specs/refusal-spec.md`; the refusal and its test already existed.
- F2: `discussionparser.ErrNoDecisionsHeading` replaces the formatted error, `decisionVerb` refuses it with a `way forward:` clause, with a spec row and a `TestDecisionVerb_Refusals` case.
- F3: the Bouncer's unreadable-decision and already-settled Reasons now carry a `way forward:` clause, with two shed spec rows and tests asserting the clause.

## Deferred

None.
