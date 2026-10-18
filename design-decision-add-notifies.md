# lyx loom decision add notifies the run live LLM producer of the new decision

`decision-add-notifies` · Note · enhancement, loom, creel

## Seen

Loom runs on the Go driver since `a52467411` (2026-10-15, hub `shed.yaml` `driver_kind: go`): no LLM lives for the whole run, and Go spawns one only when it needs it.
The earlier operator decision that a run's driver takes every downward message for the run, an instruction from the operator or the orch such as a decision about a card, a scope correction or a stop (2026-10-07, confirmed 2026-10-12), so has no receiver.
Today the orch records content with `lyx loom decision add <slug> --by parent`, then looks up the strand working at that moment (`lyx reed list`) and messages it; a stop is `lyx batten pause`.
Upward messages, stops and escalations, are settled in `umbrella-daemon` (WS 005) and are not this note's subject.

## Operator decision (2026-10-18)

No new verb: `lyx loom decision add` itself notifies the run, by default.

1. It records the decision in the run's decision record, as today.
2. When an LLM producer of the run is live, it gives that strand one short line saying a new decision is in the decision record; Go knows the current producer and its strand.
3. When only Go runs, recording is enough: a later LLM producer reads the decision record when it starts, and when the work it concerns is already done, the alignment agent at PR-Gate judges the run against the decision record and its reject sends the deviation to PR-Rework.

A stop stays `lyx batten pause`.
A late correction so costs a rework round at PR-Gate instead of an early goto, which is rare and keeps the verb simple.
Creel only carries the line in step 2 and knows nothing of runs; tackler, a short-lived Haiku that gets a stopped step going, is not the receiver, since judging what a decision means for the plan is not its job.

## Dropped

A separate verb such as `lyx loom tell`, an alias such as `ly:<slug>:active` for the strand that owns the current step, and a queue holding a message for the next LLM producer.

## Open

- Whether a flag turns the notice off, for a decision recorded only for the record.
- The orch role's Fixing a run section and the board policy then drop the separate message step.
