# Twill for Plan-Write at opus[medium]

`twill-plan` · Task · enhancement, loom, stencil · running · Discussion-Burler-Review

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

## Bench round 2 (2026-10-11)

Changes: the directive spliced into the plan prompt before Step 3 so forks run before the plan is written, a rejected BLOCKING or MEDIUM lens finding must cite a code read, and four lenses tuned from round 1 (`criteria-coverage`, `row-reachability`, `forced-edits`, `test-economy`; `card-structure` and `pattern-fit` dropped as noise).
Second reference `run-time-by-role` added; spend $28.82; full record in `.scratch/twill-plan-bench/results.md` (Round 2) and `round-2/`.
Repeated / partly repeated keys, mean of two runs per arm:

| Reference | single `opus[high]` | twill `opus[medium]` |
|---|---|---|
| `shuttle-startup-ready` (lenses tuned on it) | 1.0 / 0.5 | 0.5 / 0.5 |
| `run-time-by-role` | 4.5 / 2.5 | 2.0 / 3.5 |

Twill cost $5.24 per plan against $2.63, about 3.5 minutes more writer time.
All three changes held: every writer forked before writing, every rejection cited code that checks out, and each closed key traces to a named lens finding.
Missed by every arm: a check against the wrong parent directory, and test names with a banned PATTERN token (caught only by the verify gate); a `pattern-scans` lens is the candidate.
The agent recommends building twill for Plan-Write at `opus[medium]` with these four lenses and the cited-rejection rule, then benching whether Plan-Review can run one round.

## Bench round 3: twill `opus[high]` (2026-10-11)

Round 2's setup unchanged, writer at `opus[high]`; spend $23.44; record in `.scratch/twill-plan-bench/results.md` (Round 3) and `round-3/`.

| Reference | single `opus[high]` | twill `opus[medium]` | twill `opus[high]` |
|---|---|---|---|
| `shuttle-startup-ready` | 1.0 / 0.5 | 0.5 / 0.5 | 0.5 / 0.0 |
| `run-time-by-role` | 4.5 / 2.5 | 2.0 / 3.5 | 1.0 / 3.5 |

Twill `opus[high]` averaged $5.86 and 12.5 minutes per plan, 12% and about 2.3 minutes over `opus[medium]`.
The gain on `run-time-by-role` is one key avoided by wording, not by a lens or a read of the vocabulary scan, and lies within the run-to-run spread; the agent recommends `opus[medium]`.

## Bench: twill `sonnet[high]` (2026-10-11, operator's request)

Round 2's setup unchanged, writer at `sonnet[high]`; spend $19.09; record in `.scratch/twill-plan-bench/results-sonnet.md` and `round-sonnet/`.
Repeated / partly repeated: 1.0 / 0.5 on `shuttle-startup-ready` at $4.47 per plan, 2.5 / 2.0 on `run-time-by-role` at $5.08, close to twill `opus[medium]` and within the arm's spread, at about 9% less cost and more writer time.
Behavior: forks spawned on time, only one of four first `validate-plan` passes succeeded, the forks raised more and mostly LOW findings, and two of 18 rejections kept a key defect (one on a wrong, uncited premise), where `opus[medium]` kept none.
The agent finds no reason to prefer it over `opus[medium]`.

## Bench: sonnet[medium] twill and single Fable (2026-10-12, merged from `twill-plan-sonnet-medium`)

Round 2's setup, two runs per reference per arm; record in `.scratch/twill-plan-bench/results-sonnet-medium-fable.md` and `tokens-sonnet-medium-fable.md`.

| Arm | Ref 1 R / P | Ref 2 R / P | Cost per plan | Writer time |
|---|---|---|---|---|
| twill `opus[medium]` (round 2) | 0.5 / 0.5 | 2.0 / 3.5 | $5.24 | 10m10s |
| single `opus[high]` (round 2) | 1.0 / 0.5 | 4.5 / 2.5 | $2.63 | 6m49s |
| twill `sonnet[medium]` | 2.0 / 1.0 | 2.5 / 4.0 | $2.99 | 7m25s |
| single `fable[high]` | 3.5 / 0.5 | 4.0 / 3.0 | $5.67 | 10m8s |

`sonnet[medium]` twill costs 43% less than twill `opus[medium]` but repeats more key defects, and four of its 21 rejections kept a key defect.
Single Fable lands at single `opus[high]` quality or below, at twice its cost.
Neither arm reaches twill `opus[medium]`; an `opus[medium]` writer with `sonnet[medium]` forks is the untested cheaper variant.

## Build (operator, 2026-10-12)

The operator chose to build twill for Plan-Write at `opus[medium]`, as the benches recommend:

- A Plan-Write producer `twill` beside `single`, selected in `loom.yaml` like `discussion_producer`, with the writer at `opus[medium]` and its forks inheriting it.
- The four tuned lenses `criteria-coverage`, `row-reachability`, `forced-edits` and `test-economy`, as in `.scratch/twill-plan-bench/round-2/directive.md` in the hub prime; `card-structure` and `pattern-fit` stay out.
- The directive placed so the lens forks run before the plan is written (round 2's splice before Step 3), and the rule that a rejected BLOCKING or MEDIUM lens finding must cite a code read.
- Instead of a `pattern-scans` lens, `lyx loom validate-plan` checks deterministically what PATTERN's scans would refuse in a plan's named tests and glyphs, starting with banned tokens in test names, the defect every bench arm missed and only the verify gate caught.
- No further bench tuning before the build: the lenses are tuned from Plan-Review and Webster findings a lens should have caught, once twill runs.

## Open

- Whether lens review lets Plan-Review run fewer rounds, as the operator chose for Discussion: once built, measure with a lower `plan_max_bounces`.
- The other miss every arm shared, a check against the wrong parent directory, has no lens or check yet.
