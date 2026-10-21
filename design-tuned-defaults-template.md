# The operator tuned loom defaults move into the lyx template, so every new hub gets them

`tuned-defaults-template` · Task · enhancement, config, loom · running · Plan-Write

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

The copy missed `shed.yaml`: the template seeds `driver_kind: llm`, while loomyard sets `go`, so the quarry hub's first run, `csharp-alphabet`, was seeded with an LLM driver to the operator's surprise; the orch then set `driver_kind: go` in quarry.

Other modules where the loomyard hub differs from the template, each to be judged general preference or loomyard's own: `batcher` (budget), `board` (labels), `claude` (timeouts), `darn` (verify, reviewer and fixer models and fans), `gate` (slots), `landing` (publish_verify, deploy), `lenses` (added lenses), `models` (model entries), `orch` (token thresholds), `webster` (`kickstart_pack: false`).

## Operator decision (2026-10-21)

The tuned values move into lyx's template, so they are the default and every new hub gets them with no copying.
Hubs already cloned get them too: the NORCE Models hub was cloned before this, and a seeded default layer writes every key, so an explicit neutral value such as `discussion_lenses: ""` would keep overriding a changed template; the task makes existing hubs follow the template, such as by dropping seeded entries that equal the old template.
The loomyard and quarry hubs' default layers then drop the entries that equal the template.
Everything the operator tunes in loomyard is by default the template everywhere.

### Split (orch, on the operator's word, 2026-10-21)

Into the template, as general preference:

- `loom`: every tuned value listed above;
- `shed`: `driver_kind: go`;
- `darn`: the reviewer and fixer models and fans;
- `claude`: the timeouts;
- `models`: the model entries;
- `orch`: the token thresholds;
- `webster`: `kickstart_pack: false`;
- `batcher`: the budget.

Kept in loomyard's own default layer, as this repo's own:

- `board`: the labels, which name lyx's modules;
- `darn.verify`, `landing.publish_verify` and `landing.deploy`: Go and lyx commands;
- `lenses`: the lenses loomyard added;
- `gate.slots`: a machine's setting, not the template's.

After the change, loomyard's default layer holds only the kept entries.
Note `lenses-language-sets` covers which lens set a hub seeds by language, which is separate.
