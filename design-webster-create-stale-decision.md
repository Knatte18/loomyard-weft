# webster: Create check ignores a decision that drops the Create; recover-batch advised anyway

`webster-create-stale-decision` · Note · bug, webster

Imported from [issue #171](https://github.com/Knatte18/loomyard/issues/171).

## What happened

Batch 01 of a four-card batch failed `lym webster record-batch 01` and then `lym webster recover-batch 01` with the same `create-not-done[blocking]` finding: a card's Create target did not resolve (quarry answered `not_found`, "the unit exists and the member is missing").
The run's parent had recorded a decision (`lym loom decision add`) that drops that Create, so the member is meant never to exist.
Both refusals still gave the generic way forward: "recover-batch 01 to build the target as the card names it", or amend the card's Create target and run `lym webster rebaseline --card NN`.
The run then halted `blocked` at Webster (anomaly `escalation-to-human`) with the note "the parent's decision drops that Create, so card 1 needs a plan amendment".

## Why it is abnormal

The verifier checks the card's Create target against the worktree and ignores the recorded decision that drops it.
`recover-batch` is advised although it can only rebuild what the card names, which the decision forbids.
The refusal does not tell the operator or parent that a dropped Create needs a card amendment and rebaseline, nor does it say whether the decision record can retire the Create finding.
The result is a halt on a state the parent already resolved.

## What to do

- Make the Create check read the run's decision record and treat a Create dropped by a decision as satisfied, or report it as "dropped by decision" rather than `blocking`.
- When a finding matches a dropped Create, make the way forward name the card amendment plus `lym webster rebaseline --card NN` and not offer `recover-batch` as the first option.

Provenance: task `wemod-invariants`; notes `loom-halt.md`, `webster-refusal-record-batch.md`, `webster-refusal-recover-batch.md`.
