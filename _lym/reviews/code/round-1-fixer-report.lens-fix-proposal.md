```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1–F5), as an outsider.

**F1 (proposal: exhausted batches take the steps `RecoverSpawnOrAttach` gives)**

- **MEDIUM.** The changed text is `createTargetWayForward`'s `exhausted` arm in `internal/websterengine/recordbatch.go`, which returns `exhaustedRecoveryWayForward`. The arm returns steps that never amend the card, so for an exhausted batch it can still dead-end, one hop later.
  - Evidence: its output is "reset --to batch-start --batch 01; recover-batch 1; or reset --to start; run". The failure it replaces is a Create target that does not resolve or a signature mismatch. A recovery after a batch-start reset builds the target as the card names it, so the same target fails again unless the card is amended.
  - Nothing in the text tells the operator to amend the card first.
  - The `signature-mismatch` case is the worst. The exhausted arm now also overrides the `signatureMismatch` flag, which was the only branch that insisted on an amendment (test row `ExhaustedRecoveriesBeatSignatureMismatch`).
  - The reviewer's proposal reused the text because it already exists for an exhausted batch. That text was written for a failure a recovery can clear, not for a plan defect.
  - A sound alternative is to prefix the exhausted steps with the amend-and-rebaseline step. I could not confirm whether `reset --to batch-start` accepts a plan edit and a rebaseline. Check that before choosing, since it decides whether the amend step is possible there at all.

**F1, `in.Batch.Recoveries >= maxRecoveries` as the trigger**

- No finding. In `recordbatch.go`, a first record-batch failure has `Recoveries` 0, so it still gets the amend-then-recover route. After the second counted recovery it hits the exhausted branch. This matches `RecoverSpawnOrAttach`'s own condition, minus the `rendersNewAmendment` and `waitsOnParent` clauses. Those are false for a terminal-failed batch, per the review.

**F2 (proposal: export `failCardNotDone` and `cardNotDoneFor` through `export_test.go`)**

- No finding. I could not build a case where it passes with the defect in place:
  - Dropping the `signature-mismatch` case in `cardNotDoneFor` turns that row into the recover-and-amend text.
  - Ignoring `signatureMismatch` in `createTargetWayForward` does the same.
  - Reordering the `wayForward == ""` guards flips the precedence rows.

**F3 (proposal: word the amend step "Create target or Rename new side")**

- **LOW.** The wording applies to every `createNotDone` way forward, but only `create-not-done` and `signature-mismatch` set the flags. A Rename's plain `rename-not-done` finding sets neither flag, so it still gets the default recover-batch way forward. That is the dead end issue #62 describes, now for Renames. It predates the fix and the review never asked for it, so treat this as an observation only.

**F4, F5**

- No finding. Both are test-only changes. The exact equality passed in the gate.

Tool calls made: none; my context already held everything this lens needed.
