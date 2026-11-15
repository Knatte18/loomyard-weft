# Raddle cuts darn families to run the most parts at once

Raddle's split gains one rule on concurrency, so more darns of a family run at once at no extra price.
The raddle agent is told it, and the split perch judges it.

## The rule

- Never add a part to gain concurrency: the part count stays what the work gives.
- Of the cuts with that part count, take the one that lets the most parts run at once:
  a short `depends_on` chain, and parts with no dependency between them touching different packages.
- Never let a part guess at the interface of a part that runs beside it:
  when one part builds what another uses, either chain them with `depends_on` or fix the interface in both parts' bodies.

## Where it lands

- `contracts/stencils/raddle/raddle-template-agent.md`: Step 3, "Weigh the split", carries the three bullets.
- `contracts/stencils/loom/loom-rubric-raddle-review.md`: "Judge the split" gains two judgements,
  "The cut runs the most parts at once" and "No part guesses at another's interface",
  and the `design` finding class now names a cut that runs fewer parts at once than its part count allows.

## Design decisions

- The rubric does not repeat the different-packages half of the rule as a judgement:
  the `raddle` gate already fails two unordered parts that share a package (`packages-shared`),
  and the rubric tells the reviewer never to re-derive the gate's checks.
- The split stays operator-approved and smaller darns stay; the rule changes which cut is chosen, never how many parts it has.
