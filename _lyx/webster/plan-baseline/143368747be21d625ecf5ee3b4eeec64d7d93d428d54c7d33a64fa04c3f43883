# Card 15 — loom-merge-config

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`

**Intent:**
Add the Merge-Review perch's three `loom.yaml` keys (D2).
`Config` gains `MergeMaxBounces int` (`merge_max_bounces`), `MergeAutoAccept bool` (`merge_auto_accept`) and `SkipMergeReview bool` (`skip_merge_review`).
The template carries `merge_max_bounces: 1`, `merge_auto_accept: true` and `skip_merge_review: false`, each with a comment in the style of the Webster keys; the `skip_merge_review` comment says it skips only the review, while Merge-In, the parent lock and the reroute still run.
`LoadConfig` refuses a negative `merge_max_bounces` beside the other per-perch budgets, and its file comment names the three keys.
A pair whose `loom.yaml` lacks the keys reads their zero values until `lyx config reconcile` adds them: budget 0 takes `review_max_bounces`, no auto-accept and the review on, the stricter side.
`TestLoadConfig_Loads` asserts the template values and the zero values of a file without the keys; `TestLoadConfig_Refuses` gains the negative budget row.
`TestConfigTemplate_ContainsEveryConfigYAMLTag` covers the template keys unchanged.

**ImpactSummary:** three new per-worktree loom keys with no reader yet.

**Commit:** 15: loom: merge_max_bounces, merge_auto_accept and skip_merge_review
