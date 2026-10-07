# An operator decision can amend the card of a begun Webster batch

`webster-amend-begun-card` · Note · bug, webster

An operator decision made while Webster runs cannot reach the plan card of a batch that has begun.
Seen 2026-10-07 in `bugfix-sessions`: the operator decided that the submit settle of card 1 comes from config, while batch 1 was running.
The decision went into the decision record, but `lyx webster rebaseline --card 01` refused ("the edited plan changes the cards of a batch this run already begun"), and its way forward, `lyx webster reset --to start` then `lyx webster run --fresh`, has no path through loom: the Webster producer always runs with `Fresh: false` (`internal/shedadapters/webster.go`).
An edit left on disk while the master recorded the batch blocked the run with `plan_drifted` until the plan was restored.
So the card still says "hard-coded constant" while the code reads config, and the Webster review sees a deviation the operator approved.

Wanted (operator, 2026-10-07): an operator decision taken mid-Webster lands in the plan and the decision record together, without extra review or PR-Gate steps, whether or not the card's batch has begun.


## From issue #413

Imported from [issue #413](https://github.com/Knatte18/loomyard/issues/413).

## What happened

A loom run was sent back to `Plan-Write` by a `goto` after the `Batchifier` had already handed batch 1 to `Webster` and the batch's card had been begun.
`Plan-Write`, `Plan-Bouncer`, `Plan-Burler` and `Batchifier` then completed again with an edited plan, and the run reached `Webster` again.
`lyx webster record-batch 1` refused with `plan_drifted`:

```
webster: on-disk plan fingerprint does not match this run's recorded state: ...; the plan changed since state.json was created; way forward: run `lyx webster rebaseline --card 01` to accept the edit, or restore the plan the run recorded with "lyx webster restore-plan", otherwise 1) lyx webster reset --to start; 2) lyx webster run --fresh
```

`lyx webster rebaseline --card=1` then refused as well:

```
webster: the edited plan changes the cards of a batch this run already begun: batch 1 card 01-paced-text-submit changed since it was begun; way forward: restore those cards in the plan, or 1) lyx webster reset --to start; 2) lyx webster run --fresh
```

The loom halted `blocked` at `Webster` with anomaly `escalation-to-human`.

## Why this is abnormal

- The loom recipe itself re-ran the plan stages after Webster state existed, so the drift was produced by lyx's own flow, not by a stray manual edit.
- The first refusal names `rebaseline` as the way forward, but for a plan that edits an already-begun card `rebaseline` can never succeed.
  The first refusal's advice leads straight into the second refusal.
- The only exits left are `lyx webster reset --to start` and `lyx webster run --fresh`, which discard the run's Webster progress.
  Nothing in the first refusal says that the rebaseline path is closed for this case.

## What to do

- Make the loom `goto` back to `Plan-Write` reset or rebaseline Webster state as part of the loop-back when batches were already begun, or refuse the `goto` with a clear reason.
- Have `lyx webster record-batch` check whether `rebaseline` would be accepted and only offer it as a way forward when it would; otherwise name the reset path first.

Provenance: task `bugfix-sessions`; notes `loom-halt.md`, `webster-refusal-record-batch.md`, `webster-refusal-rebaseline.md`.

Correction to issue #413 (orch, 2026-10-07): the reflection read the drift as produced by the goto to Plan-Write; the drift came from the orch's hand edit of card 1 while batch 1 was recorded, and the goto had happened before this Webster run began.
Its finding stands: `record-batch` offers `rebaseline` as the way forward for a begun card, where `rebaseline` always refuses, so the advice should name only paths that can succeed.


## From issue #416

Imported from [issue #416](https://github.com/Knatte18/loomyard/issues/416).

## What happened

`lyx webster begin-batch 1` refused with: `batch 01-paced-text-submit already has a report at _lyx/webster/reports/01-paced-text-submit.yaml — begin-batch never overwrites finished work; a report left behind by a crashed session is consumed by lyx webster record-batch 1 (or lyx webster recover-batch 1 for a recovery batch), and a stuck batch escalates via lyx webster recover-batch 1`.
The loom history shows the run had gone back to Plan-Write via a `goto` and re-batchified before Webster started, so the report was left over from earlier work, not from a crashed webster session.

## Why it is abnormal

The refusal text offers only crash and stuck-batch remedies.
After a `goto` back to planning and a fresh Batchifier pass, a stale report should be archived or invalidated by lyx, or the refusal should name that case, rather than leaving the Webster agent to guess between `record-batch` and `recover-batch`.

## What to do

Decide whether a `goto` to Plan-Write or a Batchifier re-run should archive existing `_lyx/webster/reports/` entries, or whether the refusal message should cover the stale-report case.
The evidence here is one refusal note; the cause of the leftover report is inferred from the history.

Provenance: task `bugfix-sessions`; note `webster-refusal-begin-batch.md`.
