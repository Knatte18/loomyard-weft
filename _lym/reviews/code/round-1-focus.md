---
round: 1
focus:
  - "contracts/stencils/drift/drift-template-baseline.md Step 2 scope list: does it keep `full` for every task whose intended change the suite reaches, keep `--skip` only for tests the task makes meaningless or nondeterministic, and keep `off` for a task the suite cannot reach, as the board entry's Expected section asks? Check that no rule the entry did not ask to change (the reachability check before the list, Step 3's single-call rule, scope_note) is dropped or weakened."
  - "Step 4 wording: does the baseline report instruction have the agent name the tests and lines it expects to drift so Drift-Verify accepts each with a reason and catches any other line that moves? Read contracts/stencils/drift/drift-template-verify.md against it: does the verify stencil's own accept and regression rules still agree with the new baseline wording, or does the diff leave a contradiction there?"
  - "Commit message e12efa51: does its account of the stencil change match what the diff does?"
---

The diff is one stencil file.
The decision record carries only the board entry under `## Goal`, so the entry's Expected section is the measuring stick.
