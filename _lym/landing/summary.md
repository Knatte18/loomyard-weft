# Darn writers and the sampler agent commit after each step

The darn writer and the sampler agent are now told never to hold their work uncommitted until the end.
A stall, a dead session or a reboot then loses at most one step.
Before this, a darn writer could work through a multi-step change without committing, and a dead one left it uncommitted.

- The darn writer's Step 4 list has it commit after each coherent step, so the parent also sees progress in the branch's history.
- The sampler agent keeps its one commit per settled step, and its line now says why.

Closes #246

## Design decisions

- The two darn fragments time the commit differently.
  `darn-fragment-make-change` commits a step once its gate passes, and each commit leaves the tree green.
  `darn-fragment-make-change-gated` commits as soon as a step's edits are made and asks for no green tree,
  since a cold fixer clears that row's verify gate.
- The change is stencil text only; no Go code enforces the commit cadence.

## Merge danger

**Door:** two-way
**Blast radius:** stencils
