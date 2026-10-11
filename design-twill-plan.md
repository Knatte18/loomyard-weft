# Twill for Plan-Write: audit and bench

`twill-plan` · Note · enhancement, loom, stencil

## Idea (operator, 2026-10-11)

Audit and bench a twill producer for Plan-Write: the plan writer reviews its own finished plan through lenses, one fork per lens, before it ends, as twill does for Discussion-Write.
Run it like the Discussion bench (`docs/research/discussion-bench.md`, `.scratch/twill-bench/results.md`): driverless Plan-Write runs on a fixed base, graded against a reference run's Plan-Review and Webster findings, with wall time and cost from `tools/tokencount`.

## Seen (2026-10-11)

- Twill exists only for Discussion: `discussion_producer: twill` and the `twill-discussion` fan in `burler.yaml`; the twill directive stencil is already host-neutral.
- Discussion twill lens reviews in `config-layers-run-settings` and `orch-wake-policy` each fixed two BLOCKING findings before parent review.
- Plan defects today often surface first in Webster, where a fix costs most.
- The Discussion bench found twill `opus[medium]` the best value: `opus[high]` twill cost 1.31x for no measurable gain, and the misses that remained came from code not read, not from effort.
  Hub `loom.yaml` runs `plan: opus[high]`, so the Plan bench should run both efforts.

## Bench round 1 (2026-10-11)

Reference `shuttle-startup-ready` (six grading keys); prototype through a pair-local wrapper, no Go change; spend $14.53; full record in `.scratch/twill-plan-bench/results.md`.
Repeated / partly repeated keys: single `opus[high]` 1 / 1 at $2.67, single `opus[medium]` 4 / 0 at $2.77, twill `opus[medium]` 3 / 0 and 1 / 1 at $4.44 and $4.65.
The lenses found real defects, but the writers rejected the ones that mattered, once on a wrong premise, and both spawned their forks later than the directive says.
A round 2 is worth it only with changes: forks at the directive's point, a rejected finding must cite a code read, and a second reference task.

## Open

- Plan lenses: candidates are card coverage of the design, `Edit:` lists against the code, card order and dependencies, tests and the test budget, and PATTERN fit.
- Whether lens review lets Plan-Review run fewer rounds, as the operator chose for Discussion.
