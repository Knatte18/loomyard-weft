# webster record-batch: zero fork transcripts for a forked batch forces a full re-drive

`record-batch-no-transcripts` · Note · bug, webster

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
