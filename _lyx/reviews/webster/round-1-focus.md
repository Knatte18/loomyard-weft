---
round: 1
focus:
  - "Card 17, orch role stencil: does every statement about the batten loop and Finalize match what the batten and loom recipes do (pair creation, driving, resuming a decided child, teardown, landing by push to main and closing the pull request)?"
  - "Cards 18 and 19, `contracts/stencils/claims_test.go` `wordingClaims`: does each removed per-phrase test have a claim row or a scan covering the same phrase, and does a scan over a Go-declared set stay a scan rather than becoming rows?"
  - "Card 20: do the deleted constant-getter and delegation tests leave any behavior uncovered, and does anything still reference them?"
  - "Card 21: do the skill-skip table tests on the shuttle and orch surfaces keep both causes covered, each with its own assertion rather than a shared one?"
  - "Test economy across the diff: any new test that duplicates an existing test's coverage, and any behavior added with no coverage."
  - "Per-card mechanical checks: for each card's groups, did the type-specific check (for example `assert-no-callers` on a `Delete` group) run against every target of that group?"
  - "Comment convention: new or changed doc comments and `doc.go` text follow `CLAUDE.md` (semantic line breaks, no pinned tallies)."
---

Review range: no rework round is expected for this generation, so the range is the merge base with `product.parent`; the live round check in the rubric still decides it.
