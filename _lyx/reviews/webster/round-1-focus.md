---
round: 1
focus:
  - "Cards 1, 2, 12, 18 (`_windows.go` bodies): check each card's `**Verify:**` ran `GOOS=windows go vet` over its own packages, and settle whether the Windows bodies match the Linux semantics for tree kill and job tie."
  - "Cards 11, 12, 13 (loop body, detach and wait, watchdog): settle whether the stop classification, the dead-loop stop and the idle kill behave as the plan states, and whether every behavior is tested untagged through `stepCmd` over the `LoopSpec` seams, with one `integration` test for the real child."
  - "Cards 4, 5 (engine stop conditions and `WriteFailedStop`): settle whether `pause_before` and `pause_after` clear on goto and whether the failed write holds the run lock while the file reads running, against the plan's wording."
  - "Cards 3, 15 and the `loop-files-told-through-spec` decision: check `internal/shedverbs` reaches loop files only through `Spec.Loop` and the told closures, with no derived paths or `internal/shedrun` import."
  - "Cards 8, 9, 11, 12 and the `pattern-overview-cap` decision: measure `wc -c PATTERN.md` against the 16200 cap and confirm no invariant was dropped when overview lines were shortened."
  - "Cards 17, 19, 20 and every card with a `**Verify:**` or group mechanical check: confirm each group's type-specific check ran against that group's own targets, not just the first."
  - "Test economy across the new tests: look for tests duplicating an existing test's coverage, per-helper tests already covered by a public-surface test, and diff behavior with no coverage; name the covering test for each redundancy."
---

Seed focus drawn from the plan's Card Index and Shared Decisions; no prior round exists.
