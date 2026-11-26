# Webster: plan fixes after begun batches, and recovery that ends asking

`webster-plan-recovery` · Task · bug, webster, quarry · starting · Open-Window

Bundled from the notes below; each part keeps its note's text.

# Plan fixes after begun batches block Webster; glyph-not-found on members a card creates

Imported from [issue #221](https://github.com/Knatte18/loomyard/issues/221).

## What happened

Two Webster halts after the plan was edited by a plan review/fix round while batches were already begun.

1. `blocked` / `escalation-to-human`:

```
webster: the auto-rebaseline of the changed plan blocks this run: webster: the edited plan changes the cards of a batch this run already begun: 00-overview.md changed outside its Card Index; it carries the plan's integration verify and only its Card Index is rebaselined; way forward: restore 00-overview.md, or fix the plan: move a done card's edit to a follow-up card after the last begun batch that carries the decision, with its Card Index line in 00-overview.md, then re-step the Webster row
```

2. After the plan was fixed and the run re-stepped, `failed` / `producer-hard-failure`, plan validation refused with four `glyph-not-found` findings, all of the form:

```
glyph-not-found/<card>[blocking]: target "<Unit>#<Member>"'s unit exists but the member is missing — check for a misspelled member
```

The members named were ones the plan's cards (5 and 6) introduce or modify, and the targets include a method signature with a generic parameter type (`IReadOnlyList<double>`) and a nullable (`TimeSpan?`).
Card 5 and card 6 cite the same new member as a target.

## Why abnormal

- The plan review loop (`Plan-Burler-Fix`) edited `00-overview.md` outside its Card Index and the run then blocked on it; the plan-fix stage gives no warning that such an edit is unrecoverable once a batch has begun, so each such fix costs an operator escalation.
- If the glyph targets name members that an earlier card of the same plan creates, validation cannot find them at plan time and refuses a valid plan. If they are misspelled, the finding text does not say which member of the unit is closest. Either way the finding should say whether the member is expected to be created by a card.
  I could not tell from the notes which case applies.

## What to do

- Have the plan-fix stage refuse or warn on edits to `00-overview.md` outside the Card Index once any batch has begun.
- Let glyph validation accept a target that an earlier card of the plan declares as created, and print the unit's actual member names beside a `glyph-not-found` finding.

Provenance: task `models-torque-drag-adapter`; notes `loom-halt-4.md`, `loom-halt-6.md`.


# Webster recovery ending dead with dead_reason asking drops the question and needs a manual reset

Imported from [issue #220](https://github.com/Knatte18/loomyard/issues/220).

## What happened

A Webster batch's verify failed on a test the plan could not satisfy.
The batch recovery session ended `dead` with `dead_reason: asking`, i.e. the recovery agent stopped to ask a question instead of finishing, and `recoveries: 1`, `recovery_retry: false`.
Then:

```
lym webster recover-batch 3 refused:
webster: batch 03-torque-drag-stage recovery (session ...) ended dead and the batch has no recovery left; way forward: end the run stuck, naming batch 03-torque-drag-stage
```

The run halted `blocked` (anomaly `escalation-to-human`).
After the plan was edited and rebaselined, Webster halted `blocked` again:

```
webster: a batch of this run is at a dead end: batch 03-torque-drag-stage is dead and its recovery earns no retry, so a Master would only end the run stuck on it; way forward: 1) lym webster reset --to batch-start --batch 03; 2) re-step the Webster row; or 1) lym webster reset --to start; 2) re-step the Webster row
```

## Why abnormal

A recovery agent that asks a question has no channel that answers it, so the question becomes a dead recovery with no retry.
The agent's question, which would tell the operator what is wrong, is not carried into the halt text: the halt note only restates "dead_reason: asking".
Plan edits made to fix the cause did not clear the dead state either; a manual `webster reset` was needed.

## What to do

- When a recovery ends with `dead_reason: asking`, put the agent's last message in the halt/refusal note.
- Either route the question to the parent (`lym creel post --parent --expect-reply`) or let a plan rebaseline that touches the dead batch's cards re-arm one recovery, so the operator does not need a reset.

Provenance: task `models-torque-drag-adapter`; notes `loom-halt.md`, `loom-halt-5.md`, `webster-refusal-recover-batch.md`.


