---
verdict: CONTINUE
rationale: "Round 2 carries BLOCKING F1 (design): the allowlist still admits battenrecipe.RecipeEngines and fabricvocab.FailsBareCheck, which read no own-package state and write no record; it maps to key allowlist-keeps-pure-shedcheck-in-production, open since round 1. No relabel."
---

Round 2 raised three findings.

- F1 (BLOCKING, design) is the same defect as round 1's allowlist key, now on two other entries.
  `PATTERN-test-only-code` lets a test-only function stay in production only when it reads its package's own state or writes a record its package alone writes.
  Neither function does, so the entries fall outside the exception.
  The rule makes a BLOCKING finding gating, and the key is open from an earlier round, so the loop continues.
- F2 (MEDIUM, consistency) is the stale `_lym/landing/summary.md`, deferred to the run's parent.
  It is not gating.
- F3 (LOW, consistency) reopens `fabricengine-doc-names-gone-snapshot-reader`.
  It is not gating.

No finding is relabelled and there are no focus departures.
