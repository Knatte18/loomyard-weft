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
