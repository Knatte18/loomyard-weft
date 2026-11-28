# Reviews and plans check their claims against the code first

Review findings and plan cards made claims that reading the code would have disproved:
a fix that named test infrastructure the repository lacks, a fix that reused an identifier invisible from the test's package,
a fix resting on "this test only reads" when a callee writes, and plan cards wrong about who reads a field or what a package can reach.
A plan also split a new exported function from its first caller across cards, which the reachability guard on production functions refuses in between.
This change makes the review and plan prompts check those facts first.

Closes #285
Closes #288
Closes #290
Closes #291

## What changes

- `contracts/stencils/burler/burler-review-format.md`: before writing a `fix`, the reviewer reads each fact it rests on:
  the package clause of the edited file and the export status of any identifier the fix reuses,
  every function on the call chain behind a claim that code only reads,
  and that any kit, recording, stand-in or test tier the fix names exists and can run the tests moved onto it.
  A fix the code disproves is never written: the reviewer proposes one that holds without the fact and names what is missing,
  or, when none exists, raises a `decision` finding offering to accept the gap or build what is missing.
  A fact the reviewer could not establish is named in the `fix` as unverified.
- `contracts/stencils/burler/burler-step-fix.md`: the fixer reads a fact a `fix` names as unverified before applying the change,
  and fixes with its own change or defers when the code disproves it.
- `contracts/stencils/loom/loom-block-plan-cards.md` (the plan writer) and `contracts/stencils/loom/loom-rubric-plan-review.md` (plan review):
  - each card's claims are read in the code: readers and writers through the call chain, export status and visibility from the using package,
    whether a test or seam can reach a member, and which input reaches a branch past the checks before it;
  - cards are checked against each other for contradicting rules, and for a claim another card's change makes false;
  - a new production function goes on the card that adds its first production caller; a test-only caller does not count.
- `contracts/specs/loom-plan-spec.md`: the card-granularity rule that gives a public-surface symbol its own card now carries that first-caller exception.

## Design decisions

- The fix checks live in the review-file format, which every review round reads, rather than in one rubric,
  so code, plan and discussion reviews all check their fixes the same way.
- For a split between a new function and its first caller, the function moves onto the caller's card.
  The alternative, a defined way for a card to carry a temporary reachability allowlist entry and a later card to retire it, is not built:
  it adds a mechanism and a stale-entry failure to every plan that uses it, where moving the function needs neither.
- The first-caller rule is an exception to the existing granularity rule, stated beside it in the spec, the writer block and the review rubric,
  so the three do not contradict each other.
- No test pins the new wording: the change is prompt prose, and the repository's test-economy rules forbid tests asserting a stencil's static prose.

## Merge danger

**Door:** two-way
**Blast radius:** stencils
