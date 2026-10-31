---
verdict: CIRCLING
rationale: "Gating findings sit again on two keys the facts list as open in earlier rounds: F1 (BLOCKING design) on d4-inflight-darn-halt-unreachable, open in rounds 1 and 2, and F4 (MEDIUM design) on d13-retired-key-refusal-mechanism, open in rounds 1 and 2; no finding was relabeled."
---

Round 3 carries one BLOCKING finding and three more `design` findings.

- F1 (BLOCKING, `design`) maps to `d4-inflight-darn-halt-unreachable`.
  Rounds 1 and 2 raised the same in-flight route to Darn-Write, and the round 2 fix produced a route Shed refuses.
- F4 (MEDIUM, `design`) maps to `d13-retired-key-refusal-mechanism`.
  The retired-key check has now been placed wrongly in each of three rounds.
- F3 and F5 (MEDIUM, `design`) are first raised in round 3, so they do not block on their own.
- F2, F6 and F7 are `consistency` and non-gating.
  F2 maps to `d4-decision-add-darn-refusals-way-forward`, F6 to `d4-record-write-skip-not-resume-safe`, F7 to `d5-validate-guard-reads-fenced-headings`.

Both gating keys are settled by a fix the finding already states: F1 names one route (`goto --to Record-Write`, then `resume`) with its evidence, and F4 names configengine as the place for the check.

No finding is relabeled, and the review has no `## Focus departures` section.
