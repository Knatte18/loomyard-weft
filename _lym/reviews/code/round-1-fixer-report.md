---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "The proposed override (keep harden-live-copies when the hub sets it) cannot be detected, since gateslot.LoadConfig fills the template's harden-live-copies at tier 0 for every hub, so the key would always read as set. The key is retired instead and the copies call runs under harden-live, the entry's second option."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: "The proposed profile instruction on the HardenFix row is not rendered: composeFixPrompt never printed Target.Instructions. It now appends them to the fix step's work statement, and the recipe row carries them under profile.target.instructions. The proposed 'one commit per finding' is left out because the fix step's own group rule governs commits."
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
---

# Fixer report, round 1

## Changes

- F1: `Campaign.RoundFacts` opens with the round's focus file (the charter) and the decision record (the operator's ask), through a new `Campaign.DecisionRecordPath` that `hardenCampaign` sets.
  The rubric's judge section says to read both, and names them as the one exception to the judge's rule against opening artifacts.
  `TestCampaignRoundFactsNameTheCharterAndTheOperatorsAsk` covers rounds 1 and 2.
- F2: the board entry read is `readBoardEntry` over a `boardTaskReader`, and `hardenBoardEntry` wires it to the hub's board.
  `TestReadBoardEntry` covers the present, absent, read-failure and open-failure paths and asserts the refusal's slug, `hardener.packages` and `lym loom resume`.
  `TestHardenCampaign_EntryRefusesAnEntryTheHubsBoardLacks` reaches the production seam through `hardenCampaign`.
- F3: `hardenPreflight` takes the campaign's `Entry` and reads it when the config names no package, so a missing entry refuses before any agent runs.
  `TestHardenPreflight` has a row for it.
- F4: the copies call runs `harden-live` with `--copies`, and `harden-live-copies` is retired from the template, `goReadGates`, the gateslot test defaults and the PATTERN table.
  `CheckCampaign` applies the billed-tier, one-tier-when-copies-run and package-test checks to `harden-live` alone.
  `TestCheckCampaign` and `TestLiveGate` follow, and the template comment, the hardenshed doc and the `NewLiveGate` doc are updated.
- F5: the rubric gains the merge bar, the safety pass's no-invented-work rule and the count rule without a count in the focus.
  The fix-half rules the review listed (determinism proof, maintained sandbox suite, docs and PATTERN updates) went to the fixer's instructions under F6.
  The seed template's cost declaration leaves a fan or cluster test off the ban when the operator's focus is that path.
- F6: `composeFixPrompt` appends the profile's target instructions to the work statement, and the hardener recipe's HardenFix row carries the fix-half rules there.
  `TestComposeFixPrompt_TargetInstructions` covers it.
  The rubric keeps the review-half rules.
- F7: `TestHardenRoundKey` gains a row that drives the seed call through a named stub stencil and asserts the campaign's brief reaches the prompt.
- F8, F9: `hardenPreflight` returns the `gateslot.LoadConfig` error for an absent hub gate.yaml, and `TestHardenPreflight` seeds a sparse gate.yaml for its passing rows and has a row for the absent file.
- F10: an empty commit carries the `Loosened-Check` trailers.
  The commit of F1 to F9 carries those for the `TestCheckCampaign` rows this round changed.
- F11: the default `new` subtest asserts exactly two bares.

Gate repair: five comment lines that ran to 120 characters or more, in `campaign_rows.go`, `doc.go`, `live_test.go` and `hardenwiring.go`, are split at sentence boundaries.
A second gate pass flagged the `RoundFacts` doc comment as a wrapped sentence, and it is rewritten as three short sentences.

A verify run then flagged the F11 assertion message in `cli_integration_test.go` for naming the warp and weft sides, which the fabric-vocabulary scan reserves.
The message now says the code bare and the records bare.

Gates: tier 0 over the changed packages and tier 1 over `internal/hardencli` are green.

## Self-check

- fix-consistency, MEDIUM: the shared judge template forbids opening the artifacts while the rubric and facts direct the judge to the focus file and decision record. Fixed: the rubric names both as the one exception.
- fix-code-correctness, LOW: `hardenBoardEntry` was reached by no test. Fixed: `TestHardenCampaign_EntryRefusesAnEntryTheHubsBoardLacks`.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
