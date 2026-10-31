---
verdict: CIRCLING
rationale: "F1 (BLOCKING, design) is the affected-set method gap on key card6-missed-moved-key-readers, open in rounds 1 and 2 and still open in round 3 with new instances (indirect readers of changed values); F2 (MEDIUM, design) reopens tapestry-run-wide-key-routing; no relabels."
---

The latest round carries a BLOCKING design finding (F1) on a key the facts list as open in two earlier rounds.
It is the same method gap each time: the plan writer builds each card's affected set from key literals and misses tests that read a changed value indirectly, here a `LoadConfig` read-back of fields a card stops filling and the numbers `loomrecipe.EntryNumbers` gives each row.
Counts fall (13, 7, 3), but the gating key persists, so the rule gives `CIRCLING`, not `CONTINUE`.

Mapping to ledger keys:

- F1 maps to `card6-missed-moved-key-readers`, BLOCKING, design.
- F2 maps to `tapestry-run-wide-key-routing`, MEDIUM, design: the round 2 routing defect is fixed, and the residual is a missing named test for the run-wide hard cap under a plan perch's own soft cap.
- F3 maps to `shed-beam-seed-no-reseed-way-forward`, LOW, consistency.

The round 2 keys `not-committed-refusal-darn-write-in-flight`, `card10-duplicate-drift-verify-edge-test`, `card17-runrecordbullets-missing-resign-arrow` and `card18-duplicate-escalate-judged-first-assertion` are resolved: the review's rejected section clears each.

No finding is relabeled, and the review has no `## Focus departures` section.
No declared PATTERN entry is cited by a finding.
