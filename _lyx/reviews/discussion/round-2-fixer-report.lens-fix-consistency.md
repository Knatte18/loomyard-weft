```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-2 change set agrees with the sibling text it touches, and find any change whose twin was left as it was.

**LOW: D4's caller-capture sentence was left at `Drift-Await` only.**
- Changed text: D3 now says darn's failed-capture Stuck names "`lyx drift capture` followed by `lyx loom resume`". The re-entry paragraph says a lock-holding capture "is waited on in darn".
- Problem: the twin sentence in D4 still reads "Run by a caller at a `Drift-Await` halt, `lyx drift capture` starts the same detached, registered child…". So it is never said whether a caller's capture at darn's `Drift-Baseline` halt runs detached or in the foreground.
- Evidence: D4's main-capture rule admits "`Drift-Baseline`, or `Drift-Await` while the run is halted there". The detached-start sentence names only `Drift-Await`.

**LOW: D2's new reroute sentence ignores a waive.**
- Changed text: D2 says "A loom Finalize reroute re-enters `Drift-Verify` through `Parent-Align` and the Code perch and compares again".
- Problem: D6 now says that after `lyx drift waive`, "on every later entry `Drift-Verify` returns Done without a compare". So a rerouted run that was waived is not compared again.
- Evidence: the D2 reroute sentence and D6's waive paragraph, quoted above.

**LOW: two support-log lines still point at text this round removed.**
- Changed text: F14 dropped the first Open risks bullet and rewrote Q1. F5 removed D2's ungated `Drift-Await` paragraph and marked Q6 superseded.
- Problem: Interview turn 3 still says "goal-scope 1, board-fit 1 … already under Open risks for the parent; no change". It also says "goal-scope 3: Drift-Await's ungated departure unnamed — fixed (D2)". Both now point at text that no longer exists, unlike the lines marked "superseded in review round 1".
- Evidence: support-log.md Interview turn 3, the goal-scope 1 and goal-scope 3 bullets.

**LOW: the new refusals name no way forward.**
- Changed text: D6's `lyx drift waive` refusal ("refuses unless the run is halted at `Drift-Verify` with no passed capture record"). Also D7's widened `lyx drift accept` refusal of a withdrawn test, from a caller outside a decision halt.
- Problem: the untouched Constraint says "Every refusal and halt names its way forward". The waive refusal states no way forward. The withdrawn-test refusal names none either. Only the scope-`full` accept refusal names one ("naming the parent's ruling").
- Evidence: the Constraints bullet "Every refusal and halt names its way forward and a test reaches each", against the D6 and D7 text quoted above.

No further tool calls: everything needed was already in context.
