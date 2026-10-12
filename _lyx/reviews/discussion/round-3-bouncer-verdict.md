---
verdict: CIRCLING
rationale: "F1 (BLOCKING, design) sits on key hard-cap-way-forward-misses-run-settings-layer, open in round 2 and still open, now escalated from MEDIUM to BLOCKING; F2 (MEDIUM, design) is on the also-open key mechanical-mistag-bound-missing. No relabels; no Focus departures section in the review."
---

The gating findings are F1 and F2.

- F1 maps to `hard-cap-way-forward-misses-run-settings-layer`.
  Round 2 flagged that the named way forward misses the `run_settings` layer.
  Round 3 shows the replacement route still never reaches the running task worktree, and that it breaks `PATTERN-refusal-way-forward`.
  The key is listed as open in an earlier round and is open again at BLOCKING, so the decision rule gives CIRCLING.
- F2 maps to `mechanical-mistag-bound-missing`, open in rounds 1 and 2.
  The Bound is now wrong about PR-Gate review and about downstream review segments on Merge-Review.
- F3 (consistency, MEDIUM) maps to `replay-vs-harvest-auto-accept-outcome` and F4 (consistency, LOW) is a new key.
  Neither is gating.

Three round-2 keys are resolved on the review's own focus verdicts: the goto bypass of the hard cap, the escalation over a recorded continue, and the step 4 returning-key premise.
