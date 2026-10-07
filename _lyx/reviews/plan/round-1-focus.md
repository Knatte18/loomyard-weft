---
round: 1
focus:
  - "Test economy, cards 4 and 6: for each new row or test, name the existing test (in the files the card already edits, or in `internal/fabriccli/adoptremote_integration_test.go`) that covers the same behavior. Card 4 plans rows across four fabricengine/fabriccli test files plus the new fabriccli file, and card 6 plans the same notice behavior in `finalize_test.go`, `finalize_integration_test.go` and `landingdeps_integration_test.go`. Settle which rows duplicate another and which add distinct coverage."
  - "Test economy, card 1: the hubgeom `parent_test.go` case and geometry step (b) both assert the `ResolveParent` lock-directory error. Settle whether one covers the other."
  - "Fidelity, D1 to D3: does card 1 carry every weft-creating helper D1 lists (fork, drop-runs fork, add's adopt, reconcile's `adoptWeftWorktree`, and the D7 origin-adopt path), and is the origin-adopt path's `.weft/` creation carried by card 1, card 2 or card 4?"
  - "Fidelity, D6 and the acceptance criteria: trace each D6 bullet and each `add` acceptance criterion (including the skip-option row, the archive-tag-plus-local-weft row, the fast-forward row, and the failed-Add-keeps-fast-forward row) to a card row. Flag a criterion no card tests and any card scope D6 does not license."
  - "Fidelity, D4: do cards 5 and 6 carry every D4 clause (HubWide filter, diff read before any parent-side mutation, failed read still lands and queues a notice, `QueueNotice` error and `queued=false` leave the verdict, resume queues again, empty diff queues nothing)?"
  - "Granularity, card 4: it edits `leftoverKind`, `leftoverDivergedAdoptedWeft`-adjacent types, `KindRepoAdvanced` and several probe functions in one card. Settle whether any private supporting type is carded separately or any public-surface symbol is bundled wrongly."
  - "Attack surface, cards 2, 3 and 4: each widens an adoption path (archive-guard bypass, a warp refusal replaced, a fetch-created branch). Settle whether each card's `**Intent:**` states what it now lets through and what bounds it, per D6 and D7, and whether rollback and the origin-untouched bound are stated where a new branch is created."
  - "Verify coverage: cards target `internal/fabricengine`, `internal/fabriccli`, `internal/hubgeom`, `internal/landingshed` and `internal/loomcli`, and read `configreg`, `configengine`, `orchcli` and `orchengine`. Settle whether any package a card edits or creates a test in is missing from the `## verify:` test runs, build-tagged tiers included."
  - "Test paths after the prunes (D9 and the post-Discussion decision): for each test file or named test a card edits, check it exists in the tree today and holds the behavior the card extends. Flag a card naming a deleted, merged or renamed file or test."
---

Seed for round 1; no round has been judged.
Candidates were checked against `Do not flag` and the stencil symmetry rule: none concerns dependency fields, `ImpactSummary` on non-Edit cards, or the format checks the gate runs.
