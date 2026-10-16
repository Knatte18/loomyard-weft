---
round: 1
focus:
  - "Review range and `fans_recorded`: check that the fix half, a resumed reviewer and the Bouncer read a round's fan names only through the recorded round fans and gate on `fans_recorded`, never on the `fan` key, and that `recordRoundFans` is the only writer, called before the spawn."
  - "`fan_review` retirement in `loom.yaml` and `configreg`: check that no reader, default, schema entry or test still references the retired key, and that the per-round fan and fixer fan lists default to today's behaviour."
  - "Byte identity of solo and single rounds after the reviewer prose moves into stencils: check that the diff leaves those renders unchanged when the new marker is absent, and that fanned and twill rounds keep the lens order."
  - "Twill fixer engine: check that the fork self-check part, the fork audit and the refusal naming its keys match the plan card's `Intent`, and that `lyx burler run` reads `fix-fan` from its profile as card 4 states."
  - "Bouncer `exclude_lenses`: check that the judge asks for exclusions only when the next round runs the judged round's fan, in both the same-fan and fan-change directions."
  - "Per-card mechanical checks and test economy: confirm each card group's type-specific check ran against every target, and that new tests neither duplicate existing coverage nor pin a stencil's or lens's static prose."
---

Seed focus drawn from the plan's Card Index and Shared Decisions; no round has been judged yet.
