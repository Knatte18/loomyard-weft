# Webster: recovery never dead-ends, stale config fails at the start, a computed Merriam base

`webster-dead-ends` · Task · bug, enhancement, webster · awaiting · PR-Gate

## Goal

Webster's recovery paths never dead-end: every recovery instruction names a lyx verb, a recovery prompt states the worktree it starts from, an operator decision reaches a begun card, a failed record-batch leaves a retryable batch, and a stale batcher config fails at the start of a run instead of mid-Webster.
The batchifier's Merriam base is computed instead of fitted.

## Reset targets

`webster-audit-recovery` (PR #381) added `lyx webster reset --to start|pre-fix` so recovery never needs a denied `git reset --hard`.
Three sites still tell the agent to run `git reset --keep <sha>` rather than a lyx verb.
They move back to the report head or the last-batch head in record-batch, recover-batch and accept-audit (`internal/websterengine/gitwrap.go`), plus standalone mode.

Widen `lyx webster reset` with the report-head and last-batch-head targets, and point the three sites and standalone mode at it, so every Webster recovery instruction names a lyx verb.

## Recovery prompt on a dirty tree (from issue #406)

A recovery run for a Webster segment was started with the reason `none` and an instruction to continue from the committed tree, while the worktree held a large uncommitted body of work from the earlier attempt.
The recovery prompt did not mention these changes or say whether to keep or discard them, so the agent had to judge on its own.

When lyx starts a recovery, it detects uncommitted changes in the worktree and names them in the recovery prompt, with an explicit keep-or-discard instruction.

Provenance: task `test-suite-prune-core`.

## An operator decision amends a begun card (from issue #413)

An operator decision made while Webster runs cannot reach the plan card of a batch that has begun.
Seen 2026-10-07 in `bugfix-sessions`: the decision went into the decision record, but `lyx webster rebaseline --card 01` refused ("the edited plan changes the cards of a batch this run already begun"), and its way forward, `lyx webster reset --to start` then `lyx webster run --fresh`, has no path through loom: the Webster producer always runs with `Fresh: false` (`internal/shedadapters/webster.go`).
An edit left on disk while the master recorded the batch blocked the run with `plan_drifted` until the plan was restored.

Issue #413 shows the same dead end from loom's own flow: a `goto` back to `Plan-Write` after batch 1 had begun re-ran Plan and Batchifier with an edited plan, `record-batch 1` refused with `plan_drifted` and named `rebaseline`, and `rebaseline` refused because the card had begun.
The first refusal's advice leads straight into the second, and the only exits discard the run's Webster progress.

Wanted (operator, 2026-10-07): an operator decision taken mid-Webster lands in the plan and the decision record together, without extra review or PR-Gate steps, whether or not the card's batch has begun.

Since `bugfix-webster` (d536a1a4e) the drift advice offers rebaseline only for an edit that keeps begun batches' cards, and rebaseline accepts an edit to a failed, dead or stuck batch.
Still open: a decision against a begun, non-terminal batch, a plan re-run by loom after a batch began, and the Webster producer's `Fresh: false`.

## begin-batch after a failed record-batch (from issue #441)

After `lyx webster record-batch` failed on a config-load error, `lyx webster begin-batch 1` refused: the batch already has a report and state.json records it as begun and not terminal.
A record-batch refusal that is not about the report itself leaves the batch begun with a report on disk, so the retry path through begin-batch is blocked, and the refusal lists several remedies without saying which applies.

- record-batch fails before touching batch state when its config does not load, or a re-run of record-batch after the config is fixed recovers the state.
- The begin-batch refusal names the remedy that applies to the recorded state.

Provenance: task `lifecycle-waits`.

## Stale batcher config caught at the start (from issue #440)

A run halted `blocked` at Webster when `record-batch` refused because the pair's `batcher.yaml` still carried `startup_context`, which the binary no longer accepts.
The run had passed Preflight, Loom-Preflight, Discussion, Plan and Batchifier before the drift surfaced, and it landed as a human escalation rather than a config failure.

- Preflight or Loom-Preflight loads and validates `batcher.yaml`, so schema drift fails at the start of a run.
- A removed key such as `startup_context` is migrated or reported clearly when a pair config was seeded from an older template.

Provenance: task `lifecycle-waits`.

## Computed Merriam base

Left from `batcher-v2`'s PR-Gate (2026-10-07).
`batcher-v2` sizes a batch's start context as `master_base + (k−1) × batch_growth`, with `master_base: 52000` a fitted constant in `internal/batcher/template.yaml`.
The operator wanted the base computed, since it is fixed by what Merriam loads.
Measured over three `cautious` runs, Merriam starts at about 20.6K and orients itself up to 50–54K before the first fork; a larger plan or another repo can land far from 52K.

- A function in `websterengine` computes Merriam's fixed part at partition time: CLAUDE.md, the Webster stencil with the inlined PATTERN overview, and the plan files Merriam reads; the system prompt and tools enter as a measured constant.
- `Batch` takes that value, beside `before`, and `startContext` becomes `computedBase + orientation + (k−1) × batch_growth`.
- `master_base` gives way to an `orientation` coefficient in `Weights`, `template.yaml` and the calibrate fit in `tools/tokencount`.
