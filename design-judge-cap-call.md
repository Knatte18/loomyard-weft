# Judge settles the review budget cap; arbiter only for stuck runs

`judge-cap-call` · Note · enhancement, loom, burler

At a spent review budget the Bouncer escalates, and today the driver hands that stop to the arbiter: a cold `opus[high]` agent, about a minute per spawn.
Whether to run one more Burler round or accept unconverged is not a "totally stuck" stop, so the operator sees the arbiter as the wrong tool for it (2026-10-12): the judge should make that call itself, and the arbiter stays for runs that are stuck.

## Seen

`convergence-plan`, 2026-10-12, Discussion-Bouncer with `review_max_bounces: 2`, arbiter records in `_lyx/shed/convergence-plan/arbiter/` in the run's weft:

- attempt 1 (after round 2): a BLOCKING finding made the design wrong (D10's landing join), the fixer rewrote it, nothing had reviewed the rewrite; verdict `circling continue`.
- attempt 2 (after round 3): every finding fixed as prescribed, none design-wrong; verdict `circling accept`.

## Idea

The judge reads only what Burler wrote, never the code or the fixer's diff.
From the finding text alone it can tag each finding: a mechanical fix that is accepted at the cap, or a design-wrong finding that needs a re-review.
At the cap the Bouncer then continues one round when a design-wrong finding was fixed but not re-reviewed, and accepts unconverged when only mechanical findings remain, with no arbiter spawn.
The arbiter is spawned only when that rule cannot settle the stop.

## Soft cap and hard cap

The operator wants Burler bounded, but a cap of 2 means little when the judge decides past it anyway (2026-10-12).
The orch proposed two caps, and the operator asked to record them here:

- **Soft cap**, today's `review_max_bounces` (for example 2). Up to it, rounds run as today.
  Past it the judge must justify each further round from the finding text:
  - only mechanical findings remain: accept unconverged;
  - a design-wrong finding was fixed but not re-reviewed: one more round;
  - the same blocker returns in that round: circling, stop.
- **Hard cap**, a new setting (for example 4), never exceeded.
  A design-wrong blocker still standing there means the run is stuck: then the arbiter, or the parent.

The soft cap sets the cost of a review that goes well; the hard cap stops loops.
The arbiter is spawned only at the hard cap or on a returning blocker.

## Open

- The hard cap's default and its config key.
- Whether a stuck stop at the hard cap goes to the arbiter or straight to the parent.
