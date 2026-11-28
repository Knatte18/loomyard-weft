# Check review fixes and plan card claims against the code

Reviews proposed fixes that rested on facts the code disproves,
such as a test kit that does not exist, a "read-only" test that writes, or an identifier the target file cannot see.
Plans likewise passed review with claims about readers, visibility and reachability that were false,
or that another card contradicted.
This change adds stencil instructions to check those facts. It changes no Go code.

## What changes

- **Review findings** (`burler-review-format.md`): before writing a `fix`, the reviewer reads the facts it rests on:
  - for a reused identifier, the package clause of the file the `fix` edits, and whether that package can see it;
  - every function on the call chain, when the claim is that code only reads;
  - that any named test infrastructure (kit, recording, stand-in, test tier) exists and can run the tests moved onto it.
- **A `fix` may not rest on a fact the code disproves.**
  The reviewer proposes a fix that holds without it and names what is missing.
  When there is none, the finding is raised as class `decision`: accept the gap, or build what is missing.
  A fact the reviewer could not establish is named in the `fix` as unverified.
- **Fixing** (`burler-step-fix.md`): the fixer reads an unverified fact before applying the change,
  then gives the finding its usual disposition.
- **Plan writing** (`loom-block-plan-cards.md`): each claim a card makes about the code is one the writer read there,
  and no card contradicts, or is falsified by, another card.
- **Plan review** (`loom-rubric-plan-review.md`): the reviewer checks each card's claims against the code
  and each card against every other.

Closes #285
Closes #288
Closes #291
Closes #290

## Design decisions

- A carried finding keeps its class, so it never becomes `decision`; its `fix` names both choices instead.
- A `fix` that carries another reviewer's change, which may not be rejected (a kept parent finding's),
  is the one exception to the disproved-fact rule.
  It keeps the change and the class, and appends the disproved or unverified fact.
- The fixer gets no separate rule for a disproved unverified fact.
  The ordered dispositions apply, so a disproved premise can be disputed.
- #290 asked for a new production function to stay with its first caller, or for a sanctioned temporary allowlist entry.
  Neither rule is added.
  The reachability guard it hit is already limited to the upkeep run (commit `2326ef4da`),
  and `PATTERN-test-only-code` allows a card to add a function that only a later card calls.
  The message of commit `e489adf46` still describes a first-caller rule that a later commit on this branch removes.

## To check by hand

- No test asserts this stencil wording, and its effect on a live review or plan has not been tried.
- In `contracts/stencils/burler/burler-step-fix.md`, check that the line on an unverified fact
  leaves Dispute available when the code disproves that fact.
- In `contracts/stencils/burler/burler-review-format.md`, check that the exception bullet
  lets a kept parent finding carry the parent's change without breaking the ban on a `fix` resting on a disproved fact.

## Merge danger

**Door:** two-way
**Blast radius:** stencils
