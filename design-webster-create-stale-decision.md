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


## From issue #178

Imported from [issue #178](https://github.com/Knatte18/loomyard/issues/178).

What happened: Webster halted `blocked` (anomaly `escalation-to-human`) on the first batch.
`lym webster record-batch 01` and then `lym webster recover-batch 01` both refused with `batch_failed` and `create-not-done[blocking]`: a card's Create target still did not resolve, and quarry answered `not_found` (the unit exists, the member is missing).
The halt message says the batch's fork had reverted the work under a decision the parent had since superseded; the parent's later decision (#2210) reverses it.
So the fork built against a stale decision although a newer one was on record.

Two lym defects show here:
1. The implementer fork was not given, or did not weigh, a later parent decision that supersedes an earlier one.
   Decision precedence (the later entry wins) does not reach the fork, so the batch fails and the run escalates to the operator.
2. The two refusals give different way-forward text for the same finding.
   `record-batch` offers two paths: `recover-batch` to build the target as the card names it, or amend the card, run `lym webster rebaseline --card NN`, then `recover-batch`.
   `recover-batch` offers only the amend-and-rebaseline path.
   Here the right fix was to build the target as the card names it, which `recover-batch` does not offer; the missing option is misleading for a re-run that is itself the recovery.

What to do: make the batch fork's prompt carry the decision record with later entries marked as overriding earlier ones, and have the batch check flag a fork whose output contradicts a superseding decision.
Make `recover-batch` list the same way-forward options as `record-batch`, or say why it omits the build-as-named path.

Provenance: task single-repo-namespace-layout; notes loom-halt-2.md, webster-refusal-record-batch.md, webster-refusal-recover-batch.md.