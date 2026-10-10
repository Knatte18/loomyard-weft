# loop-small-bugs: five small loop faults fixed

Batch 01 (`01-review-budget-on-bouncer-row`) recorded `done` at head `d084607f0a956112d8efc1acf52ebe4c7a6aee79`.
The batch held the plan's six cards, committed in order:

- Review budget: the Burler's two hand-backs for judgment are budget-exempt, so the Bouncer's own check is the segment's one budget on judged rounds.
- PR-Gate prose: the loom and darn recipe headers and the overview state the PR-Gate episode rule instead of a per-task budget.
- Batten: the quiet and api-error idle clocks measure awake time through an injectable awake clock.
- `PromptOffset` is the events-file size read immediately before the prompt is sent.
- A turn end before the prompt's own turn start is logged, never held and never notified.
- The held wait ends at the next turn start, and the disagreement line says the loop stands.

## Deviations

The digest lists files touched outside the batch's declared file-ops: `internal/shedadapters/bouncer.go`, `internal/shedadapters/bouncer_clear_test.go`, `internal/shuttleengine/fakes_test.go`, `fixture_test.go`, `hold_test.go`, `wait.go`, `waitmark.go`, `waitshadow.go` and `rundir.go`.
The warnings were informational scope notes on helpers and test fixtures in `internal/shuttleengine` and `internal/battenshed`.

## Verify gate fixes

The first verify attempt failed on six tests:

- `envkit` `TestFullEnv_FillsEverySeam`, `TestNilSeams_ReportsExactPaths` and `TestNilSeams_SkipsNilLegalPaths`, plus `battencli` and `loomcli` `TestWire_EverySeamFilled`: `InnerRun.Awake` was reported as an unfilled seam.
- `loomrecipe` `TestEscalation_BudgetContinueGrantsOneRoundThenAccept`: it counted 3 budget-exempt Burler Stuck entries where it wanted 1.

The fixer reports one commit, `e4fcc3847` (`fix: list InnerRun.Awake as nil-legal and count every Burler hand-back as exempt in the escalation test`):

- `internal/testkit/envkit/envkit.go` lists `InnerRun.Awake` as nil-legal, since a nil value has a documented default that `NewInnerRun` resolves.
- `internal/loomrecipe/escalation_test.go` now asserts that every Burler Stuck entry is exempt and that their count equals `burler.Calls`, matching the new card 1 rule.

The fixer ran `lyx gate test --tags integration` over `loomrecipe`, `envkit`, `battencli` and `loomcli`, and it passed.
