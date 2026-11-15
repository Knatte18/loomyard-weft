# Raddle cuts darn families so more parts run at once

Raddle's split now weighs concurrency, so more darns of a family run at once at no extra price.
The cost report (`docs/research/loom/darn-cost-2026-11-08/report.md`) found the 12 darns of the `fabric-test-speed` split ran about 2.2 at a time, held back by a dependency chain.

The rule has three parts:

- Never add a part to gain concurrency: the part count stays what the work gives.
- Of the cuts with that part count, take the one that lets the most parts run at once:
  a short `depends_on` chain, and parts with no dependency between them touching different packages.
- Never let a part guess at the interface of a part that runs beside it.
  When one part builds what another uses, either chain them with `depends_on`, or fix the interface in both parts' bodies.

The change is prompt text only, in two stencils:

- `contracts/stencils/raddle/raddle-template-agent.md` carries the rule for the agent that writes the proposal.
- `contracts/stencils/loom/loom-rubric-raddle-review.md` gets two judgement bullets, "The cut runs the most parts at once" and "No part guesses at another's interface".
  Its `design` class now names a part added only for concurrency and a cut that runs fewer parts at once than another cut with the same part count.

No code changes.

## Design decisions

- Smaller darns stay: each part keeps its own run and review perches, so the rule never merges parts to cut the fixed cost per family.
- The rule does not touch approval: the operator still approves each raddle proposal in the pane, `raddle_auto_approve` stays off, and `arras` stays unused until the splits are trusted.
- The rubric judges the rule, so the split review flags a cut that breaks it.

## To check by hand

- The rubric's `design` class is open on one finding: it may fault every chain.
  The new wording faults a cut that runs fewer parts at once than another cut with the same part count.
  Check that a chain the work needs, or one that fixes an interface, is not flagged by that wording.
- Run a raddle split on a family with a real dependency and read the proposal and its review.
  Neither stencil has been run against a split.
