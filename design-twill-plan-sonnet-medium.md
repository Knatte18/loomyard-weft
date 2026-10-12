# Twill-plan bench: sonnet[medium] twill and single Fable arms

`twill-plan-sonnet-medium` · Note · enhancement, loom, stencil

## Idea (operator, 2026-10-11)

Run one more arm of the `twill-plan` bench: twill with the plan writer on `sonnet[medium]`, planned for 2026-10-12.

Second arm, same setup (operator, 2026-10-11): a single Fable 5.1 plan writer without twill, against twill `opus[medium]`.
At list prices Fable costs 2.5x Opus on output and cache writes and 1.25x on cache hits; on the token volume of twill `opus[medium]` it would cost about $9.60 per plan against $4.90, and on that of single `opus[high]` about $5.50-6.50.
The question is whether a single Fable writer finds as much as twill `opus[medium]` with fewer turns and no forks.
Take the Fable model key from the hub's models config; raise the spend cap for this arm accordingly.

## Setup

Exactly the round-2 setup of `twill-plan` (directive before Step 3, the four tuned lenses, cited rejections, references `shuttle-startup-ready` and `run-time-by-role`, two runs per reference), graded against the same keys and costed with `tools/tokencount`, so it compares with the four arms already run.
Records: `.scratch/twill-plan-bench/results.md`, `results-sonnet.md`, `tokens-*.md`; a fresh agent runs it, spend cap about $20.

## Seen so far

- Twill `sonnet[high]` matched twill `opus[medium]` within spread but cost only 9% less: per plan it used about 1.3x the output and 1.5x the cache reads, from more `validate-plan` passes (one of four first passes succeeded) and more, mostly LOW, fork findings.
- Two of its 18 rejections kept a key defect; `opus[medium]` kept none.
- The open question is whether medium effort cuts the token volume enough to pay off, or worsens the judgement and the `validate-plan` retries.

## Bench: sonnet[medium] twill and single Fable

Run 2026-10-12 on round 2's setup, two runs per reference per arm; records in `.scratch/twill-plan-bench/results-sonnet-medium-fable.md` and `tokens-sonnet-medium-fable.md`.

| Arm | Ref 1 R / P | Ref 2 R / P | Cost per plan | Writer time |
|---|---|---|---|---|
| twill `opus[medium]` (round 2) | 0.5 / 0.5 | 2.0 / 3.5 | $5.24 | 10m10s |
| single `opus[high]` (round 2) | 1.0 / 0.5 | 4.5 / 2.5 | $2.63 | 6m49s |
| twill `sonnet[medium]` | 2.0 / 1.0 | 2.5 / 4.0 | $2.99 | 7m25s |
| single `fable[high]` | 3.5 / 0.5 | 4.0 / 3.0 | $5.67 | 10m8s |

- Medium effort pays off on cost: `sonnet[medium]` twill costs 43% less than twill `opus[medium]`, but repeats more key defects on both references, and four of its 21 rejections kept a key defect.
- Single Fable lands at single `opus[high]` quality on reference 2 and below it on reference 1 (no start-level test in either plan), for 8% more than twill `opus[medium]`: fewer turns (18-21 messages) with half the cache reads, but no more findings.
- Recommendation: keep twill `opus[medium]` for Plan-Write; neither arm reaches its quality.
  An `opus[medium]` writer with `sonnet[medium]` forks is the untested cheaper variant.
- Spend: $11.98 sonnet arm, $22.66 Fable arm; all eight pairs and notes torn down.
