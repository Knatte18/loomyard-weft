# Darn writers and the sampler agent commit after each step

A darn writer could work through a multi-step change and hold it all uncommitted until the end.
When such a session stalls, dies or loses its machine to a reboot, the uncommitted work is lost with it, and until then the parent sees no progress in the branch's history.

## What changes

- `contracts/stencils/darn/darn-fragment-make-change.md` (verify gate without a cold fixer): step 6 now tells the writer to commit after each coherent step, as soon as that step's gate passes, each commit leaving the tree green, and never to hold the change uncommitted until the end.
- `contracts/stencils/darn/darn-fragment-make-change-gated.md` (verify gate with a cold fixer, where the writer runs no gate): the same rule, with the commit following the step's edits, since this writer has no gate of its own to wait for.
- `contracts/stencils/sampler/sampler-template-agent.md`: the sampler agent's "commit as you go" line now forbids holding the work uncommitted until the end.

Each line gives its reason: a stall, a dead session or a reboot then loses at most one step.

## Design decisions

- The Weaver batch implementer already commits once per card, and the landing fixer, the gate fixers and the verify-fix strands commit once per fix, so their stencils are unchanged.
  The raddle agent commits nothing, and none of the `plugins/ly` skills instruct a writer, so no skill changes either.
- The change is wording only, in stencils the hub reads at call time through `stencilstore`, so no Go code, test or doc describes the changed lines.

## Merge danger

**Door:** two-way
**Blast radius:** stencils
