---
round: 1
focus:
  - "Rubric `loom-rubric-code-review` in `contracts/`: settle whether the darn form (board entry snapshot plus record, own-entry and unrecorded-parent-correction checks) keeps every rule the deleted `loom-rubric-darn-review` held, and whether the loom form lost none of its own."
  - "Config migration (`configreg.KeyMoves`, `MigrateConfig`, Reconcile, run-settings preparation): settle whether each retired `darn_*`, `review_*` and older `review_fan` key moves to the right `code_*` or run-wide key, and whether the default move rule behaves as the summary claims at base and sparse layers."
  - "`commitDarnDiscussion` and `darnengine.BoardEntrySnapshot`: settle whether the snapshot is written before the discussion directory commit, appears in both the Bouncer's `artifact_paths` and the review profile's `fasit.paths`, and fails the commit on a board read error or a missing entry."
  - "Budget 0 and reroute (`rerouteReviewed`, `perchHardCap`): settle whether a darn run with `code_max_bounces` 0 still skips the perch and `Describe`, loom's budget-0 behaviour is unchanged, and the hard cap reads loom.yaml for every Bouncer row."
  - "Leftover `Darn-*` perch names, `NameDarn*` constants, `darn_*` review keys or `loom-rubric-darn-review` references anywhere in code, recipes, stencils, docs and tests, apart from the intended `reportengine` literals."
  - "Tests: settle whether deleted or rewritten darn perch tests leave any behavior uncovered, and whether new tests duplicate existing coverage."
  - "Docs: settle whether the `doc.go` files, `docs/overview.md`, `pattern/PATTERN-verified-tree.md` and the orch role stencil match the diff, and whether the summary's claims hold."
---

Seed focus from the change description and the board entry; the reviewer still reads the whole diff.
