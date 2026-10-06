---
round: 1
focus:
  - "Card 3 (`03-tooling-contracts-prune.md`): the decision record's parent addition 'cmd/testtiming's tests are core's' says card 3 drops `cmd/testtiming` from its package list and this task edits no file under it, tests included. Settle whether card 3's `Edit` targets, Intent and Commit still carry `cmd/testtiming` for pruning, and whether card 18's `Uses` of it is the only remaining touch."
  - "Skill-loading carve-out: the decision record's parent addition 'Skill-loading tests are left to skills-in-one-turn' (tests naming `Spec.Skills`, `SkillLoadSequence`, `SkillUnknown`, `DefaultSkillLoadTimeout`, `loadSkill`, `loadSkillsThenPrompt`, the `SkillLoader` fakes, in `internal/shuttleengine/claudeengine`) is a Decision. Settle whether any card or Shared Decision carries it, in particular card 14 and the `prune-procedure` fold, merge and delete steps, so a prune pass cannot touch those tests and a candidate among them gets `//testtiming:keep` naming skills-in-one-turn."
  - "Fidelity sweep over the other parent additions and Constraints in the decision record: scenario within one tier, steps sharing state with no restore bound, merges judged by fixture builds with no per-family timing, measurements in a committed report linked from `running-tests.md`. Settle whether each is carried by a card or Shared Decision as the record words it, and whether any card or Shared Decision re-introduces the superseded per-family timing or restore conditions."
  - "Sizing and batching of the group cards, notably card 17 (21 packages), card 16 (17 packages) and card 13 (9 packages): settle whether each is one independently reviewable unit, and whether the `Verify` lines on cards 13 to 15 cover the packages and tags their cards touch."
  - "Per-package cards 4 to 10, which this seed pass did not read: settle that each names its `failed` entries, tier-move leads and scenario-merge leads consistently with the record, and that none plans new tests or per-symbol tests (test economy)."
---

The seed pass read the overview, the decision record and cards 1 to 3, 11 and 13 to 18.
`support-log.md` was not read.
The first two entries come from a direct comparison of the decision record against the plan.
Neither is on the `Do not flag` list, and the writer stencil was not consulted, so a missing-item framing in the symmetry rule is for the reviewer to check.
