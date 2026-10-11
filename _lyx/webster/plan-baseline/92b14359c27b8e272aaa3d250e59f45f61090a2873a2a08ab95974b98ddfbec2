# Card 8 — perch-budget-keys

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomengine#templateConfig`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Intent:**
D10's config half: `loom.yaml` gains a bounce budget and an auto-accept flag for each of the three loom perches.

`Config` gains `DiscussionMaxBounces`, `PlanMaxBounces` and `WebsterMaxBounces` (`discussion_max_bounces`, `plan_max_bounces`, `webster_max_bounces`, ints) and `DiscussionAutoAccept`, `PlanAutoAccept` and `WebsterAutoAccept` (`discussion_auto_accept`, `plan_auto_accept`, `webster_auto_accept`, bools).
The template sets every budget key to 0, meaning the perch takes `review_max_bounces`, and every flag to false, so the defaults keep today's behavior; `review_max_bounces` and `review_circling_checkpoint` stay the run-wide defaults, and the model keys keep picking each half's model per round.
Each template comment states its key's meaning, that a budget of N gives N judged Burler rounds, that a spent budget with the perch's flag true settles the round as accepted instead of escalating to the parent, and the per-perch relation: a checkpoint above a perch's budget means that perch never rules CIRCLING and reaches the budget escalation, or the accept, instead.
`LoadConfig` refuses a per-perch budget key below 0, naming the key: `loom config key "webster_max_bounces": must not be negative, got -1; set it to 0 to take review_max_bounces, or to a positive integer, in loom.yaml`.
A resolved budget below 1 can then come only from `review_max_bounces`, which `LoadConfig` already refuses naming that key, so every resolved budget below 1 is refused naming the key it came from.
`config.go`'s header lists the new keys.

The refusal-spec row `config: review key below 1` names the three per-perch keys beside `review_circling_checkpoint` and `review_max_bounces` (a per-perch key below 0), with both ways forward; `TestLoadConfig_Refuses` gains a row per per-perch key at -1, each asserting the refusal names that key, and `TestLoadConfig_Loads` gains a row loading set values of all six keys, while `templateConfig` holds their template defaults.
`docs/overview.md`'s `loom.yaml` key list names the six keys.

**ImpactSummary:** six additive `loom.yaml` keys and one new load refusal; nothing reads the keys until card 9.

**Commit:** 8: loom: per-perch bounce budget and auto-accept keys in loom.yaml
