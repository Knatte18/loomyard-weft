# The operator tuned loom defaults move into the lyx template, so every new hub gets them

`tuned-defaults-template` · Note · enhancement, config, loom

## Seen (orch, 2026-10-21)

`lyx fabric clone` seeds a new hub's config from lyx's embedded templates, whose loom defaults are neutral: no twill writer lenses, no review fans, default reviewer models and bounce budgets.
The operator's preferred loom setup lives only in the loomyard hub's default layer (`_board/_lyx/config/loom.yaml`), tuned there over time.
When the quarry hub was cloned, the orch copied those values over by hand with `lyx config loom --layer default --set ...`; without that, quarry runs would review less than loomyard runs.

The values that differ from the template today:

- `discussion_lenses: twill-discussion`, `plan_lenses: twill-plan`
- `discussion_reviewer_model: ["opus[medium]", "opus[medium]", "sonnet[high]"]`, `plan_reviewer_model: ["opus[medium]", "sonnet[high]"]`
- `webster_reviewer_model`, `webster_fixer_model`, `merge_reviewer_model`, `merge_fixer_model`: `sonnet[medium]`
- `discussion_reviewer_fan: ["review-discussion", ""]`, `plan_reviewer_fan: ["review-plan", ""]`, `webster_reviewer_fan` and `merge_reviewer_fan: ["review-code", ""]`
- `discussion_fixer_fan: fix-discussion`, `plan_fixer_fan: fix-plan`, `webster_fixer_fan` and `merge_fixer_fan: fix-code`
- `driver: haiku[medium]`, `discussion_max_bounces: 2`, `webster_max_bounces: 1`, `webster_auto_accept: true`

## Operator decision (2026-10-21)

The tuned values move into lyx's template, so they are the default and every new hub, NORCE Models included, gets them with no copying.
The loomyard and quarry hubs' default layers then drop the entries that equal the template.
Any other module whose loomyard values are the operator's general preference, rather than this repo's own, goes the same way; the task checks each.
Note `lenses-language-sets` covers which lens set a hub seeds by language, which is separate.
