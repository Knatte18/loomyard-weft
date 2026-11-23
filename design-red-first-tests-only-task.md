# Verify gate requires a fail-on-base test for a tests-only task

`red-first-tests-only-task` · Task · bug, gate, darn · running · Drift-Verify

Imported from [issue #184](https://github.com/Knatte18/loomyard/issues/184).

The loom run for a tests-only task halted `blocked` at `Darn-Write` (anomaly `escalation-to-human`) after the gate failed three times.
The gate finding was the same each time:

```
Every test the diff adds or changes passed on the merge base, so none of them shows the bug the fix fixes.

Way forward: make a test fail on the merge base and pass with the fix, reaching the bug through code the base already has, and commit it.
```

The task adds golden tests that pin existing behaviour; it fixes no bug.
Its new tests pass on the merge base by design, so the "fails on base, passes with fix" check can never be satisfied.
The agent had no valid way forward, and the run burned its verify budget and escalated.

What to do:
- Have the verify gate skip the fail-on-base check for a task that is not a bug fix (a tests-only or golden-test task), or let the Sketch or plan declare that kind.
- Reword the finding so it does not tell the agent to invent a bug when the task has none.

Provenance: task `board-readme-golden-tests`, note `loom-halt.md`.
