---
round: 2
focus:
  - "Check that the Harden-Bouncer judge's inputs, built in `internal/hardenshed/campaign_rows.go` `Campaign.RoundFacts`, name the current round's focus file (the charter) and the decision record, so the judge can restate the charter and stay inside the operator's focus; settle whether round 1's seed charter is covered too."
  - "Check that the missing-board-entry refusal in `internal/loomcli/hardenwiring.go` (`hardenCampaign`'s Entry seam) has a test that reaches it and asserts a message naming the slug, `hardener.packages` and the way forward, per PATTERN-refusal-way-forward; also whether a SeedBrief failure in `internal/shedadapters/bouncer.go` (`runSeedSpawn`, `seedCall`) still degrades to an empty-lists seed instead of refusing."
  - "Check that the harden copies run's tier in `internal/hardenshed/live.go` (`NewLiveGate`) and `CheckCampaign` follows the board entry's No hub gate writes design (derived from `harden-live` or folded in), and that `hardenPreflight` in `internal/loomcli/hardenwiring.go` refuses a hub with no gate.yaml before any agent spends tokens."
  - "Check that the fix-half rules the diff adds to `contracts/stencils/harden/harden-rubric-review.md` (fix every finding, one commit per finding, deferral, regression tests, package doc update) reach the HardenFix fixer, whose burler prompt carries no rubric, and that the rubric and `harden-template-seed.md` carry the Crucible rules the board entry asks to carry over."
  - "Check that a `harden_round` Bouncer row built in `internal/shedrecipe/entries_bouncer.go` carries the campaign's SeedBrief and `seed_stencil` into the seed prompt, and that a test asserts it."
---

The focus lists the places the committed diff's riskiest wiring sits; it caps nothing.
