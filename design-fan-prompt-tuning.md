# Tune the Burler lens fan prompts

`fan-prompt-tuning` · Note · enhancement, burler, stencil

## Idea (operator, 2026-10-13)

The Burler lens fan is parked because the fan-lab bench found it no better than a solo round, and its forks found mostly overlapping findings.
The operator thinks the fan could pay with better-tuned prompts: lenses and fork instructions that send each fork after different defects.

## Starting point

- Bench results: `docs/research/lens-fan-lab-phase1.md` ("Conclusion") and `lens-fan-lab-phase0.md`, cost in `lens-fan-cost-analysis.md`.
- On Discussion-Review a `sonnet[medium]` fan missed the BLOCKING defect that solo `opus[medium]` found; a `sonnet[high]` fan found it but passed 300k context and cost more than solo.
- On Plan-Review the `sonnet[high]` fan's surplus was mostly LOW findings, at about six times a solo round's cost.
- Variant D's master manifest (pre-read without judging) is the report's named starting point if the fan is revisited.
- Forks inherit the reviewer session's model (`fan_review`), so master and forks cannot run on different models.

## Scope to discuss

- Lens texts that overlap less: what each lens owns, and what it must leave to the others.
- Measure overlap per lens against the solo round's findings, on the fan-lab fixtures, before any hub setting changes.
- Fan on or off per round (for instance fan in round 1 only) needs code; see `model-downgrade` for per-round models.
