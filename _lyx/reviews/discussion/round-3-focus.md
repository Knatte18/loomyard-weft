---
round: 3
focus:
  - "Decision 5 (cost model) and the acceptance criterion that two small cards sharing a file group merge: starting from the Decision 5 formula, work out whether a merged batch can ever cost less than the sum of its cards alone, so that the exact DP of Decision 7 can pick a multi-card batch; settle whether some term is paid once per fork."
  - "Decision 1 (card order), Decision 3 (calibration report runs with the coefficients of a named `batcher.yaml` profile) and Decision 8 (profile parsing and the template's initial cost profile): settle which card first provides the profile type, its parse and the initial coefficients, and whether the calibration report's card can be built from the cards before it."
  - "Decision 4 (estimator) against Decision 5: settle whether the estimator's inputs, outputs and the definitions of read set and message count each appear once, and whether the coefficients are applied exactly once."
  - "Decision 5 and Decision 6 thresholds (`alone_above`, `budget`) against the calibration rule's scale factor: settle which unit the thresholds are in before and after calibration, and which coefficients the factor multiplies."
  - "Decision 10 (a state without a recorded partition runs the identity batchifier) against Decision 11 (rebaseline replaces the recorded partition): settle what a rebaseline does on a state with no recorded partition."
---

The entries name questions only; each is to be judged against the current text of `decision-record.md`.
