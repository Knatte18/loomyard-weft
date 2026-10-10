---
verdict: CONTINUE
rationale: "Round 3 carries BLOCKING F1 (design), a new key not open in any earlier round, so the loop continues; F2 is a new LOW consistency key and gates nothing. No finding maps to an earlier open key, so this is not circling."
---

F1 is a strand-table read failure that reaches `preMerge.clear` as a stop error with an empty guid.
It is new in round 3 and unrelated to round 2's `abort-failure-reason-has-no-test-row`, which concerned the `AbortMerge` exit that round 3's focus directive 1 confirmed is now covered.
F2 is about where the parent-branch read sits in `refuseOverUnfinishedMerge`; round 2's `parent-branch-read-error-has-no-test-row` concerned a missing test row, which directive 2 confirmed is fixed, so F2 is a distinct key.
No relabels. There are no focus departures.
