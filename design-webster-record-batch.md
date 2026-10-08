# Webster record-batch and rebaseline: no needless re-drive, a calibrated Merriam base

`webster-record-batch` · Task · bug, enhancement, webster · running · Webster

## Goal

Webster's record-batch and rebaseline in websterengine never force a needless re-drive or dead end: an abbreviated head_sha that prefixes HEAD is accepted, a forked batch's transcripts are found, a Card-Index-only change is accepted on a record without an overview frame, and the Merriam fixed context can be re-measured. The first half of plan-gate-done-cards (the plan gate skips done cards) landed on main on 2026-10-08; only its rebaseline half is in scope.

## From `record-batch-short-sha`

Imported from [issue #456](https://github.com/Knatte18/loomyard/issues/456).

`lyx webster record-batch` refused a batch report because the report's `head_sha` was an abbreviated SHA (9 characters) while the worktree's HEAD is the full 40-character SHA.

Refusal text (paths trimmed):

```
webster: batch report _lyx/webster/reports/01-carry-over-write.yaml: head_sha "ae24b9bb0" does not match the worktree's actual HEAD "ae24b9bb0eca5384fc9fd38764b9cb446aced655"; only merge commits, such as a parent merge-in, may sit between a fork's reported head and HEAD; way forward: 1) run `git reset --keep ae24b9bb0` to move HEAD back to the report's head_sha; 2) re-run this verb
```

Why this is abnormal: the abbreviated SHA is a prefix of HEAD, so the report names the right commit.
The comparison looks like an exact string match, so it reports a mismatch and then suggests `git reset --keep` to the commit HEAD already is, which is a no-op and does not resolve the refusal.
A fork writing `git rev-parse --short HEAD` into the report is a natural thing to do.

What to do: resolve the report's `head_sha` with `git rev-parse` (or compare by prefix) before comparing, and/or require and document the full SHA in the report schema and the fork prompt.
The "way forward" text should not suggest a reset when the reported SHA already resolves to HEAD.

Provenance: task `review-loop-convergence`; notes: webster-refusal-record-batch-2.md

## From `record-batch-no-transcripts`

Imported from [issue #457](https://github.com/Knatte18/loomyard/issues/457).

`lyx webster record-batch` refused to attribute a batch report with "zero new fork transcripts since the previous batch boundary", although the batch's fork had delivered a report.

Refusal text (paths trimmed):

```
webster: batch 01-carry-over-write's report could not be attributed: zero new fork transcripts since the previous batch boundary — the batch was never forked (or its transcript is not on this machine: fork transcripts are machine-local, so a crash window resumed on a different machine cannot re-attribute its report); report archived to _lyx/webster/reports/01-carry-over-write-<timestamp>.yaml; way forward: lyx webster begin-batch 01 re-drives the batch
```

Why this is abnormal: a report existed for the batch, so work was done, yet attribution found no fork transcript.
Possible causes to check: the transcript boundary advanced past the fork's transcript (for example a repeated `record-batch` or `begin-batch` for the same batch), or the fork's transcript was not located.
The refusal archived the report and told the operator to re-drive the whole batch, which discards finished work; a second refusal on the same batch followed (see the head_sha issue filed alongside this one).

What to do: find out why the transcript count was zero on a batch that was forked, and make `record-batch` idempotent or recoverable for an already-attributed batch instead of forcing a re-drive.

Provenance: task `review-loop-convergence`; notes: webster-refusal-record-batch.md

## From `plan-gate-done-cards`

Imported from [issue #463](https://github.com/Knatte18/loomyard/issues/463).

Seen 2026-10-08 in `shuttle-send`, following #462: the hub moved a run blocked at Publish back with `lyx loom goto --to Plan-Bouncer`, so Plan-Review would raise an uncarried parent decision against the plan.

- The plan gate validates every card against HEAD, so cards 1-10, which Webster had already run, all failed (`create-already-exists`, `glyph-not-found`, `path-missing`, `caller-uncovered`); the fixer could clear them only by retiring or rewriting done cards.
- The fixer added follow-up cards 11 and 12 with Card Index lines, but `lyx webster rebaseline --card 11 --card 12` refused: "00-overview.md changed and this run recorded no overview frame", because the run's webster record predates overview-frame tracking. Its only way forward, `lyx webster reset --to start`, would re-run all ten done cards.
- `goto` from Plan-Bouncer cannot reach Webster, so there is no route past the plan gate once the plan holds done cards.

The hub restored the recorded plan and will fix the defect forward on the task branch at Publish.

Expected: a plan gate after Webster began judges only cards after the last begun batch (done cards are history, not targets), and a webster record without an overview frame accepts a Card-Index-only change, or records the frame lazily.

## From `merriam-fixed-context-calibrate`

From webster-dead-ends PR-Gate (2026-10-08), moved out of `binary-change-and-rules` at its Discussion.

`merriamFixedContext` (`internal/websterengine/merriambase.go`) is a constant from one measurement of the system prompt and tools, so the computed Merriam base drifts silently as Claude Code changes.
A calibrate step in `tools/tokencount` re-measures it.
