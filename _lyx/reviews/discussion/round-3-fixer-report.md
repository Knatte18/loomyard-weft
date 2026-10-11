# Fixer report — discussion round 3

## Fixed

- **F1 (MEDIUM, consistency).**
  Confirmed against `contracts/stencils/loom/loom-template-discussion.md`: the section's lead-in at line 159 says "the two named in Step 5", line 179 says "Committing your two files", and line 183 says "those two files"; the phrase "the two files" does not occur.
  D2's second bullet now names the two sentences the twill paragraph widens by role, the lead-in's write scope and the do-not-repair rule's "those two files", and states that the commit sentence stays about the two discussion files, since the marker is untracked (D7).
  The fourth Acceptance criterion now names the same two sentences and keeps the commit sentence out.
- **F2 (LOW, design).**
  Confirmed against `pattern/PATTERN-daemon-wakeups.md`: it exempts only "a bounded wait that ends within a minute", so a 250 s capped call is not clearly out of scope.
  D9's poll bullet drops the "not an unbounded wait" claim and states that the poll meets the rule either way: the one-second floor, no backoff because a stat spawns no process and captures no pane, and stat in place of `fswatch` for the stated reason.
  The support log's rejected alternative on the 200 ms poll drops the "does not bind" clause and keeps the floor.

`lyx loom validate-discussion` passes after the edits.

## Disputed

None.

## Deferred

None.
