# Plan review round 2 — fixer report

## Changes

- **F1** (BLOCKING, design), fixed as proposed.
  Every premise checked out in the tree: `internal/loomengine/config_test.go:93` `templateConfig()` is the `reflect.DeepEqual` want of `TestLoadConfig_Loads`; `internal/darnengine/config_test.go:35` `templateConfig(mutate)` is the base of each `TestLoadConfig` want; `wiring_test.go:944` and `wiring_darn_test.go:164` compare `SegmentSettings` whole; and `internal/shedrecipe/doc.go` and `internal/burlerengine/doc.go` list the changed fields and keys.
  - Card 07: targets `internal/loomengine#templateConfig` and `internal/darnengine#templateConfig`; the Intent says both gain `ReviewAmendMaxBounces: 1`.
  - Card 15: targets `internal/loomcli#TestWire_ReviewKeysReachTheEnv` and `internal/loomcli#TestWire_DarnRecipe`; the Intent says `wantSegments` and `wantSegment` gain `AmendMaxBounces: 1`.
  - Card 13: targets `internal/shedrecipe/doc.go`; the Intent names the allowance in its segment-settings paragraph, and the `subject` key and its seams in its review-segment-rows paragraph.
  - Card 19: targets `internal/burlerengine/doc.go`; the Intent names `SessionStates` among the `ReviewResult` fields in "What the halves return".
  - To apply the recurrence method to the other added fields, I searched for whole-struct fixtures of `SegmentSettings`, `burlerengine.ReviewResult` and `shuttleengine.Result`.
    Only the two wiring tests above compare `SegmentSettings` whole.
    The `ReviewResult` literals are partial inputs.
    `isZeroResult` compares with `Result{}`, which a nil `SessionStates` keeps.
- **F2** (LOW, scope), fixed as proposed, with the bound's last clause corrected by the self-check (below).
  Card 02's D6 paragraph gains a bound sentence: only the Bouncer's successful seed of a new generation (card 12) sets the mark, once per generation and never on a re-bounce or a degraded seed.
  Shed trusts the mark as it trusts `BudgetExempt`.
  A generation starts only when the row's subject changed since it settled, or, with no settle record, since its last judge spawn, Merge-Review's merge-in being its subject.

`lyx loom validate-plan --overlay` with `{}` returns `ok: true` after every change.

## Self-check

- fix-consistency: no findings.
- fix-claims, MEDIUM: card 02's bound named only a changed settled subject, while card 12 also starts a generation in case 3, which has no settle record and compares against the last judge spawn.
  Fixed as proposed: the sentence now names both cases.
- fix-proposal: no findings.
- fix-affected-set, LOW: card 07 does not target `docs/overview.md`, whose loom config entry lists the budget keys.
  Disputed, premise false: card 20 targets `docs/overview.md`, and its Intent (line 10) says loom's config module entry and the Darn-Review budget sentence name `review_amend_max_bounces` beside the other budget keys.

## Disputed

- fix-affected-set's LOW finding (self-check), premise false: see Self-check.

## Deferred

None.
