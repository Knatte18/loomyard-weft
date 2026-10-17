---
round: 1
focus:
  - "Card 5 (`05-run-bounded-lookup.md`) bundles the `FindPullRequest` re-sign, `Deps.RunStart`, four landing rows, both approve and reject verbs, `landingDeps` and many test helpers in one card. Settle whether that is one independently reviewable unit or a granularity finding, and whether the re-sign arrows on `landingDeps`, `approveDeps`, `rejectDeps` and `findPR`-using symbols match what its `Intent` changes."
  - "Card order across 4, 5 and 6: card 5 calls `seedRunStart` (card 4) and card 6 reads the `olderOpen` result that card 5 introduces. Settle whether any card depends on something a later card does, and whether card 5 leaves the tree compiling with Publish discarding `olderOpen`."
  - "Reachability of the refusals: card 4's unreadable-seed error and card 5's approve and reject `SeedUnreadable` refusal, and card 6's older-open stop. For each, settle whether a named input and a named test reach it, and whether an upstream check rejects that input first."
  - "Attack-surface question on card 5's zero `RunStart` bound, which keeps the unbounded lookup, and on card 3's required marker for the orch role stencil. Settle whether each card's `Intent` states what the change lets through and what bounds it."
  - "Fidelity to the decision record: D1 to D4, the Scope-In list (including the `doc.go` updates for `internal/landingshed`, `internal/loomcli` and `internal/alignment`) and the Constraints. Settle whether each is carried by some card and whether any card adds scope the record does not license."
  - "Test economy in cards 4 to 6: new top-level tests `TestSeedRunStart` and `TestFindRunPullRequest` against the Constraint to extend existing tables. Settle whether an existing test already covers the behavior, naming it."
  - "Verify coverage: whether every package a card targets, including `internal/alignment`, `internal/orchengine` and `contracts/stencils`, has its untagged and `-tags integration` tests run by the `## verify:` section."
---

Focus is derived from the plan overview, the decision record and cards 4 to 6.
Cards 1 to 3 were not read in depth for this seed.
