# One verb to give a running task an instruction, with no LLM between

`loom-tell` · Note · enhancement, loom, creel

## Seen

Loom runs on the Go driver since `a52467411` (2026-10-15, hub `shed.yaml` `driver_kind: go`): no LLM lives for the whole run, and Go spawns one only when it needs it.
The earlier operator decision that a run's driver takes every downward message for the run, an instruction from the operator or the orch such as a decision about a card, a scope correction or a stop (2026-10-07, confirmed 2026-10-12), so has no receiver.
Today the orch records content with `lyx loom decision add`, stops a run with `lyx batten pause`, and messages whichever agent works at that moment, which it must look up first.
Upward messages, stops and escalations, are settled in `umbrella-daemon` (WS 005) and are not this note's subject.

## Orch recommendation, not a decision (discussed with the operator 2026-10-18)

One `lyx` verb, for example `lyx loom tell <slug> "<text>"`, with no LLM between:

1. It records the text in the run's decision record, always.
2. When an LLM producer of the run is live, it gives that strand one short line saying a new decision is in the decision record; Go knows the current producer and its strand.
3. When only Go runs, recording is enough: a later LLM producer reads the decision record when it starts, and when the work it concerns is already done, the alignment agent at PR-Gate judges the run against the decision record and its reject sends the deviation to PR-Rework.

A stop stays `lyx batten pause`.
A late correction so costs a rework round at PR-Gate instead of an early goto, which is rare and keeps the verb simple.
Creel only carries the line in step 2 and knows nothing of runs; tackler, a short-lived Haiku that gets a stopped step going, is not the receiver, since judging what a decision means for the plan is not its job.

## Dropped

An alias such as `ly:<slug>:active` for the strand that owns the current step, and a queue holding a message for the next LLM producer.
