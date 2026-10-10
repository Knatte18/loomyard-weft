# Model and effort set per role and per review row

`per-role-models` · Note · enhancement, config, loom, burler

## Seen

The haiku-roles analysis (`docs/research/haiku-judge.md`, 5459376bb) found that one `judge` key in `loom.yaml` sets the model for all three Bouncer rows and their seed passes; a per-row model exists only as `model`/`effort` in the recipe, which is built into the binary.

## Idea (operator, 2026-10-10)

Most agents a run spawns should take a model of their own in config.

- Give each Bouncer row's judge its own model key in config, so Discussion-Bouncer can stay on Sonnet while Plan-Bouncer and Webster-Bouncer try Haiku 5.5.
- Give the seed pass (role `bouncer-seed`, writes only `round-1-focus.md` before round 1, never gates) a model key separate from the judge's; the analysis rates it Haiku-safe on every row, so it moves to Haiku 5.5 on all three rows once the key exists.
- Find every other role in the loom and darn recipes, batten and webster whose model is shared with another role or settable only in the recipe, and give it its own key in the same task.
- Once it lands, the haiku-roles trial runs Haiku 5.5 on the Plan-Bouncer and Webster-Bouncer judges only.
