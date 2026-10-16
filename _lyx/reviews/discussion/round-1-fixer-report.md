# Round 1 fixer report — Discussion

Every finding's premise was checked against HEAD (`clusterRulesBlock` and `auditClusterRound`, loom's `ConfigSpec` categories, `configengine` `RetiredKeys`, `loomcli` `wire`, `burlerFixProfile` and `lyx burler run`'s profile, the bench report's unrun next changes) and holds.

## Changes

- F1 (BLOCKING), fixed as proposed: D4's second bullet no longer replaces the spawn trigger, and a new sub-bullet keeps its spawn mechanics (one message, Agent tool, `subagent_type: "fork"`, one per lens, no `name`) in the fanned part. The first Acceptance criterion requires them.
- F2 (MEDIUM), fixed as proposed: D3's load refusal is replaced by declaring `fan_review` in loom's `RetiredKeys` with `CategoryNone`, so an unreconciled layer, local layer or `run_settings` still loads. The sixth Acceptance criterion adds that such a layer loads with the key ignored. I left out the proposal's trailing "reconcile prunes it" because the bullet above already says `MigrateConfig` drops the key on reconcile.
- F3 (MEDIUM), fixed as proposed: D2 drops the "never mid-run" bound and gains a bullet fixing a round's reviewer and fixer fans in the usage record when its review half starts. A new Acceptance criterion covers a fan-list edit between the two halves.
- F4 (MEDIUM), fixed as proposed: D7 gains the `fix-fan` profile bullet with its recipe-profile conflict refusal, way forward and spec row. The fourth plan-writer note is cut.
- F5 (MEDIUM), fixed as proposed: Open risks' first bullet now names every unbenched part of the Discussion fan, and D5's Rationale says "the bench's lens set with the report's two unrun lens changes".
- F6 (LOW), fixed as proposed: the named spawn is added to D9's violation list and to the seventh Acceptance criterion.
- F7 (LOW), fixed as proposed: D3's first bullet states that the fanned round and its forks move to the round's reviewer-list entry, with the hub's `sonnet[high]` example.
- F8 (LOW), fixed as proposed: D3's engine refusal names the key the round's reviewer model came from (`discussion_review` or `plan_review` when set, else `review`). For consistency, the spec-row bullet below it now says "change to that key". D7 gains the twill-fixer engine refusal bullet with its keys, way forward and spec row.

## Disputed

None.

## Deferred

None.
