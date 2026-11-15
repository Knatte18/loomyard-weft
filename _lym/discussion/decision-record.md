## Goal

```markdown
## Seen (darn-cost-measure, 2026-11-08)

`docs/research/loom/darn-cost-2026-11-08/report.md`:

- A darn costs about $4.48 fixed plus $1.43 per 1000 changed lines (r 0.31 over 279 darns).
- The 12 darns of the `fabric-test-speed` raddle split ran 359 awake minutes in 163 calendar minutes, about 2.2 at a time, held back by a dependency chain.

## Decisions (operator, 2026-11-08 and 2026-11-11)

- Smaller darns stay: each part gets its own run with its own review perches, and a lower fixed cost per family is no reason to cut larger parts.
- A large change runs as `arras`, then a project of darns.
- The operator approves each raddle proposal in the pane; `raddle_auto_approve` stays off and `arras` is not used until the splits are trusted.

## Change (operator, 2026-11-15)

Add one rule on concurrency to raddle's split, so more darns of a family run at once, at no extra price:

- Never add a part to gain concurrency: the part count stays what the work gives.
- Of the cuts with that part count, take the one that lets the most parts run at once: a short `depends_on` chain, and parts with no dependency between them touching different packages.
- Never let a part guess at another running part's interface: when one part builds what another uses, either chain them, or fix the interface in both parts' bodies.

Put the rule in raddle's agent stencil (`contracts/stencils/raddle/raddle-template-agent.md`) and in the split review's rubric (`contracts/stencils/loom-rubric-raddle-review.md`), so the split perch judges it too.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - design-class-faults-every-chain (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
