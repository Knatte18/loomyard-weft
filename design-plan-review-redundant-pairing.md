# Plan reviewer suggests a file-plus-member card pairing that the redundant-file-target gate blocks

`plan-review-redundant-pairing` · Note · bug

Imported from [issue #460](https://github.com/Knatte18/loomyard/issues/460).

**What happened:** A plan-review fix said to keep a file target beside a member target of that file on the same card, on the premise that a file and a member of it on the same card do not overlap.
The plan gate's `redundant-file-target` check blocks exactly that pairing, so applying the fix as written failed validation, and both cards had to drop the file target again.

**Why it is abnormal:** The reviewer proposes a fix the gate rejects, which costs a wasted fix round.
A reviewer with the gate's check list in view would not suggest it.

**What to do:** Give the plan reviewer the gate's check list (at least `redundant-file-target`), or have its fixes validated against the gate before they are handed to the fixing agent.

Provenance: task `webster-dead-ends`, note `burler-plan-r4.md`.
