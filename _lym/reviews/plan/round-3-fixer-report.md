# Round 3 fixer report — plan

All three findings were fixed, and each premise was checked against the code first.
`lym loom validate-plan` reports only the baseline informational `create-new-unit` finding after every change.

## Changes

- **F1 (BLOCKING), fixed differently in one target.**
  The corrected method was applied again from scratch, not limited to the review's list: a search for every test that reads a `Config` plan field (`PlanLenses`, `PlanReviewerModel` and the rest) after `LoadConfig`, and for every consumer of `loomrecipe.EntryNumbers` (`overviewEntryNumber`, `renderChecklist`) together with every numbered-row string in the tests.
  The method itself is written in the plan writer's instructions, which lie outside this round's write surface (`_lym/plan`), so the round applied the method's results to the cards.
  - Card 06 gains the Edit target `internal/loomengine#TestMigrateConfig` and a reader bullet.
    The bullet says the "migrated layers load with the advisors each layer's migration left" subtest drops its `PlanLenses == "twill-plan"` conditions and their message operands, while the table rows above it keep pinning the `plan_producer` → `plan_lenses` rewrite.
    No other test reads a plan field after `LoadConfig` without already being a card 6 target: `TestPlanTable` sets `cfg.Plan` on a config it builds itself.
  - Card 09 gains `internal/loomrecipe#TestEntryNumbers`, `internal/battencli#TestOverviewRunCells`, `internal/battencli#TestDobbyRunCells_ReadsTheRunsFromDisk` and `internal/battencli#TestDobbyRunCells_ResolvesTheChildPairAgainInEachRunShedSpell`.
    It also gains an Intent bullet saying the tapestry map renumbers (Discussion-Write 2 through Friction-Reflect 15), `6.6 Plan-Write` becomes `6.5 Plan-Write`, and `TestOverviewRunCells`' `6.7 Plan-Burler-Review` becomes `6.6 Plan-Burler-Review`.
    The review named that last row nowhere, but the search found it.
    **Fixed differently:** `internal/battencli#TestDobbyLines` is not added.
    Its `cells` map is a literal input (`"flying": {Stage: "6.6 Plan-Write"}`, `internal/battencli/overviewdobby_test.go:50`) that the test renders as given, so no recipe number reaches it, and card 9 cannot break it.
  - Card 10 gains `internal/loomrecipe#TestEntryNumbers` and a sentence saying Drift-Verify takes the number Code-Bouncer held and the Code entries take the next.
  - Card 11 gains `internal/loomrecipe#TestEntryNumbers` and `internal/loomcli#TestRenderChecklist`.
    It also gains a sentence saying the darn map runs Drift-Baseline 3 through the Code-Bouncer entries 7, and the darn checklist rows read `[ ] 2. Record-Write`, then `[ ] 3. Drift-Baseline` through `[ ] 11. Friction-Reflect`.
  - No other card adds, drops or reorders recipe rows, so none needs these consumers.
    Cards 03, 12, 13 and 15 edit recipe files, but only for keys, gates or subjects.
- **F2 (MEDIUM), fixed as proposed.**
  Card 06's Tests paragraph gains a second `TestCheckResolved` row.
  In that row `RunWideSpec` is set, the plan perch's own `plan_max_bounces` lies above the run-wide `review_hard_max_bounces`, and `plan_hard_max_bounces` is 0.
  The row asserts the prefix `loom config key "review_hard_max_bounces"`, the hard-cap route `lym config loom` and the soft-cap route `lym config tapestry`.
- **F3 (LOW), fixed as proposed, then extended by the self-check.**
  Card 02's Intent says the `wantKind == ""` branch of `TestArmFromSeed_RefusalEnvelopeKinds` asserts `wantSuffix`, and counts `way forward:` clauses against `wantWayForwardCount`, when the row sets them.
  The "unknown recipe on status" row sets the re-seed suffix and a count of 1.

## Self-check

- fix-consistency, MEDIUM: card 06's Tests sentence "`TestLoadConfig_Loads` asserts the plan fields of `Config` come from `tapestry.yaml`" contradicted the new `TestMigrateConfig` bullet and the Intent's "`LoadConfig` never reads `tapestry.yaml`".
  Fixed as proposed: the sentence now says `TestLoadConfig_Loads` drops its plan-field expectations and `TestLoadTapestryConfig` asserts the fields `Config.WithTapestry` applies.
- fix-claims, MEDIUM, and fix-proposal, MEDIUM, one finding reported by both lenses: card 02 claimed the status row counts one way-forward clause, but the count check sits after the early return (`internal/shedcli/cli_test.go:274-293`).
  Fixed as proposed: the kind-less branch also checks `wantWayForwardCount`, and the status row sets it to 1.
- fix-affected-set: no findings.
- fix-claims, wording nit (the forks did not raise it): the card 11 checklist range "`[ ] 3. Drift-Baseline` through `[ ] 11. Friction-Reflect`" also spans the `[x]` and `[>]` lines, which shift too.
  Left as written, since "through" covers every line in the range.

## Disputed

None.

## Deferred

None.
