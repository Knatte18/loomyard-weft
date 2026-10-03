# Fixer report — plan, round 2

## Fixed

- **F1:** cards 7 (`anomalies-under-step`) and 8 (`anomaly-pending-retry`) are deleted, and cards 9 to 11 are renumbered 7 to 9, with their files, titles and Card Index rows.
  The overview drops its Tier 1 summary sentence and the operator-additions, resumed-halt-loss and who-files-a-halt sections.
  It gains one Shared Decision, as round 1's judge proposed: the board's two post-Discussion items are not carried, need the decision record amended through Discussion, and `run`'s existing `detectAndFileAnomalies` call is unchanged.
  The fakeable-seam section now names only the reflection shuttle, as the fake for the agent and its filing.
- **F2:** card 2's Intent cites only D2.
  Card 6 drops `tierOneOn`, the ownership paragraph and its test bullet, and the card-7 scenario sentence; its scenario names the `shedfake.Shuttle` as the fake issue filer, and its tests state the `selfreport` knob off so `loomPostRun`'s existing Tier 1 call files nothing.
  Card 8 (reflection-stencil) drops the Tier 1 halt-note rule.
  Card 9 (friction-capture-docs) drops the Tier 1 clause from the invariant and the `selfreport` module-line bullet.
  The references to card 10 in cards 5 and 6 now read card 8.
- **F3:** card 2's ImpactSummary names `battencli` (hook nil, empty `friction` key, opts out of error-path extras) and `shedcli` (takes its `Spec` from the recipe's `ArmAt`, no change) in place of lifecycle.

`lyx loom validate-plan` reports `ok: true` after the edits, with informational `create-new-unit` findings only.

## Deferred

None.
Carrying Tier 1 under `step` and the resumed-halt check is now left for the hub to take back to Discussion, as the overview's new Shared Decision states.
