---
verdict: CONTINUE
rationale: "Round 1 carries BLOCKING F1 (D6 names recover-batch for a dead fork record that refuses at the cap, a PATTERN-refusal-way-forward violation) and design-class MEDIUM F3 and F4; no relabels."
---

Gating findings in the latest round:

- F1, BLOCKING, design: D6 keys on record kind, but recover-batch decides on count and amendment first. Its named way forward refuses at the cap, which `PATTERN-refusal-way-forward` forbids, so the finding stays BLOCKING.
- F3, MEDIUM, design: D3's only guard has no Acceptance row, and its fallback home cannot see the real command tree.
- F4, MEDIUM, design: D2's glob bound assumes bash and leaves unquoted parentheses unchecked.

F2 is MEDIUM but `consistency`, so it is not gating; it is still fixed.
F5 to F9 are non-gating.
No finding was relabeled, and the review has no `## Focus departures` section.
No ledger key recurs, since this is the first round.
